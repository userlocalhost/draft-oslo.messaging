### 実装 (RabbitMQ 編)
　まずはサーバ側のコード `src/rpc_server.py` について解説します。以下はコードの抜粋になります。  

```Python
    10	class TestEndpoint(object):
    11	  target = oslo_messaging.Target(namespace='foo', version='1.2')
    12	
    13	  def hoge(self, ctxt, arg):
    14	    print("[TestEndpoint] hoge(%s, %d) is called" % (ctxt, arg))
    15	    return arg * 2
    16	
    17	def start_server(tgt_topic = DEFAULT_TOPIC, tgt_server = DEFAULT_SERVER, url=''):
    18	  transport = oslo_messaging.get_transport(cfg.CONF, url=url)
    19	  target = oslo_messaging.Target(topic=tgt_topic, server=tgt_server)
    20	  endpoints = [
    21	    TestEndpoint(),
    22	  ]
    23	
    24	  server = oslo_messaging.get_rpc_server(transport, target, endpoints)
```
 
　10 行目の `TestEndpoint` が RPC リクエストを処理するメソッド (hoge) を実装したエンドポイントです。ここではエンドポイントの定義を一つだけしか指定していませんが、章の冒頭で説明したとおり複数のエンドポイントを指定することもできます。  
　24 行目の get_rpc_server メソッドの呼び出しで [Server](http://docs.openstack.org/developer/oslo.messaging/server.html) オブジェクトを生成します。Server オブジェクトは 18 行目で生成された Transport オブジェクト(各種 MQ を抽象化したオブジェクト)、及び後述する Target オブジェクト、そして 10 行目で定義されたエンドポイントのオブジェクトにひも付きます。  
　Target は API の namespace と version 情報を保持したデータ構造で、エンドポイントと Server にひも付きます。Target は、接続する RPC クライアント、及び受け取る RPC リクエストを取捨選択を出来るようにするためのクラスで、サンプルでは 11 行目でエンドポイントの Target オブジェクトを生成しており、19 行目で Server の Target オブジェクトを生成しています。  
　Endpoint オブジェクトにおいて Target の設定を省略した場合には、グローバルな namespace (=None) と version (=1.0) が暗黙的に設定されます。ただし Server オブジェクトにおいては Target オブジェクトを topic 及び server パラメータ付きで指定しなければなりません。  
　Target オブジェクトの topic パラメータは、サーバが提供する API を識別するための項目で amqp ドライバにおいては [名前付きキュー](https://www.rabbitmq.com/tutorials/tutorial-one-python.html) の名前に対応します。クライアントはこの値だけを知っていれば、サーバの場所 (IP アドレス/ホスト名) を意識せずに RPC リクエストをサーバに送ることができます。  
　また Target オブジェクトの server パラメータは、文字通り API を提供するサーバを指定する項目ですが、サーバを一意に識別できる任意の文字列になります（ホスト名や IP アドレスと一致していなくても問題ありません）。これはクライアントが topic パラメータで指定した API を持つサーバ群のうち、特定のサーバに対してメッセージを送りたい場合に利用されます。amqp ドライバにおいては topic 名で指定した名前付きキューに加えて、"${topic}.${server}" の名前付きキューを生成します (${topic} と ${server} にはそれぞれパラメータの設定値が入ります)。  
　最後に、Endpoint のコールバックメソッドは `ctxt` と `arg` の２つのパラメータを取ります。サンプルの実装では、`ctxt` に任意の dict 型の値を受け取り、`arg` の値を 2 倍してクライアントに結果を返すという処理を行います。  

　次にクライアントのコード `src/rpc_client.py` について見て行きます。同じようにコードの抜粋を以下に示します。  
  
```Python
     9	class TestClient(object):
    10	  def __init__(self, transport, tgt_topic, tgt_server):
    11	    target = oslo_messaging.Target(topic=tgt_topic, server=tgt_server)
    12	    self.client = oslo_messaging.RPCClient(transport, target)
    13	
    14	  def hoge(self, ctxt, arg):
    15	    cctxt= self.client.prepare(namespace='foo', version='1.1')
    16	    return cctxt.call(ctxt, 'hoge', arg = arg)
    17	
    18	def send_request(ctx, arg, topic=DEFAULT_TOPIC, server=DEFAULT_SERVER, url=''):
    19	  transport = oslo_messaging.get_transport(cfg.CONF, url=url)
    20	  client = TestClient(transport, topic, server)
    21	
    22	  return client.hoge(ctx, arg)
```

　18 行目の send_request メソッドの呼び出しでサーバへ RPC リクエストを送信し、サーバで実行された処理の結果を返します。内部では RPC クライアント処理を行う oslo.messaging のクラス [RPCClient](http://docs.openstack.org/developer/oslo.messaging/rpcclient.html) のオブジェクトを生成し、サーバ処理と同様に Transport オブジェクトと Target オブジェクトをひも付けて RPC リクエストを送ります。  
　9 行目で定義している TestClient は RPCClient のラッパーで 20 行目でオブジェクト化しています。引数に Transport オブジェクトを取り、内部で RPCClient オブジェクトを生成し Target オブジェクトとのひも付けを行っています。  
　そして 22 行目のメソッド `hoge` の呼び出しで、サーバに対して RPC リクエストを送ります。内部では 15 行目の prepare() メソッドを呼び出し、RPCClient オブジェクトが内部に持つ Target オブジェクトの namespace と version パラメータを設定します。  
　11 行目と 15 行目で 2 回 Target オブジェクトの設定をしていることを疑問に思うかもしれません。先ほどのサーバ側の処理で Server とエンドポイントの 2 つに別々の Target をひも付けていたことを思い出してください。11 行目の Target オブジェクトは RPC リクエストを送るサーバを識別するために設定されたもので、15 行目で内部的に設定される Target は RPC リクエストを処理するエンドポイントを識別するための設定です。  
　また 15 行目で設定された version パラメータ値 '1.1' がサーバのエンドポイント `TestEndpoint` で設定した値 '1.2' と一致していないことに気づいたかもしれません。Server 内部で実施されるバージョンネゴシエーション処理では、クライアントからの要求 version のメジャーバージョン (上の位) が一致しており、かつマイナーバージョン (下の位) がサーバで設定した値以下であれば、互換性がある要求として RPC リクエストにマッチするメソッドを実行します。このため、クライアントが送信した RPC 要求は、サーバで定義したエンドポイント `TestEndpoint` と互換性があると判断され、エンドポイントのコールバックメソッド `hoge` が実行されます。  

　それでは、実際にこれらのスクリプトを実行します。次のように 2 つのターミナルを開き、一つでサーバスクリプト `src/server.py` を実行し、もう一つでクライアントスクリプト `src/client.py` を実行してみてください。  

![実行結果](https://github.com/userlocalhost2000/draft-oslo.messaging/blob/master/img/execution_result.png?raw=true)
(左：クライアント、右：サーバ)  

　クライアント側からの呼び出しで、サーバ側で定義したエンドポイントのコールバックメソッドが呼び出され、クライアント側で実行結果が受け取れていることが確認できました。  
