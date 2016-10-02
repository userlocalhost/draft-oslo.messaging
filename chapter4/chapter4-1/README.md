### 実装 (RabbitMQ 編)
　各データ構造の役割と関連がわかったところで、サンプルコードを見ながら実際に動かしてみます。  
　まずはサーバ側のコード `src/rpc_server.py` について簡単に解説します。以下はコードの抜粋になります。  

```Python
    10	class TestEndpoint(object):
    11	  target = oslo_messaging.Target(namespace='foo', version='1.2')
    12	
    13	  def hoge(self, ctx, arg):
    14	    print("[TestEndpoint] hoge(%s, %d) is called" % (ctx, arg))
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
  
　24 行目の get_rpc_server メソッドの呼び出しで Server オブジェクトを生成します。RPCClient からの RPC 要求に対して endpoints パラメータで指定したオブジェクトにマッチするメソッドを呼び出します。その際、エンドポイントに target が設定されている場合には、namespace と version ネゴシエーションの確認を行います。target が指定されていない場合には、デフォルトの namespace (=None) と version (=1.0) が内部的に設定されます。  
　コールバックメソッド `hoge()` は `ctx` と `arg` の２つのパラメータを取り、それぞれユーザ定義のオブジェクト (ディレクトリ型) と引数を表します。そして引数 `arg` の値を 2 倍してクライアントに結果を返すという内容の処理を行います。  
  
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
...
    28	  print(send_request({}, 10))
```

　28 行目でユーザ定義の RPCClient ラッパーの hoge を呼び出し、内部的に 16 行目でサーバに対して hoge メソッドの呼び出しを行っています。その際 15 行目の prepare() メソッドの呼び出しで RPCClient オブジェクトが内部で持つ Target オブジェクトの namespace と version パラメータを上書きします。  
　version パラメータがサーバ側のエンドポイント hoge() で指定した値と一致していないことに気づいたかもしれません。Server 側のバージョンネゴシエーション処理では、クライアントからの要求 version のメジャーバージョン (上の位) が一致しており、かつマイナーバージョン (下の位) がサーバで設定した値以下であれば、互換性がある要求として当該メソッドを実行し結果を返します。  

　それでは、実際にこれらを実行してみます。次のようにターミナルからサーバスクリプト `src/server.py` を実行し、次の別のターミナルを開いてクライアントスクリプト `src/client.py` を実行してみてください。  

![実行結果](https://gist.github.com/userlocalhost2000/627c0f16516fb9a351b68a494751128c/raw/e1d8bde469f70cdfdf728bb04743a5bc22ccb9d2/execution_result.png)
(左：クライアント、右：サーバ)  

　クライアント側からの呼び出しでサーバ側で定義したエンドポイント hoge() が呼び出され、クライアント側で実行結果が受け取られていることが確認できました。  

