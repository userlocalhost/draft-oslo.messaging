## oslo.messaging を使ったRPC 処理の実装
  それではサンプルコードを動かしながら汎用的な RPC over MQ 処理を行なう方法を解説して行きます。  
  その前に、システムのアウトラインについて簡単に把握したいと思います。以下は、クライアント/サーバの内部アーキテクチャについて表した図になります。  
 
![内部アーキテクチャ](http://tsuchinoko.dmmlabs.com/wp-content/uploads/2016/08/structure.png)
(出典：[ツチノコブログ](http://tsuchinoko.dmmlabs.com/?p=4371))  

　クライアント側では RPCClient オブジェクトがユーザアプリケーションに対して、RPC 処理の仕組みを提供します。また Transport オブジェクトが、RabbitMQ や Kafka , ZeroMQ などの各種 MOM を抽象化し、ユーザは RPCClient が提供するインターフェイスを通して MOM 非依存な RPC リクエストをサーバに送ることができます。  
　サーバ側では、ユーザからの RPC 要求を処理する Server オブジェクトを生成します。その際、ユーザ側で各 PRC 要求に対応する処理 (Endpoint) を一つ以上実装してやり、それらを Server オブジェクト生成時に指定します。  
　また Server 及び Endpoint(s) に対して RPC API の名前空間と互換 version について設定することができます。これらの設定を保持したものが Target オブジェクトになり Server 及び Endpoint にひも付きます。  
　Endpoint オブジェクトにおいて Target を省略した場合には、グローバルな名前空間と version1.0 が暗黙的に設定されます。ただし Server オブジェクトにおいては Target オブジェクトを topic 及び server パラメータ付きで指定しなければなりません。  
　Target オブジェクトの topic パラメータは、サーバが提供する API を識別するための項目で amqp ドライバにおいては名前付きキューの名前に対応します。クライアントはこの値だけを知っていれば、サーバの場所を意識せずにサーバに対して RPC 要求を送ることができます。  
　また Target オブジェクトの server パラメータは、文字通り API を提供するサーバを指定する項目ですが、実際に存在するサーバのホスト名ないしは IP アドレスを指定しなければならないわけではく、サーバを指定する任意の文字列になります。これはクライアントが topic パラメータで指定した API を持つサーバ群のうち、特定のサーバに対してメッセージを送りたい場合に利用されるパラメータになり amqp ドライバにおいては、topic 名で指定した名前付きキューに加えて、"${topic}.${server}" の名前付きキューを生成します (${topic} と ${server} にはそれぞれパラメータの設定値が入ります)。  

---

* [4-1. 実装 (RabbitMQ 編)](https://github.com/userlocalhost2000/draft-oslo.messaging/tree/master/chapter4/chapter4-1)
* [4-2. 実装 (ZeroMQ 編)](https://github.com/userlocalhost2000/draft-oslo.messaging/tree/master/chapter4/chapter4-2)
* [4-3. OpenStack (Nova) の利用例](https://github.com/userlocalhost2000/draft-oslo.messaging/tree/master/chapter4/chapter4-3)
