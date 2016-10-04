### 実装 (ZeroMQ 編)
　章の冒頭で述べた通り oslo.messaging では、アプリケーションコードを変更せずに別の MQ を利用して同じ処理を行うことができます。ここでは、使用する MQ を `RabbitMQ` から `ZeroMQ` に変更し、先ほどの RPC スクリプトを実行します。  

　oslo.messaging で利用可能な MQ は `RabbitMQ` の他に `Kafka` や `ZeroMQ` などがありますが、デフォルトで選択される `RabbitMQ` が一般的に利用されており `ZeroMQ` などのサブドライバが利用しているパッケージは oslo.messaging の依存パッケージに含まれていません。そのため、`ZeroMQ` ドライバの依存パッケージを別途インストールしてやる必要があります。  
　以下では `ZeroMQ` ドライバを利用する上で必要な `ZeroMQ` の Python ライブラリ、及び `Redis` サーバとライブラリをインストールしています。  
```
vagrant@vagrant:~$ sudo apt-get install -y redis-server
```
```
vagrant@vagrant:~$ sudo pip install pyzmq redis
```
　ここで `ZeroMQ` ドライバに `Redis` が必要な理由を説明します。  

　`ZeroMQ` ドライバでは `Matchmaker` と呼ばれる仕組みによって、動的に追加されたホストに対する `topic` 通信が行えるようになっています。  
　何故こうした仕組みが必要なのかについて理解するために、`ZeroMQ` と `RabbitMQ` の構造の違いについて簡単に解説します。  

　以下は `RabbitMQ` を利用したプロセス間のメッセージ送信処理を表した図になります。 `RabbitMQ` には "ブローカー(Broker)" と呼ばれるメッセージを仲介する存在があります。メッセージを送信するプロセス (Publisher) はブローカーに対して送信することで、メッセージを受信するプロセス (Subscriber) がどこにどれだけ存在するかについて意識しなくてよくなります。  
　この仕組みによって、Subscriber の動的な追加 (あるいは削除) された場合でも Publisher は何も意識 (コード修正や設定変更) せずにメッセージ送受信処理を継続させることができます。  

![case of Brokered MQ](http://tsuchinoko.dmmlabs.com/wp-content/uploads/2016/06/architecture3.png)
出典：[ツチノコブログ](http://tsuchinoko.dmmlabs.com/?p=3935)

　これに対して `ZeroMQ` では、以下の図で示すように Broker が存在せず、直接 Publisher から Subscriber に対してメッセージを送る構造になっています。これによって、メッセージを低レイテンシで、且つ少ないトラフィック量で Publisher から Subscriber に送ることができます。しかしその反面、動的に Subscriber が追加 (あるいは削除) された場合、その都度 Publisher 側において送信先設定の変更を行う必要があります。  

![case of Broker-less MQ](http://tsuchinoko.dmmlabs.com/wp-content/uploads/2016/06/architecture2.png)
出典：[ツチノコブログ](http://tsuchinoko.dmmlabs.com/?p=3935)

　そこで oslo.messaging の `ZeroMQ` ドライバでは、`Matchmaker` という仕組みによってこの問題に対処しています。以下は `Matchmaker` を利用した RPC Server/Client のアーキテクチャと処理の流れを表した図になります。`Matchmaker` は RPC サーバのホスト情報を保持するハッシュテーブルを持っており、RPC サーバが起動した際に `Matchmaker` を通してサーバのホスト名とポート番号が登録されます。RPC Client は `Matchmaker` を通じてこの情報を参照することで、RPC Server 動的に追加 (あるいは削除) された場合においてもリクエストを問題なく送ることができます。  

![architecture using Matchmaker](https://github.com/userlocalhost2000/draft-oslo.messaging/blob/master/img/zeromq-driver-architecture.png?raw=true)

　また `Matchmaker` 自体がプラグイン機構 (Pluggable) になっており、多様なデータストアに対応が可能です。ただ本稿執筆時点で利用可能なプラグインが `Redis` 向けのプラグインしかないため、`Redis` サーバと `Redis` にアクセスするための Python パッケージをインストールしました。  

　では実際に `ZeroMQ` ドライバを利用して、先ほどの RPC server/client のサンプルを実行します。  
　念のため `RabbitMQ` を使わずに RPC が行えることを確認するために `RabbitMQ` を停止させます。  
```
vagrant@vagrant:~$ sudo service rabbitmq-server stop
```
　まず `ZeroMQ` ドライバを使ってサーバを起動させます。`ZeroMQ` ドライバを使う場合には、次のように `--config-file` パラメータに `ZeroMQ` ドライバを使用する設定を記述したファイル `config/driver_zmq.conf` を指定します。  
```
vagrant@vagrant:~/oslo-messaging-examples-master$ src/rpc_server.py --config-file config/driver_zmq.conf 
```
　次に `ZeroMQ` ドライバを使ってクライアントからサーバに対して RPC を実行します。クライアントも同様に `--config-file` パラメータに `ZeroMQ` ドライバ用設定ファイル `config/driver_zmq.conf` を指定します。   
```
vagrant@vagrant:~/oslo-messaging-examples-master$ src/rpc_client.py --config-file config/driver_zmq.conf 
20
vagrant@vagrant:~/oslo-messaging-examples-master$ 
```
　コード修正を行わず、設定の変更だけで `RabbitMQ` の場合と同様の結果が得られることを確認できました。  
