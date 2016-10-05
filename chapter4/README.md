## oslo.messaging を使ったRPC 処理の実装
　それではサンプルコードを動かしながら oslo.messaing で RPC over MQ を行なう方法を解説して行きます。  
　その前に、システムのアウトラインについて簡単に把握したいと思います。以下は、クライアント/サーバの内部アーキテクチャについて表した図になります。  
 
![内部アーキテクチャ](http://tsuchinoko.dmmlabs.com/wp-content/uploads/2016/08/structure.png)
(出典：[ツチノコブログ](http://tsuchinoko.dmmlabs.com/?p=4371))  

　クライアント側では RPCClient オブジェクトがユーザアプリケーションに対して、RPC 処理のインターフェイスを提供します。また Transport オブジェクトが、RabbitMQ や Kafka , ZeroMQ などの各種 MQ を抽象化し、ユーザは RPCClient が提供するインターフェイスを通して MQ 非依存な RPC リクエストをサーバに送ることができます。  
　サーバ側では、ユーザからの RPC リクエストを処理する Server オブジェクトを生成します。その際、サーバ側のアプリケーションで各 PRC 要求に対応するコールバック関数を定義したクラス (Endpoint) を一つ以上実装してやり、それらを Server オブジェクト生成時に指定します。  

　それではサンプルコードを見ながらそれぞれの使い方を把握し、実際に動かしてみます。  

---

* 4-1. [実装 (RabbitMQ 編)](https://github.com/userlocalhost2000/draft-oslo.messaging/tree/master/chapter4/chapter4-1)
* 4-2. [実装 (ZeroMQ 編)](https://github.com/userlocalhost2000/draft-oslo.messaging/tree/master/chapter4/chapter4-2)
* 4-3. [OpenStack (Nova) の利用例](https://github.com/userlocalhost2000/draft-oslo.messaging/tree/master/chapter4/chapter4-3)
