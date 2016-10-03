### 実装
#### クライアント実装
　まずはクライアントの実装について解説します。oslo.messaging では、通知を送る主体のことを [通知人(Notifier)](http://docs.openstack.org/developer/oslo.messaging/notifier.html) と呼んでいますが、ここでは便宜上 'クライアント' という名称で解説します。以下に、クライアント処理を抜粋します。  

```Python
class NotifyClient(object):
  def __init__(self, topic='test_topic', driver='messaging', pub_id='', url=''):
    transport =  oslo_messaging.get_notification_transport(cfg.CONF, url=url)
    self.notifier = oslo_messaging.Notifier(transport, driver=driver, publisher_id=pub_id, topic=topic)
```
　NotifyClient は oslo.messaging の通知クライアントオブジェクト Notifier のラッパークラスになります。ここでは RPC client 同様に transport オブジェクトを生成した後、Notifier オブジェクトを生成します。  
　以下はクラインと処理における NotifyClient オブジェクトの生成と、メッセージの通知処理を実行するコードになります。  

```Python
  client = NotifyClient()
  client.info({'context': 'foo'}, 'event-type: hoge', {'hoge': 'abcd'})
```
 
　`NotifyClient` オブジェクトの `info` メソッドは `oslo_messaging.Notify` オブジェクトの `info` メソッドにパススルーされ、サーバに対して通知が送信されます。その際 3 つの引数を取り、左から順番に `context`, `event-type` 及び `payload` を表しています。各パラメータのデータ型と OpenStack で主に設定されている値を以下に示します。  

| パラメータ  | 型     | 主な設定値                                                                                         |
| ----------- | ------ | -------------------------------------------------------------------------------------------------- |
| context     | dict   | [oslo.context](http://docs.openstack.org/developer/oslo.context/) の `RequestContext` オブジェクト |
| event-type  | string | 通知名を表す文字列                                                                                 |
| payload     | dict   | 通知の詳細情報を表すデータ構造                                                                     |

　`context` と `payload` パラメータはそれぞれ同じ型ですが、OpenStack においては `context` パラメータにはデータを `oslo.context` の `ReqeustContext` オブジェクトでラッピングした値を設定する傾向にあります。というのも、先述した `oslo.messaging` の RPC 機能は基本的にコンポーネント内のサービス間通信で利用されているのに対して、通知機能は別のコンポーネントから参照される場合があるため、`oslo.context` を利用してデータ構造を共通化させています。こうすることで、様々なコンポーネントから送信された通知を共通のアルゴリズムで処理することができます。  
　サンプルプログラムでは説明の都合上 `oslo.context` を使わずに `dict` 型の値を指定しています。  

#### サーバ実装
　次にサーバの実装について解説します。oslo.messaging では、通知を取得する主体のことを [通知リスナ (Notification Listener)](http://docs.openstack.org/developer/oslo.messaging/notification_listener.html) と呼んでいますが、ここでは便宜上 'サーバ' という名称で解説します。以下は、サーバ処理の抜粋になります。  

```Python
def start_server(endpoints=[], topic='test_topic', url=''):
  transport = oslo_messaging.get_notification_transport(cfg.CONF, url=url)
  targets   = [oslo_messaging.Target(topic=topic)]
  server    = oslo_messaging.get_notification_listener(transport, targets, endpoints)

  try:
    server.start()
    server.wait()
```

　こちらも `rpc_server` と同様に `transport` 及び `target` オブジェクトを用意し、サーバオブジェクトを生成して起動させます。このとき、RPC サーバを生成する場合には `oslo.messaging.get_rpc_server` を呼び出しましたが、通知を受け取るサーバの場合には `oslo_messaging.get_notification_listener` を呼び出します。  
　サーバ側では、RPC 同様に通知を受け取るエンドポイントを定義し、サーバに設定します。ただし RPC の場合と異なり、エンドポイントにおいて値の返却は行いません。以下にエンドポイントの定義を抜粋します。  

```
class HogeEndpoint(object):
  filter_rule = oslo_messaging.NotificationFilter(event_type='.*: hoge$')

  def info(self, ctxt, publisher_id, event_type, payload, metadata):
    print("[HogeEndpoint] ctxt: %s" % (ctxt))
    print("[HogeEndpoint] publisher_id: %s" % (publisher_id))
    print("[HogeEndpoint] event_type: %s" % (event_type))
    print("[HogeEndpoint] payload: %s" % (payload))
    print("[HogeEndpoint] metadata: %s" % (metadata))


class FugaEndpoint(object):
  def info(self, ctxt, publisher_id, event_type, payload, metadata):
    print("[FugaEndpoint] ctxt: %s" % (ctxt))
    print("[FugaEndpoint] publisher_id: %s" % (publisher_id))
    print("[FugaEndpoint] event_type: %s" % (event_type))
    print("[FugaEndpoint] payload: %s" % (payload))
    print("[FugaEndpoint] metadata: %s" % (metadata))
```

　2 つのエンドポイント (HogeEndpoint, FugaEndpoint) を定義しています。それぞれ、受け取った通知内容について出力に表示するだけのものですが、HogeEndpoint には filter_rule が設定されています。ここではエンドポイントが受け取る通知の内容に対してフィルタ設定を行っています。  

#### 動作確認
　それでは、これら通知サーバ/クライアントスクリプトを実際に動かし、通知処理の動作を確認して行きます。  
　まずは次のように通知サーバを起動します。Transport ドライバはデフォルトの RabbitMQ ドライバを利用するため、停止している場合には起動させてから通知サーバを実行してください。  

```
vagrant@vagrant:~$ sudo service rabbitmq-server start
vagrant@vagrant:~$ cd oslo-messaging-examples/
vagrant@vagrant:~/oslo-messaging-examples$ src/notifier_server.py
```

　続いて別のターミナルを開き、次のように通知クライアントを実行します。  

```
vagrant@vagrant:~$ cd oslo-messaging-examples/
vagrant@vagrant:~/oslo-messaging-examples$ src/notifier_client.py 
```

　すると、以下のようなサーバ側のターミナルでクライアントが送った通知の中身が表示されます。  

```
[HogeEndpoint] ctxt: {u'context': u'foo'}
[HogeEndpoint] publisher_id: 
[HogeEndpoint] event_type: event-type: hoge
[HogeEndpoint] payload: {u'hoge': u'abcd'}
[HogeEndpoint] metadata: {'timestamp': u'2016-09-26 09:09:50.130070', 'message_id': u'51c23030-8ba8-48e8-bde4-05c21bd9833f'}
[FugaEndpoint] ctxt: {u'context': u'foo'}
[FugaEndpoint] publisher_id: 
[FugaEndpoint] event_type: event-type: hoge
[FugaEndpoint] payload: {u'hoge': u'abcd'}
[FugaEndpoint] metadata: {'timestamp': u'2016-09-26 09:09:50.130070', 'message_id': u'51c23030-8ba8-48e8-bde4-05c21bd9833f'}
[FugaEndpoint] ctxt: {u'context': u'bar'}
[FugaEndpoint] publisher_id: 
[FugaEndpoint] event_type: event-type: fuga
[FugaEndpoint] payload: {u'fuga': u'efgh'}
[FugaEndpoint] metadata: {'timestamp': u'2016-09-26 09:09:50.150497', 'message_id': u'187c4cd7-3102-48c9-b712-fa05ff67b306'}
```

　`HogeEndpoint` が 1 度しかよばれていないのに対して、`FugaEndpoint` が 2 回呼ばれています。これは `HogeEndpoint` に設定したフィルタ ([NotificationFilter](https://specs.openstack.org/openstack/oslo-specs/specs/kilo/notification-dispatcher-filter.html)) によるものになります。以下に `HogeEndpoint` で設定されているフィルタの設定値を改めて以下に抜粋します。  

```
class HogeEndpoint(object):
  filter_rule = oslo_messaging.NotificationFilter(event_type='event-hoge')
```

　`HogeEndpoint` では、送られてくる通知のうち `event_type` パラメータに `event-hoge` という値が設定されている通知のみを受け取るようにフィルタの設定を行っています。フィルタの設定値は、サンプルコードのような完全一致の文字列を指定しましたが、NotificationFilter のフィルタリング処理では正規表現によるマッチング処理をおこなうため `event_type` パラメータに '.*-hoge' と指定しても同様の結果が得られます。更に NotificationFilter では event_type パラメータだけでなく、通知の全てのパラメータ (`context`, `publisher_id`, `event_type`, `payload`, `metadata`) に対して同様のフィルタルールを設定できます。  
　設定したフィルタは、各エンドポイントクラス (`HogeEndpoint` や `FugaEndpoint`) のクラス変数 `filter_rule` に設定することで oslo.messaging から参照されます。`filter_rule` が設定されていない場合、当該エンドポイントは (`FugaEndpoint` のように) 全ての通知を受け付けます。  
