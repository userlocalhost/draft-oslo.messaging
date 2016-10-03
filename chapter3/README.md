## oslo.messaging の環境構築方法
　oslo.messaging の使い方を実際に動かしながら理解するために oslo.messaging の実行環境の構築を行います。ここでは、vagrant で構築した Ubuntu16.04 環境に oslo.messaging の実行環境を構築する方法を紹介します。  
　vagrant のインストール方法につきましては、[過去の記事](http://codezine.jp/article/detail/8255?p=2) で詳しく解説されておりますのでご参照ください。ここでは、以下の通り VirtualBox と vagrant がお使いの環境にインストールされた状態から解説をします。  
``` 
$ VBoxManage --version
5.0.26r108824
$ vagrant -v
Vagrant 1.8.5
$ 
```
　使用する Vagrant の Box は、[chef が公開している](https://github.com/chef/bento) モノを利用します。vagrant がインストールされた PC で以下のコマンドを実行すると Ubuntu16.04 環境が構築されます。  
```
$ vagrant init bento/ubuntu-16.04; vagrant up --provider virtualbox
```
　構築後は以下のコマンドで、構築した環境にログインできます。以降の操作は構築した Ubuntu16.04 環境内で実行してください。  
```
$ vagrant ssh
```
　まずは必要なパッケージのインストールを行います。  
```
vagrant@vagrant:~$ sudo apt-get update; sudo apt-get install -y rabbitmq-server python-pip unzip
```
　次に oslo.messaging と依存パッケージのインストールを行います。  
```
vagrant@vagrant:~$ sudo pip install oslo.messaging
```
　最後に oslo.messaging による RPC 及び通知を行うサンプルコードを取得します。サンプルは以下のリポジトリのモノを利用します。  

　- [https://github.com/userlocalhost2000/oslo-messaging-examples](https://github.com/userlocalhost2000/oslo-messaging-examples)  

　次のコマンドでリポジトリの最新版を取得します。  
```
vagrant@vagrant:~$ wget https://github.com/userlocalhost2000/oslo-messaging-examples/archive/master.zip
vagrant@vagrant:~$ unzip master.zip -d examples 
```
　以上で、oslo.messaging を動かす環境が整いました。以降では、oslo.messaging がどのように使われており、またどのように使うかについてサンプルを用いて解説して行きます。  
