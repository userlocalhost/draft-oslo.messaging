# アプリケーションエンジニアのための oslo.messaging 徹底解説

　本稿はアプリケーションエンジニアのための OpenStack の記事になります。ここでの内容によって、読者が携わるアプリケーションのサステーナビリティ (Sustainability) の向上に寄与できると思います。  

　OpenStack は IaaS クラウド機能を提供する OSS として知られていますが、普段アプリケーションの開発を行うエンジニアにとっては業務フィールドのレイヤの違いから縁遠い存在に思えるかもしれません。本稿ではそうしたエンジニアに向けてアプリケーションエンジニアが活用できる OpenStack プロジェクト `oslo` について紹介と、そのうちの RPC と通知機能を提供するライブラリ `oslo.messaging` の機能と使い方について徹底解説して行きます。  
　また OpenStack 自体について興味がある方は [マイクロサービスアーキテクチャが支えるOpenStackの動作と仕組み](https://codezine.jp/article/detail/9636) の連載記事も併せてご参照ください。  

---

* 1. [oslo とは？](https://github.com/userlocalhost2000/draft-oslo.messaging/tree/master/chapter1)
* 2. [oslo.messaging の概要](https://github.com/userlocalhost2000/draft-oslo.messaging/tree/master/chapter2)
* 3. [oslo.messaging の環境構築方法](https://github.com/userlocalhost2000/draft-oslo.messaging/tree/master/chapter3)
* 4. [oslo.messaging を使ったRPC 処理の実装](https://github.com/userlocalhost2000/draft-oslo.messaging/tree/master/chapter4)
    * 4-1. [実装 (RabbitMQ 編)](https://github.com/userlocalhost2000/draft-oslo.messaging/tree/master/chapter4/chapter4-1)
    * 4-2. [実装 (ZeroMQ 編)](https://github.com/userlocalhost2000/draft-oslo.messaging/tree/master/chapter4/chapter4-2)
    * 4-3. [OpenStack (Nova) の利用例](https://github.com/userlocalhost2000/draft-oslo.messaging/tree/master/chapter4/chapter4-3)
* 5. [oslo.messaging を使った通知処理の実装](https://github.com/userlocalhost2000/draft-oslo.messaging/tree/master/chapter5)
    * 5-1. [実装](https://github.com/userlocalhost2000/draft-oslo.messaging/tree/master/chapter5/chapter5-1)
    * 5-2. [OpenStack (Ceilometer) の利用例](https://github.com/userlocalhost2000/draft-oslo.messaging/tree/master/chapter5/chapter5-2)
* 6. その他の oslo プロジェクト
