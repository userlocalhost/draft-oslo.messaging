## その他の oslo プロジェクト
　ここまで oslo.messaging が提供する RPC と通知機能について詳しく解説してきました。ここまでの内容を駆使することで oslo.messaging を十分に使いこなせると思います。これによって、皆さんが開発するアプリケーションのサステーナビリティの向上に寄与できれば幸いです。  
　最後に oslo.messaging 以外の oslo プロジェクトについてピックアップしたものを以下に示します。  

| プロジェクト名 | 概要 |
| -------------- | ---- |
|[oslo.log](http://docs.openstack.org/developer/oslo.log/)|ログの整形、出力先設定などロギング周り全般の機能を提供。OpenStack の各コンポーネント全般で利用されている。|
|[oslo.db](http://docs.openstack.org/developer/oslo.db/)|MySQL や PostgreSQL などの RDBMS に対して共通のインターフェイスからアクセスできる仕組みを提供。|
|[cliff](http://docs.openstack.org/developer/cliff)|CLI アプリケーションのフレームワーク。コマンドライン引数の解析に加え `git` や `svn` などのような多数のサブコマンドを持つ CLI アプリケーションを開発する上で役立つ。|
|[stevedore](docs.openstack.org/developer/stevedore/)|setuptools の entry_points で指定されたモジュールの動的な呼び出しをサポートするライブラリ。実行時にロードするライブラリを選択するといったプラグイン機構を提供する多くの OpenStack コンポーネントで利用されている。|
　
　皆さんも是非 OpenStack プロジェクトの成果物を活用してみてください。  
