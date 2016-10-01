# アプリケーションエンジニアのための oslo.messaging 徹底解説

　本稿はアプリケーションエンジニアのための OpenStack の記事になります。ここでの内容によって、読者が携わるアプリケーションのサステーナビリティ (Sustainability) の向上に寄与できると思います。  

　OpenStack は IaaS クラウド機能を提供する OSS として知られていますが、普段アプリケーションの開発を行うエンジニアにとっては業務フィールドのレイヤの違いから縁遠い存在に思えるかもしれません。本稿ではそうしたエンジニアに向けてアプリケーションエンジニアが活用できる OpenStack プロジェクト `oslo` について紹介と、そのうちの RPC と通知機能を提供するライブラリ `oslo.messaging` の機能と使い方について徹底解説して行きます。  
　また OpenStack 自体について興味がある方は [マイクロサービスアーキテクチャが支えるOpenStackの動作と仕組み](https://codezine.jp/article/detail/9636) の連載記事も併せてご参照ください。  

---

* 1. [oslo とは？](https://github.com/userlocalhost2000/draft-codezine-oslo.messaging/tree/master/chapter1)
* 2. [oslo.messaging の概要](https://github.com/userlocalhost2000/draft-codezine-oslo.messaging/tree/master/chapter2)
* 3. oslo.messaging の環境構築方法
* 4. oslo.messaging を使ったRPC 処理の実装
    * 4-1. 実装 (RabbitMQ 編)
    * 4-2. 実装 (ZeroMQ 編)
    * 4-3. OpenStack (Nova) の利用例
* 5. oslo.messaging を使った通知処理の実装
    * 5-1. 実装
    * 5-2. OpenStack (Ceilometer) の利用例
* 6. その他の oslo プロジェクト
