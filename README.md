# アプリケーションエンジニアのための oslo.messaging 徹底解説

　本稿はアプリケーションエンジニアのための OpenStack の記事になります。ここでの内容によって、読者が携わるアプリケーションのミドルウェア周りの実装や、アプリケーション自体のサステーナビリティ (Sustainability) の向上に寄与できると考えております。  

　OpenStack は IaaS クラウド機能を提供する OSS として知られていますが、アプリケーションエンジニアにとっては業務フィールドのレイヤが違うため関心が薄いかもしれませんが、本稿ではそうしたエンジニアに向けてアプリケーションエンジニアが活用できる OpenStack プロジェクト 'Oslo' について紹介と、そのうちの RPC と通知機能を提供するライブラリ 'oslo.messaging' の機能と使い方について徹底解説して行きます。  
　尚 OpenStack 自体について興味がある方は [OpenStack に関する別の連載記事](https://codezine.jp/article/detail/9636) も参照してみてください。  

　本稿ではアプリケーションエンジニアの読者向けに OpenStack の共通ライブラリの一つ oslo.messaging の使い方について徹底解説して行きます。  

---

* 1. oslo とは？
* 2. oslo.messaging の概要
* 3. oslo.messaging の環境構築方法
* 4. oslo.messaging を使ったRPC 処理の実装
    * 4-1. 実装 (RabbitMQ 編)
    * 4-2. 実装 (ZeroMQ 編)
    * 4-3. OpenStack (Nova) の利用例
* 5. oslo.messaging を使った通知処理の実装
    * 5-1. 実装
    * 5-2. OpenStack (Ceilometer) の利用例
* 6. その他の oslo プロジェクト
