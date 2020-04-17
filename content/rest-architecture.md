+++
title="REST を理解しようとしている"
date=2020-04-04
category="REST"
+++

Roy Fielding の REST (REpresentational State Transfer) を理解しようとしている。全然わからない。

経緯としては、こういうことのよう。

- アプリケーションサーバが HTTP のセマンティクスを使用しないアーキテクチャを採用していた。
- アプリケーションサーバは一枚岩でスケールしない。
- REST はそのやり方を止めた。
- REST を使うことにより、容易にサーバを追加できるのでスケールしやすくなった。

つまり REST は HTTP をちゃんと使うためのアーキテクチャの１つということなのだろうか。そういう観点で REST 制約を確認するとなんとなく腑に落ちる。

## REST 制約

| 制約 | 概要 |
| :-- | :-- |
| 1. Client-server archtecture | ユーザーインターフェースと処理を分離する |
| 2. Statelessness | サーバでアプリケーション状態を持たない |
| 3. Cacheability | クライアントとサーバの通信回数を減らす |
| 4. Layered System | システムを階層に分離する |
| 5. Code on Demand (optional) | プログラムをサーバからクライアントにダウンロードして実行する |
| 6. Uniform Interface | インターフェースを固定する |
| &nbsp; 6.1 Resource identification in requests | アドレス可能性 |
| &nbsp; 6.2 Resource manipulation through representations | 表現を通じたリソース操作 |
| &nbsp; 6.3 Self-descriptive messages | 自己記述的メッセージ |
| &nbsp; 6.4 Hypermedia as the engine of application state (HATEOAS) | ハイパーメディア制約 |


REST はシステムの大きな枠の設計で、ネットワークシステムのアーキテクチャスタイルの１つ。ちなみに SWEBOK による分類だと OOP とか DDD とかはマイクロアーキテクチャパターン（デザインパターン）と呼ぶ。REST とは別のレイヤーの話になるので混同しないようにしたい。

少し理解は進んだ気がするが、やっぱり全然わからない。

---

1. Leonard Richardson, Sam Ruby『RESTful Web サービス』O'REILLY 2007年12月
2. [https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm)
3. [https://en.wikipedia.org/wiki/Representational_state_transfer](https://en.wikipedia.org/wiki/Representational_state_transfer)
4. [http://web.archive.org/web/20130330045743/http://www.geocities.jp/yamamotoyohei/rest/rest-to-my-wife.htm](http://web.archive.org/web/20130330045743/http://www.geocities.jp/yamamotoyohei/rest/rest-to-my-wife.htm)
5. [https://www.infoq.com/jp/news/2010/03/RESTLevels/](https://www.infoq.com/jp/news/2010/03/RESTLevels/)
6. [https://speakerdeck.com/koriym/rest-6-plus-4falsezhi-yue](https://speakerdeck.com/koriym/rest-6-plus-4falsezhi-yue)
