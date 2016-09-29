# アプリケーションエンジニアのための oslo.messaging 徹底解説

　本稿はアプリケーションエンジニアのための OpenStack の記事になります。ここでの内容によって、読者が携わるアプリケーションのミドルウェア周りの実装や、アプリケーション自体のサステーナビリティ (Sustainability) の向上に寄与できると考えております。  

　OpenStack は IaaS クラウド機能を提供する OSS として知られていますが、アプリケーションエンジニアにとっては業務フィールドのレイヤが違うため関心が薄いかもしれませんが、本稿ではそうしたエンジニアに向けてアプリケーションエンジニアが活用できる OpenStack プロジェクト 'Oslo' について紹介と、そのうちの RPC と通知機能を提供するライブラリ 'oslo.messaging' の機能と使い方について徹底解説して行きます。  
　尚 OpenStack 自体について興味がある方は [OpenStack に関する別の連載記事](https://codezine.jp/article/detail/9636) も参照してみてください。  

　本稿ではアプリケーションエンジニアの読者向けに OpenStack の共通ライブラリの一つ oslo.messaging の使い方について徹底解説して行きます。  

### Oslo とは？
　[Oslo](https://wiki.openstack.org/wiki/Oslo) は OpenStack の各コンポーネント (OpenStack の最も大きなソフトウェアモジュールの単位) で共通で利用するライブラリを集めたプロジェクトになります。  
　その昔の OpenStack では設定ファイルやコマンドライン引数の解析、メッセージパッシング、ロギングなどといった処理が各コンポーネント毎に個別に存在しており、開発・メンテナンスコストが掛かる問題がありました。  
　そして Grizzly サイクルで共通ライブラリ群を提供する olso プロジェクトが発足し、Havana で設定ファイルとコマンドライン引数の解析を行なう [oslo.config](http://docs.openstack.org/developer/oslo.config) と、通知や RPC の仕組みを提供する [oslo.messaging](http://docs.openstack.org/developer/oslo.messaging) がリリースされました。現在では、国際化や並行処理など、本稿執筆時点で 34 のプロジェクトが存在しています。  
　なお余談ですが、この 'Olso' のネーミングは、イスラエルとパレスチナの和平協定 'オスロ合意' にちなんで付けられたという [経緯](http://docs.openstack.org/project-team-guide/oslo.html#brief-history) があったりします。  

　oslo 自体は OpenStack のプロジェクトですが、用途は OpenStack に限定されません。  
　例えば oslo.config は、設定ファイルとコマンドライン引数の解析を行なうライブラリとして広く知られており、NTT Communication などが中心となって開発している [SDN Framework Ryu](https://osrg.github.io/ryu/) などで採用されています。  
　他にもロギング処理全般を行う olso.log や CLI フレームワークの cliff などが有名で、[OpenStack ユーザ会のメンバーによる解説](http://www.slideshare.net/h-saito/openstack-oslo-cliff) などもあります。  

　本稿で取り上げる oslo.messaging は、[RabbitMQ](rabbitmq.com) や [ZeroMQ](http://zeromq.org/) などの MQ システムを介した通知や RPC の仕組みを提供します。  

### oslo.messaging の概要
　oslo.messaging の使い方について見て行く前に、oslo.messaging についての理解を深めるために、なぜこの仕組みを使うのかについて考えてみたいと思います。  
　MQ を利用した通知や RPC の仕組みは、もちろん oslo.messaging が登場する以前から存在していました。RabbitMQ を使う場合には [Pika](https://github.com/pika/pika) という AMQP ライブラリを用いてそれらの処理を行なうことができます。また ZeroMQ を使う場合には、[PyZMQ](https://github.com/zeromq/pyzmq) ライブラリを用い、Kafka にも [同様の Python ライブラリ](http://zeromq.org/) が存在します。  
  
　しかしこれらを用いることによって、ミドルウェアないはプロトコル依存の実装になってしまいます。例えば MQ に `RabbitMQ` を使っていて、`Kafka` に切り替えたいと思った際、上記のような特定の MQ 用のライブラリを利用していた場合、それなりの規模のコード修正が必要になると思います。  

![picture1]

  これに対し oslo.messaging では各 MQ ライブラリを抽象化しており、利用する MQ に応じてドライバを選択できる仕組みを提供しています。そのため、先のように MQ を 'RabbitMQ' から 'Kafka' に変更したいといった場合でも、設定ファイルを変更するだけでアプリケーション側のコードの変更せずに切り替えることができます。  

![picture2]

　このように oslo.messaging では、ミドルウェア・プロトコル非依存の通知、RPC の仕組みを提供しており、これによってユーザは様々な MQ システムを選択することができるようになり、アプリケーションをサステーナブル (Sustainable) にすることができます。  
