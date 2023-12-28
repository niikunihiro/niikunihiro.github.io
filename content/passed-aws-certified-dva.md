+++
title="AWS の DVA に合格した話"
date=2023-12-28
category="AWS"
+++

AWS Certified Developer - Associate (DVA-02) に合格したので、ふりかえりと言うか、むきなおりしようと思います。

## 前提

試験勉強を始める前の私の AWS 力についてはこのような感じです。

- CLF と SAA を保持
- SAA を取得してもうすぐ一年たつ
- 日常的に AWS を使用してはいるものの、SAA で勉強したが使ってないサービスは全部忘れた。自分でもびっくりするくらいの忘れん坊だった。

## むきなおり（YWT）

### 勉強したこと・やったこと

#### 試験ガイド

まずは [AWS Certified Developer - Associate (DVA-C02) 試験ガイド](https://d1.awsstatic.com/ja_JP/training-and-certification/docs-dev-associate/AWS-Certified-Developer-Associate_Exam-Guide.pdf)を確認して、領域が次であることを確認しました。

> 1. AWS でのアプリケーションの開発および最適化
> 2. 継続的インテグレーションと継続的デリバリー (CI/CD) ワークフローを使用したパッケージ化およびデプロイ
> 3. アプリケーションコードとデータの保護
> 4. アプリケーションの問題の特定と解決

#### JP Contents Hub

苦手分野の克服が必至だなと思い、CI/CD が弱い気がしたので JP Contents Hub を使って次のハンズオンを実施してみました。

- [ECS Web Application ハンズオン](https://catalog.workshops.aws/ecs-web-application-handson/ja-JP)
- [Amazon ECS マイクロサービス & CI/CD ハンズオン](https://dcj71ciaiav4i.cloudfront.net/CF2469D0-FE58-11EC-96B1-AB890FC54FAE/)

なんとなく分かった気になりましたが、自分はこれで試験問題を解けるようにはならないなと気付き、模擬試験をすることにしました。

#### 模擬試験（Udemy）

- [【DVA-C02】AWS Certified Developer - Associate | 模擬試験](https://www.udemy.com/course/draft/5609942/)
  - Udemy のセールの時に買ったので 1,200 円でした。
  - 日本語版なのに誤翻訳みたいな文章があって、逆に本番ぽかったです。

下の表のように知識領域ごとに理解度がわかるので、苦手な領域・サービスを把握するのに役立ちます。

<img src="https://raw.githubusercontent.com/niikunihiro/niikunihiro.github.io/dc8856779b35e02a1a4526fbb393049e8979fba8/dva-practice-test.png" width="685" />

デプロイの知識をもうちょっと増やす必要あり。セキュリティ分野は知識がそもそもかけてるっぽいなぁという印象ですね。  
とまぁ分かってない領域が判明したので、ここのサービスのチュートリアルをやってみることにしました。

#### AWS ユーザーガイドのチュートリアル

今回一番役立ったと思うコンテンツです。テキストだけではわかりにくい個々のサービスの理解に役立ちました。
いくつかリンクを載せておきます。

- [アップデートされた Lambda CodeDeploy 関数と AWS のサーバーレスアプリケーションモデルをデプロイします。](https://docs.aws.amazon.com/ja_jp/codedeploy/latest/userguide/tutorial-lambda-sam.html)
- [Amazon S3 トリガーを使用して Lambda 関数を呼び出す](https://docs.aws.amazon.com/ja_jp/lambda/latest/dg/with-s3-example.html)
- [#1: AWS CLI を使用した Amazon DynamoDB と AWS Lambda での、フィルターを使ったすべてのイベントの処理](https://docs.aws.amazon.com/ja_jp/amazondynamodb/latest/developerguide/Streams.Lambda.Tutorial.html)

#### 対策本の本試験想定問題集

対策本の『ポケットスタディ AWS 認定デベロッパーアソシエイト』を持っていたのを思い出したので巻末の模擬試験も受けました。
問題と選択肢が簡潔に書かれているので読みやすかったです。
あと、DynamoDB のキャパシティユニットの計算方法が違う気がしたんですが前のバージョンだったのか。DVA-C02 対応が先月出てたのに今気づいたのでリンク [<i class="fa fa-link"></i>](https://amzn.asia/d/1Qirp4L) 貼っておきます。

#### Skill Builders の模擬テスト

受験当日に試験に行く前の仕上げとして受験しました。20 問中 12 問くらいの正答率でした。よく受かったもんだ。

[AWS Certified Developer - Associate Official Practice Question Set (DVA-C02 - Japanese)](https://explore.skillbuilder.aws/learn/course/14060/aws-certified-developer-associate-official-practice-question-set-dva-c02-japanese)

### 分かったこと・こうしたらよかったこと

1. 試験ガイドの把握が重要だった
2. 早めに模擬試験を受けて、苦手分野を把握する
3. 得意分野・苦手分野ごとの対策を行う

#### 試験ガイドの把握が重要だった

これは結構重要視しています。試験領域の各領域の中で何を問うかが試験ガイドの「推奨される AWS の知識」に書いてあるんですね。なので次のようなタスクをもっとやっておけば良かったなと思いました。

> - AWS サービス API、AWS CLI、および SDK を使用したアプリケーションの開発および保護
> - CI/CD パイプラインを使用した AWS でのアプリケーションのデプロイ

#### 早めに模擬試験を受けて、苦手分野を把握する

これは今回特に思ったことです。次は試験ガイドを読んだらすぐに Skill Builders の模擬テストを受けようと思っています。

#### 得意分野・苦手分野ごとの対策を行う

こんな感じでやるといいかなと思いました。

- 得意な分野
  - 模擬試験などで出題ケースになれる
- 理解が足りて無いな分野
  - 模擬試験を解いて解説を読む
  - Jp Contents Hub にあるハンズオンをやってみる
  - AWS ユーザーガイドのチュートリアルをやってみる
- 全然分かってない分野
  - AWS ユーザーガイドを読む
  - AWS ユーザーガイドのチュートリアルをやってみる

#### その他

65 問を 130 分で回答する必要がある。単純計算で 1 問 2 分で解く必要があるので効率よく文章を読むようにしないとですね。  
出題に対して複数のソリューションがある場合は何を求めているかを最初に把握するために問題文を下から読むのもありだなと思いました。コスト最適化なのかセキュリティ重視なのか運用負荷を下げたいのかなどを先に把握する方が早く答えられることもあると思うので。

### 次はこうしようと思ってること

次の試験ではこう進めるぜということを書いて終わろうと思います。

1. 試験ガイドで「コンテンツドメインと重み設定」から出題分野を確認する
2. 早期に模擬試験を受けて得意分野と苦手分野を把握する
3. 試験ガイドの「試験内容の概要」から
   1. 対象知識に書いてある用語やサービスで苦手分野のものや理解できていない物を調べていく
      1. 例えば「フォールトトレラントな設計パターン (エクスポネンシャルバックオフとジッターを使用した再試行、デッドレターキューなど)」だと「エクスポネンシャルバックオフ」が何かを調べる感じで
   2. 対象スキルに書いてある内容で苦手分野や、やったことがない・知識ゼロなことがあればチュートリアルなどをやってみる
      1. 例えば「シークレット管理サービスの使用による機密データの保護」とか「開発エンドポイントを使用したアプリケーションのテスト (Amazon API Gateway でのステージの設定など)」など色々書いてあるのでやったことないものがあれば試してみる。
4. 試験範囲の学習が終わったら、模擬試験を繰り返し受けて問題になれる。
5. AWS 認定の短期決戦はやめて日常的に学習を行うカテゴリに入れる。
