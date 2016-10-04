## oslo.messaging の概要
　oslo.messaging の使い方について見て行く前に、oslo.messaging についての理解を深めるため、なぜこの仕組みを使うと良いかについて考えてみたいと思います。  
　MQ を利用した通知や RPC の仕組みは、もちろん oslo.messaging が登場する以前から存在していました。RabbitMQ を使う場合には [Pika](https://github.com/pika/pika) という AMQP ライブラリを用いてそれらの処理を行なうことができます。また ZeroMQ を使う場合には、[PyZMQ](https://github.com/zeromq/pyzmq) ライブラリを用い、[Kafka](http://kafka.apache.org/) にも [同様の Python ライブラリ](http://zeromq.org/) が存在します。  
  
　しかしこれらを用いることによって、アプリケーションにミドルウェアないしはプロトコル依存のコードが入ってしまいます。例えば MQ に `RabbitMQ` を使っていて、これを `Kafka` に切り替えたいと思った場合、アプリケーションにそれなりの規模のコード修正が必要になると思います。  

![picture1](https://github.com/userlocalhost2000/draft-codezine-oslo.messaging/blob/master/img/picture1.png?raw=true)

  これに対し oslo.messaging は各 MQ ライブラリを抽象化しており、利用する MQ に応じてドライバを選択できる仕組みを提供しています。そのため、先のように MQ を 'RabbitMQ' から 'Kafka' に変更したいといった場合でも、設定ファイルを変更するだけでアプリケーションのコード修正を行わずに MQ を切り替えることができます。  

![picture2](https://github.com/userlocalhost2000/draft-codezine-oslo.messaging/blob/master/img/picture2.png?raw=true)

　このように oslo.messaging によって、ミドルウェア・プロトコル非依存の通知、RPC をアプリケーションに実装できます。これによってユーザは様々な MQ システムを選択することができるようになり、アプリケーションをサステーナブル (Sustainable) にすることができます。  
