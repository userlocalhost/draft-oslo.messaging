### OpenStack (Nova) の利用例
　ここで紹介した oslo.messaging の RPC 機能の使いどころを掴むため、oslo を最も利用している OpenStack がどのように利用しているかを見て行きます。  
  
　RPC はその名の通りプロセス間通信を実現する手法の一つですが、OpenStack では RPC 以外にも REST API によるプロセス間通信の仕組みを提供しています。OpenStack ではこれら 2 つの手法をどのように使い分けているのでしょうか。  

　それを理解するために、OpenStack を構成する "コンポーネント" と "サービスプロセス" の関係について簡単に整理します。  
　以下の図は一般的な OpenStack のアーキテクチャを表した図になります。OpenStack は、図の点線の四角形で囲まれた複数の "コンポーネント" と呼ばれるプロジェクトの集合から成り立っています。図中の "OpenStack Dashboard" や "OpenStack Orchestration" がそれぞれコンポーネントを表しています。 各コンポーネントはそれぞれ独立したチームによって開発が進められており、これらが有機的に組み合わさって一つの OpenStack が成立しています。  
  
![OpenStack のアーキテクチャ](http://docs.openstack.org/admin-guide/_images/openstack-arch-kilo-logical-v1.png)  
出典：[OpenStack Docs/Logical architecture](http://docs.openstack.org/admin-guide/common/get-started-logical-architecture.html)  

　そして各コンポーネント内部には、実線の角丸の四角形で囲まれた複数の "サービスプロセス" が動作しています。図中の上段中央のコンポーネント "OpenStack Orchestration" において、"heat-api" と "heat-api-cfn"、そして "heat-engine" がサービスプロセスを表しています。  

　以降ではこれら "コンポーネント" と "サービスプロセス" において、RPC と REST API をどのように使い分けているかについて見て行きます。  
  
　まずは RPC のユースケースにを見て行きます。以下は Nova コンポーネント内部における oslo.messaging による RPC の参照関係を表した図になります。nova-network, nova-conductor, nova-scheduler, そして nova-compute のサービスプロセスにおいて RPC サーバが起動しており、実線で結ばれたサービスプロセス間で RPC が行われています。  
  
![RPC relationship in Nova](https://github.com/userlocalhost2000/draft-oslo.messaging/blob/master/img/nova-rpc-usecase.png?raw=true)  
出典：[Nova System Architecture](http://docs.openstack.org/developer/nova/architecture.html)  
  
　続いて REST API のユースケースを見て行きます。冒頭で示した OpenStack のアーキテクチャの図において、各コンポーネントがそれぞれ REST API のリクエストを処理するサーバプロセス (API サーバ) を持っていることがわかります。コンポーネント間のやりとりは全て API サーバを介して行われています。図中の太線で示されたサービスプロセスが API サーバになります。  
  
　このように OpenStack では、コンポーネント間の通信には REST API を利用し、コンポーネント内の通信には RPC を利用するといった使い分けをしています。  
　こうした使い分けはとても合理的に思えます。なぜならば、もし OpenStack がコンポーネント間の通信も RPC で行う設計になっていたとしたらどうなるか想像してみてください。ある RPC サーバに登録されているメソッドが全てのコンポーネントから参照され、あるコンポーネントのたった一つのインターフェイスの変更が OpenStack 全体に波及することになります。その場合 OpenStack ほど大規模なシステムの維持が途端に難しくなることがわかります。  
