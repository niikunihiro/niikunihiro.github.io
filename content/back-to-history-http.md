+++
title="HTTP の歴史を 10 分で振り返る"
date=2020-02-29
category="HTTP"
+++

このエントリはネットワークを効率的に利用する Web アプリケーションを開発するための前提知識をまとめようという試みです。

ネットワークの視点から HTTP の歴史をバージョン順に見ていくと、通信回数とデータ量を減らすことと、通信の稼働率を上げることに力が注がれてきたことがわかります。なので TCP コネクションに絞って約 10 分でわかった気になれるようにします。


## バージョンごとの TCP コネクション

HTTP はトランスポートプロトコルとして TCP を使っています。TCP は HTTP メッセージのやり取りの前にクライアント・サーバ間でコネクションの確立を行ってから通信を開始します。他にもパケットの再送制御や順序制御などデータの転送を制御するので、コネクション型通信や信頼性のある通信とも呼ばれています。

### HTTP/1.0 以前

HTTP/1.0 まではリクエストごとに TCP コネクションを確立し切断していました。リクエストごとに3ウェイ・ハンドシェイクを行っていたので通信の効率が悪く無駄な通信が発生していました。

#### 3ウェイ・ハンドシェイク

<a title="Guojunzheng / CC BY-SA (https://creativecommons.org/licenses/by-sa/3.0)" href="https://commons.wikimedia.org/wiki/File:Three-way-handshake-example.gif"><img width="512" alt="Three-way-handshake-example" src="https://upload.wikimedia.org/wikipedia/commons/thumb/f/f0/Three-way-handshake-example.gif/512px-Three-way-handshake-example.gif"></a>

リクエストごとに上記の通信を行ってから HTTP メッセージを送っていました。毎回となるとやっぱりちょっと多い気がしますね。

### HTTP/1.1

Keep-Alive の導入により TCP コネクションを確立したらコネクションを維持し、データのやり取りが終わってから切断するようになったので、ハンドシェーク削減分の通信回数を減らすことができるようになりネットワークの効率が向上されました。  
また、HTTP パイプラインのサポートにより複数のリクエストを送るときにレスポンスを待たずに次のリクエストを送信できるようになりました。

#### HTTP パイプライン

<a title="Mwhitlock / Public domain" href="https://commons.wikimedia.org/wiki/File:HTTP_pipelining2.svg"><img width="512" alt="HTTP pipelining2" src="https://upload.wikimedia.org/wikipedia/commons/thumb/1/19/HTTP_pipelining2.svg/512px-HTTP_pipelining2.svg.png"></a>

送信側は複数の TCP コネクションを確立して、ネットワークの稼働率を上げてネットワークのパフォーマンスを向上できるようになりましたが、受信側はリクエストが来た順番でレスポンスを返すという制約があり、複数のリクエストが同時期にきた場合、その中に重いリクエストやレスポンスの処理が含まれているとその処理が終わるまで以降のレスポンスは待たされるため Web ページの表示速度が遅くなるなどの問題点がありました。

### HTTP/2

HTTP パイプラインは通信の順番を保持する制約により、重い通信がボトルネックになることから HTTP/1.1 で活かすことができませんでしたが HTTP/2 でストリームとして生まれ変わりました。

#### ストリーム

ストリームは、物理的に一つの TCP コネクションで、論理的に並列化された通信を行うものです。なんとなく細かく分けて送受信してるんだなという感じですね。

![Figure 12-3. HTTP/2 request and response multiplexing within a shared connection](https://raw.githubusercontent.com/niikunihiro/niikunihiro.github.io/assets/HTTP2_request_response_multiplexing_within_shared_connection.svg?sanitize=true "Figure 12-3. HTTP/2 request and response multiplexing within a shared connection")


論理的なイメージとしては、下図の Stream 1 ~ Stream N のように TCP コネクション上に仮想的な通信経路を複数生成し、それぞれのストリーム内でリクエストとレスポンスのやり取りを行います。

![Figure 12-2. HTTP/2 streams, messages, and frames](https://raw.githubusercontent.com/niikunihiro/niikunihiro.github.io/assets/HTTP2_streams_messages_frames.svg?sanitize=true "Figure 12-2. HTTP/2 streams, messages, and frames")

ストリーム内ではハンドシェークはなく、フラグを使ってストリームを作成したり閉じたりできるようになっています。
ストリームのIDの最大値と通信容量が許す限り、数万接続でも並列化できるようになりました。  
SPA を作るときなんかは通信回数が増える（特に初回は）ので、ストリームの恩恵を受けられる HTTP/2 にしときたいですね。

### HTTP/3

HTTP-over-QUIC と呼ばれていた仕様策定中の規格です。  
QUIC はトランスポートプロトコルとして UDP を使ってリクエストを行うのでハンドシェークを行いません。UDP はコネクションレス型通信なので信頼性のない通信ですが、ポート番号によるアプリケーションの識別やチェックサムによるデータの破損のチェックを行うことは可能です。QUIC は UDP の上で通信の再送信や多重化などの機能を提供し HTTP に利用しようとするものらしいです。まだ詳しくは理解できてないです。


## データ形式

HTTP/1.1 まではテキスト形式で通信を行っていましたが、HTTP/2 では「フレーム」と呼ばれるバイナリ形式でデータを分割して送受信するようになりました。  
テキストからバイナリへ変換する処理が不要になった分の通信時間を削減できるようになりました。


## あとがき

1990年末にティム・バーナーズ＝リーたちによって最初の Web サーバが作成されて以来、HTTP（アプリケーション層）も、ネットワークインターフェース層やインターネット層、トランスポート層と同じように、高速化と効率的な利用を行うための取り組みが行われて来たことがちょっとでも伝わっていたらうれしいです。

本稿を書くにいたるまで Web アプリケーションの作成では HTTP ヘッダやボディを使って HTTP メッセージをやり取りするという Semantics の視点が主で、ネットワークを効率的に利用するということは意識できていませんでした。  
これからはネットワークを効率的に使うという視野も持ちながら設計・開発を行っていきたいと気持ちを新たにしたので、次はキャッシュについて書こうかなと思ってます。

この稿おわり

---

#### 参考文献

1. 井上 直也、村山 公保、竹下 隆史、荒井 透、苅田 幸雄『マスタリングTCP/IP 入門編』第6版 2019年11月
2. 渋川よしき『Real World HTTP』O'REILLY 2017年6月
