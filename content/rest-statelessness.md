+++
title="REST 制約の Statelessness"
date=2020-04-18
category="REST"
+++

Hypertext Transfer Protocol はステートレス・プロトコルの代名詞みたいな物と思ってるんですが、REST ももちろんステートレス・プロトコルを利用しています。が、今回のテーマはプロトコルの話ではないのです。

Fielding の論文でステートレスなアーキテクチャスタイルとは

> The client-stateless-server style derives from client-server with the additional constraint that no session state is allowed on the server component. Each request from client to server must contain all of the information necessary to understand the request, and cannot take advantage of any stored context on the server. Session state is kept entirely on the client.

といっている。

ざっくりいうとこういうことらしい

* サーバでセッション状態を持つことを禁止
* セッション状態はクライアントで保持
* リクエストごとに必要な情報をすべて含める

で、これの何がいいのかというと

> These constraints improve the properties of visibility, reliability, and scalability. Visibility is improved because a monitoring system does not have to look beyond a single request datum in order to determine the full nature of the request. Reliability is improved because it eases the task of recovering from partial failures. Scalability is improved because not having to store state between requests allows the server component to quickly free resources and further simplifies implementation.

* 監視しやすくなって
* 復旧しやすくなって
* スケーラビリティが向上する

といっている。

ただ、リクエストごとに必要な情報をすべて含めるのでネットワークのパフォーマンスが低下する可能性があると言及している。

> The disadvantage of client-stateless-server is that it may decrease network performance by increasing the repetitive data (per-interaction overhead) sent in a series of requests, since that data cannot be left on the server in a shared context.

Fielding が Client-Stateless-Server (CSS) の次に Client-Cache-Stateless-Server (C$SS) を続けているのはキャッシュで Statelessness の欠点を補おうという意図なのかも。

セッション状態あたりに話を戻して、

Statelessness はサーバでセッション状態を持たないという話ですが、これは RESTの Representational （表現）に続く話のような気がしている。  
どういうことかというと Statelessness ではリクエストごとに必要な情報をすべて含めるとあるが、サーバはセッション状態を持つことを禁止するとしか書かれておらずサーバがやることについては言及していない。
ただ、サーバでセッション状態を持たないと言っても、アプリケーションに状態は存在する。Restful Webサービスでは状態について、<q>クライアント側で維持されるアプリケーション状態と、サーバ上で維持されるリソース状態とを区別する</q>といっている。

アプリケーション状態とはクライアントが表示しているページや経路に関する情報や入力データなどで、クライアントはアプリケーション状態の一部をサーバに送信することでリソース状態を変更する事ができる。  
リソース状態とは画像やデータベースなどのデータで、アプリケーションの操作により状態が変化する。  
サーバはレスポンスにクライアントがたどるための経路をハイパーリンクとして含めることでセッション状態を持つことなくクライアントの状態を変化させていく。  
これが REpresentational State Transfer ということなのか。

----


1. [Roy Thomas Fielding. Architectural Styles and
the Design of Network-based Software Architectures - 5.1.3 Stateless](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_1_3)
2. Leonard Richardson, Sam Ruby『RESTful Web サービス』O'REILLY 2007年12月
