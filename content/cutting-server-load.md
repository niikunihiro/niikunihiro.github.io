+++
title="リクエストが多すぎて困る？"
date=2023-12-09
category="REST"
+++

利用者の多いサービスを展開しているとリクエストが跳ね上がることも多々ありますよね。うれしい悲鳴とでもいいましょうか有難い面もあればレスポンスが遅い状態が続くとユーザーの不満足にもなるので、そうはいってられないこともあります。リクエストが増加する要因はサービスの性質によりいろんなケースがあると思いますが、予期せぬリクエストの増加はスケールが追いつかない時もありなかなか悩ましいものです。  
今回はいくつかある対応のうちサーバ負荷を上げずにリクエストを捌く方法について考察を行いたいと思います。

### インフラからのアプローチ

これは基本的に水平スケールにより負荷分散を試みます。
ロードバランサーを使って負荷分散し、リソースが足りなくなってきたらオートスケーリングを実施。
クラウドを利用している場合は設定を行うことで利用可能。  
お財布と相談がいる方法（継続的にお金がかかる）

### アプリケーションからのアプローチ

これは基本的に通信を減らすことで達成を試みます。
通信量または通信回数を減らす方法がありますが、どっちかというと通信回数を減らすのが効果的です。
通信量（データ量）が多いとリソースをその分使用するため減らすことに効果がないわけではないですが、通信回数を減らすことはサーバのリソースを使用しないことにつながります。
アプリケーションに機能として組み込むので実装することで利用可能。  
こちらもお財布と相談がいる方法（開発するので一時的にお金がかかる）

通信以外にアルゴリズムやデータ構造を変えたりすることで計算量を減らしたり、SQL を書き換えてパフォーマンスを改善だったり、SSR と CSR の使い分けもテーマとして面白いですが別の機会にゆずります。特記することがあるとすれば Redis の keys コマンドは本番環境で使わないほうがいいと思います。

---

通信回数を減らす方法の考察に入りますが、ひとつは「キャッシュ」を利用すること。  
イメージしやすくするためにシーケンス図を用意しました ↓

{% mermaid() %}
sequenceDiagram
actor user
participant C as クライアント
participant CC as クライアント<br>キャッシュ
participant S as サーバ
participant SC as サーバ<br>キャッシュ
participant BK as マイクロサービス<br>データベースなど

user->>+C: open
C->>+S: request
Note right of C: キャッシュなし時
S->>+BK: get data
BK-->>-S: data
S->>+SC: save cache
SC-->>-S: result
S-->>-C: response
C->>+CC: save cache
CC-->>-C: result
C-->>-user: view

user->>+C: open
C->>+S: request
Note right of C: サーバキャッシュ利用時
S->>+SC: get cache
Note right of SC: バックエンドへの<br>通信削減
SC-->>-S: data
S-->>-C: response
C->>+CC: save cache
CC-->>-C: result
C-->>-user: view

    user->>+C: open
    C->>+CC: get data
    Note right of C: クライアントキャッシュ利用時
    Note right of CC: サーバへの<br>通信削減
    CC-->>-C: data
    C-->>-user: view

{% end %}

クライアントから見て外部リソースを使わなければ使わないほどパフォーマンスも良くなり、サーバ負荷も抑えることができるイメージができたかと思います。

では次にどうやって実装するのかという話に入ります。

HTTP には条件付きリクエスト（HTTP conditional requests）という概念が用意されていて、キャッシュ利用のプロトコルが定められています。詳しくはこちらを参照してもらうと丸わかりです →
[HTTP 条件付きリクエスト - HTTP | MDN](https://developer.mozilla.org/ja/docs/Web/HTTP/Conditional_requests)

検証子や条件ヘッダーなど見たことある人もいるのではないでしょうか。

サーバ側のキャッシュ制御についてはこちらも見ておくとさらに理解が進みます → [HTTP キャッシュ - HTTP | MDN](https://developer.mozilla.org/ja/docs/Web/HTTP/Caching)

解説を MDN にぶん投げたままではちょっとアレなので、条件付きリクエストの動きをクライアント・サーバ・サーバのバックエンドのシーケンス図に落とし込んでみます。

{% mermaid() %}
sequenceDiagram
actor user
participant C as クライアント
participant CC as クライアント<br>キャッシュ
participant S as サーバ
participant SC as サーバキャッシュ
participant BK as マイクロサービス<br>データベースなど
user->>+C: open
C->>+S: GET /doc HTTP/1.1
Note right of C: 初回リクエスト
S->>+BK: get data
BK-->>-S: {"foo":"bar"}
S->>+SC: save cache<br>Etag,Last-Modified,{"foo":"bar"}
SC-->>-S: result
S->>-C: HTTP/1.1 200 OK<br>Last-Modified: date<br>Etag: "xyz"<br>{"foo":"bar"}
C->>+CC: save {"foo":"bar"}
CC-->>-C: result
C-->>-user: view

    user->>+C: open
    C->>+S: GET /doc HTTP/1.1<br>If-Modified-Since: date<br>If-None-Match: "xyx"
    Note right of C: 2回目以降のリクエスト<br>リソースの更新なし
    S->>+SC: check Etag or Last-Modified
    SC-->>-S: not modifed
    S->>-C: HTTP/1.1 304 Not Modified
    C->>+CC: get data
    CC-->>-C: {"foo":"bar"}
    C-->>-user: view

    user->>+C: open
    C->>+S: GET /doc HTTP/1.1<br>If-Modified-Since: date<br>If-None-Match: "xyx"
    Note right of C: 3回目以降のリクエスト<br>リソースの更新あり
    S->>+SC: check Etag or Last-Modified
    SC-->>-S: modifed
    S->>+BK: get data
    BK-->>-S: {"foo":"buzz"}
    S->>+SC: save cache<br>Etag,Last-Modified,{"foo":"buzz"}
    SC-->>-S: result
    S->>-C: HTTP/1.1 200 OK<br>Last-Modified: date2<br>Etag: "xyz2"<br>{"foo":"buzz"}
    C->>+CC: save {"foo":"buzz"}
    CC-->>-C: result
    C-->>-user: view

{% end %}

リソースの変更の有無を確認して、変更がない場合はキャッシュを利用し、変更があった場合はリソースを更新するという動きに関するプロトコルが条件付きリクエストになります。  
アプリケーションの特性によっては独自にキャッシュの利用ルールを定めることも選択肢の一つにはなり得ますが、HTTP のルールに則って実装するのも面白いと思うのでぜひ検討してほしいです。

`If-None-Match` や `Etag` , `Cache-Control` ヘッダーの使い方を理解してもらった上での補足ですが、条件付きリクエストが使えるのはクライアントとサーバ間の通信に限りません。

上記シーケンス図を例にあげるとサーバとマイクロサービス間でも HTTP を使用していれば実装可能で、サーバの場合は [RFC 7234](https://httpwg.org/specs/rfc7234.html) に準拠しているライブラリに任せることができます。

Node.js の場合は [cacheable-request](https://www.npmjs.com/package/cacheable-request) や [got](https://www.npmjs.com/package/got) というライブラリが RFC 7234 に準拠しています。他の言語の場合も RFC 7234 対応 API クライアントを利用すると条件付きリクエストのヘッダに対応してくれるので開発も割と楽にできるんじゃないかと思います。

---

ちなみに通信量を減らす方法は「圧縮」することで実現します。圧縮・展開にかかる時間は通信にかかる時間よりも少ない時間で行えるので通信のトータル時間は減ることになります。

通信自体はいろんな方法で圧縮されており、たとえばブラウザでリクエストをのぞき見ると次のようなリクエストヘッダが送信されているのを確認できるかと思います。

```
accept-encoding: gzip, deflate, br
```

gzip や deflate、br は圧縮アルゴリズムを指しています。

圧縮のネゴシエーション自体はヘッダを設定することで可能ですが JavaScript でリクエストデータを送信したりサーバでデータを返す際は圧縮の実装が要ります。サーバの場合 Apache などのミドルウェアに任せるということも可能なので、それに任せるというのもアリだと思います。ちょっと雑な書き方してるのでこの辺は別の機会に書ければということでこのエントリおわります。
