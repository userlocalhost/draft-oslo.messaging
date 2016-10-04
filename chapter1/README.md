## oslo とは？
　[oslo](https://wiki.openstack.org/wiki/Oslo) は OpenStack の各コンポーネント (OpenStack の最も大きなソフトウェアモジュールの単位) で共通で利用するライブラリを集めたプロジェクトです。  
　その昔の OpenStack では設定ファイルやコマンドライン引数の解析、メッセージパッシング、ロギングなどといった処理が各コンポーネント毎に個別で存在しており、開発・メンテナンスコストが掛かることが問題になっていました。  
　そして 2013 年にリリースされた OpenStack のメジャーバージョン [Grizzly](https://wiki.openstack.org/wiki/ReleaseNotes/Grizzly) で共通ライブラリ群を提供する olso プロジェクトが発足し、同じ年にリリースされた [Havana](https://wiki.openstack.org/wiki/ReleaseNotes/Havana) で設定ファイルとコマンドライン引数の解析を行なう [oslo.config](http://docs.openstack.org/developer/oslo.config) と、通知や RPC の仕組みを提供する [oslo.messaging](http://docs.openstack.org/developer/oslo.messaging) がリリースされました。現在では、国際化や並行処理など、本稿執筆時点で 34 のプロジェクトが存在しています。  
　余談ですが、'olso' プロジェクトのネーミングは、イスラエルとパレスチナの和平協定 'オスロ合意' にちなんで付けられたという [経緯](http://docs.openstack.org/project-team-guide/oslo.html#brief-history) があったりします。  

　oslo 自体は OpenStack のプロジェクトですが、用途は OpenStack に限定されません。  
　例えば oslo.config は、設定ファイルとコマンドライン引数の解析を行なうライブラリとして広く知られており、[SDN Framework Ryu](https://osrg.github.io/ryu/) などで採用されています。  
　他にもロギング処理全般を行う [olso.log](http://docs.openstack.org/developer/oslo.log/) や CLI フレームワークの [cliff](http://docs.openstack.org/developer/cliff/) などが有名で、[OpenStack ユーザ会のメンバーによる解説](http://www.slideshare.net/h-saito/openstack-oslo-cliff) などもあります。  

　本稿で取り上げる oslo.messaging は、[RabbitMQ](rabbitmq.com) や [ZeroMQ](http://zeromq.org/) などの MQ システムを介した通知や RPC の仕組みを提供します。  
