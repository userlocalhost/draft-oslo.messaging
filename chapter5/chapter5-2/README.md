### OpenStack (Ceilometer) の利用例
　ここまで oslo.messaging の通知機能の実装方法について見てきました。ここでは OpenStack がこの通知機能をどのように利用しているかについて簡単に紹介します。  

　例えば OpenStack の管理リソースのリアルタイム検索機能を提供している [Searchlight](https://wiki.openstack.org/wiki/Searchlight) では、リソースの状態の変更を検知するために oslo.messaging を利用して、各コンポーネントからの通知を取得します。  
　[Ceilometer](http://docs.openstack.org/developer/ceilometer/architecture.html) でも oslo.messaging の通知機能を利用しています。Ceilometer はリソースの課金を行うための計量 (Metering) 機能、つまり『だれが』『いつ』『どの』リソースを『どれだけ』利用したかの情報を蓄積し、それらを標準化する機能を提供しています。  
　具体的には、以下に示す `Notification Agent(s)` によって通知を集積し、蓄積した通知を整形して外部システムと連携できる仕組みを提供しています。  
　ここでは、これらの内部の詳しい仕組みの解説は割愛させていただきます。  

![Ceilometer Architecture](http://docs.openstack.org/developer/ceilometer/_images/ceilo-arch.png)
[出典：[System Architecture — Ceilometer](http://docs.openstack.org/developer/ceilometer/architecture.html)]
