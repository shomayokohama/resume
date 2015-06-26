# Railsアプリのデプロイ実習
- WEBアプリをサーバに設置し、利用可能にすることをデプロイと呼ぶ
- サーバ毎にインストールされているソフトウェアやそのバージョンが異なるので、デプロイはすんなりいくことはなく、難所となりがち。
- 書籍にはほとんど載っていないので、気合いで習得してください

## デプロイの流れ概略
1. サーバを用意
2. 環境構築
3. Railsプロジェクトの設定＠ローカル
4. Railsプロジェクトをサーバにクローンする
5. Railsプロジェクト設定＠サーバ
6. ＼（＾o＾）／

## サーバを用意
### サーバ関連サービスの種類
- [レンタルサーバーとVPSそしてクラウド](http://www.slideshare.net/SeijiSu/vps-15403569)
- レンタルサーバ
	- Railsが動くレンタルサーバはほぼない
- VPS
	- レンタルサーバに管理者権限がついているようなもの
- クラウド
	- いろいろ便利。お金が許せばこっち。
- PaaS（Platform as a Service）
	- すでに環境構築が済んだサーバみたいなもの
	- 海外では[Heroku](https://www.heroku.com/)、日本ではペパボ社の[Sqale](http://sqale.jp/)が有名
	- 環境構築が不要だけど、少々お高い。でも便利。
	- 実験的にリリースしたり、受託でクライアントがOKくれるならこっち使う方が楽

### VPS借りる
今回の実習では、一番基本の環境構築→デプロイをする

- [GMOクラウドVPS](http://vps.gmocloud.com/order/trial.html)を利用する
	- ２週間お試し無料
	- 【注意！】実習後、アカウントマネージャーから解約の手続きをしてください。お試し期間最終日までに解約手続きが行われないと自動的に本契約となり、利用料金が発生します。
- 申込み手続き
	- マイクロプラン
		- メモリ1GBがRailsアプリの最低ライン。
		- ストレージはあまり必要ない
	- OSはCentOS 6.2 
	- ドメイン名は「ドメイン名を利用しないでIPアドレスで利用する。」
	- 最後に電話番号での認証が必要
- 管理画面ログイン
	- 登録メルアドにログイン情報が届くので、ログインしてみる
	- 契約状態が「トライアル契約」になるまで数分待つ

### サーバにログイン
- サーバー設定が完了すると、ログインに必要な情報が届く

```GMOクラウドVPSサーバー設定完了のお知らせ：sub0000541132.hmk-temp.com
(省略)
———————————————————————————————— 
■サーバー設定情報 
———————————————————————————————— 
メインドメイン名：sub0000541132.hmk-temp.com 
IPアドレス 　　：153.122.55.166

管理者ID ：root 
管理者パスワード：rxG7cQG24ooF
(省略)
```

- IPアドレスとrootのパスワードがあればサーバにSSHでログインできる
- パスワード方式でのSSH接続コマンドは`$ ssh root@XXX.XXX.XXX.XXX -p 22`

```bash
$ ssh root@153.122.55.166 -p 22
The authenticity of host '153.122.55.166 (153.122.55.166)' can't be established.
RSA key fingerprint is 9f:c2:a7:e2:48:62:e2:e6:b9:6c:af:1b:ad:4e:17:c0.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '153.122.55.166' (RSA) to the list of known hosts.
root@153.122.55.166's password: 
[root@sub0000541132 ~]# 
```

- `[root@sub0000541132 ~]# `は違うホストにターミナルが移動していることを示す
- こうなればログイン成功
- なお、ログインはパスワード方式の他に公開鍵方式もある。講義の時間上省くけど、セキュリティ上、普通はそっちの方式でログインする
- 以降、`$`表記はサーバ上でのコマンドを表す

## 環境構築
### 環境構築の全体像
- Railsアプリの標準的なミドルウェアは次のようなもの。
	- 様々なアプリケーションに対して特定の機能を提供するソフトウェアをミドルウェアという
		- アプリの下、OSの上に位置しているため、ミドルウェア
	- WEBサーバソフトやデータベースソフトが典型的なミドルウェア
- それぞれ順を追ってインストール、設定をしていく。

![全体構成](http://re.buildinsider.net/web/rubyonrails4/0102/1-5.gif)
出典：[Railsを利用するための環境設定［Ruby、Windows、Linux］ - Build Insider](http://www.buildinsider.net/web/rubyonrails4/0102)

### 初期アップデート
- サーバにはじめから入っているミドルウェアをアップデートする
	- サーバ借りたら毎回これをやっておく
- `yum` はLinuxのパッケージ管理システム
	- Rubyのパッケージ管理システム`gem` の先祖みたいなもの

```bash
$ sudo yum update
```

- ダウンロードするコマンドの`wget`も使うのでインストールしておく

```bash
$ sudo yum install wget
```

### Apache を入れる
#### インストール
- はじめに[Apache](http://httpd.apache.org/)がインストールされていることを確認する。
	- Apacheとは、世界中でもっとも多く使われているWebサーバソフトウェア
	- システム上は`httpd`という名前になっている

```bash
$ yum list installed
```

- `httpd`が見当たらないので、インストール。

```bash
$ sudo yum -y install httpd
```

- インストールされたか確認する

```bash
$ httpd -v
Server version: Apache/2.2.15 (Unix)
Server built:   Oct 16 2014 14:48:21
```

#### 起動とサービス登録
- インストール後、Apache を起動する。
- `service` コマンドに`httpd`を指定し、`start` オプションをつけて実行する。

```bash
$ sudo service httpd start
httpd を起動中:                                            [  OK  ]
```

- 起動後、ブラウザでアクセスしてみる。
- http:// の後にサーバーの IP アドレスを指定したものが URL となる。これをブラウザのアドレスバーに入力して開く。
- Apache のテストページが表示されればOK
- 確認後、サーバー OS が起動されたとき、Apache のサービスが自動起動されるようにしておく。

```bash
$ sudo chkconfig httpd on
```

- 設定を確認すると、いくつかの項目が on になっている。

```bash
$ chkconfig --list httpd
httpd           0:off   1:off   2:on    3:on    4:on    5:on    6:off
```

- 各項目の数値は、ランレベルと呼ばれるシステムの動作モードを示している
- 覚えなくていいけど、2〜5がonになっていれば、OS再起動と同時に Apache も起動される。

|ランレベル|意味|
|:--|:--|
|0|シャットダウン ( システム停止中 )|
|1|シングル ユーザー モード ( root のみ )|
|2|マルチ ユーザー モード ( ネットワークなし )|
|3|マルチ ユーザー モード ( テキスト )|
|4|未使用|
|5|マルチ ユーザー モード ( グラフィカル )|
|6|システム再起動|

### MySQLを入れる
#### インストール
- MySQLとは、データベースソフト
	- Railsの開発環境ではSQLite3をデータベースソフトとして使ってる
	- 本番環境ではMySQLを使う
- データベースは別サーバで独立させることが多い
	- 全ての卵を同じバスケットに盛らない
- 今回はアプリ（Rails本体）とデータベース（MySQL）を同一サーバ内に共存させる
- yum から入れる。

```bash
$ sudo yum install mysql-server
```

- インストールしたらバージョンを確認しておく。

```bash
$ mysql --version
mysql  Ver 14.14 Distrib 5.1.73, for redhat-linux-gnu (x86_64) using readline 5.1
```

#### 文字コード設定
- MySQL の文字コードを設定する。
- 設定ファイルは /etc/my.cnf となる。

```bash
$ cd
# ユーザー HOME の backup ディレクトリへバックアップを取る
$ cp /etc/my.cnf $HOME/backup
$ sudo vi /etc/my.cnf
```

- 開いたファイルを以下のように編集する。

```/etc/my.cnf
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
user=mysql
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
 
# 既定の文字コードを UTF-8 に変更
default-character-set=utf8
# クライアントの文字コードを無視
skip-character-set-client-handshake
 
# 以下は機能ごとの文字コード設定
[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
 
[client]
default-character-set=utf8
 
[mysql]
default-character-set=utf8
 
[mysqldump]
default-character-set=utf8
```

#### 起動とサービス登録

- MySQL を起動する。

```bash
$ sudo service mysqld start
MySQL データベースを初期化中:  Installing MySQL system tables...
... 中略 ...
mysqld を起動中:                                           [  OK  ]
```

- Apacheと同じように、MySQL も自動起動されるようにする。

```bash
$ sudo chkconfig mysqld on
$ chkconfig --list mysqld
mysqld          0:off   1:off   2:on    3:on    4:on    5:on    6:off
```

- ランレベル 2 ～ 5 が on になったので、OK。

#### root ユーザーにパスワードを設定
- `mysqladmin` コマンドでMySQL のユーザーにパスワードを設定する。
- root ユーザーを対象とする場合、以下のようにする。

```bash
$ sudo mysqladmin -u root password '自分のパスワード'
```

- 正しく設定されたことを確認するために、root でログインしてみる。

```bash
$ mysql -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.1.61 Source distribution
 
Copyright (c) 2000, 2011, Oracle and/or its affiliates. All rights reserved.
 
Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.
 
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
 
mysql>
```

- パスワードが通り、root ユーザーでログインできた。

- プロンプトが mysql> になっているときは、MySQL のコマンドラインツールが起動している。この状態では通常のシェルコマンドを受け付けないので注意する。
- 抜ける場合は `quit` と入力して Enter キーを押すか、Ctrl + D キーを押す。

### Rubyをインストールする
- 当たり前だが、サーバでRubyが動かないとRailsも動かない
- プログラミング言語もインストールするという概念がある
- Ruby をインストールするために必要なパッケージと関連ツールをまとめてインストールする

```bash
$ sudo yum install openssl-devel readline-devel zlib-devel curl-devel libyaml-devel mysql-devel httpd-devel ImageMagick ImageMagick-devel gcc
```

- Ruby 本体の最新版ソースを入手する。
- [公式ページのダウンロード](https://www.ruby-lang.org/ja/downloads/0)から、ソースを固めたファイルの URL を調べておく。
- 最新安定版は2.2.2なのでこれを使う
	- http://cache.ruby-lang.org/pub/ruby/2.2/ruby-2.2.2.tar.gz

```bash
$ wget http://cache.ruby-lang.org/pub/ruby/2.2/ruby-2.2.2.tar.gz
# tar zxvfで解凍できる
$ tar zxvf ruby-2.2.2.tar.gz
# ダウンロードした圧縮ファイルは削除
$ rm ruby-2.2.2.tar.gz 
rm: remove 通常ファイル `ruby-2.2.2.tar.gz'? yes
```

- 解凍後のファイルを、コンパイルする
	- ソースコードを解析し、コンピュータが直接実行可能な形式のプログラムに変換すること。
- `make`は10分近くかかるからトイレいってもよい
	- なお、`make`はめちゃCPUパワー使うので、他にサービスが動いてる環境で動かしてはダメ。
	- サービス停止アナウンスした上で`make`

```bash
$ cd ruby-2.2.2
$ ./configure
$ make
$ make install
```

- 各コマンドの意味
	- `configure`は、これからインストールを行うOSのCPUやバージョンを調べたり、必要な関連ツールを調査する。そして，これらの状況を記述した`Makefile`ファイルを作成する。
	- `make`は、`configure`で作成された`Makefile`を基にしてソースコードをコンパイルする。
	- `make install`は、`make`で生成されたバイナリファイルなどを規定のディレクトリにコピー（インストール）を行う。
- 全部終わったら、rubyが入ったか確認

```bash
$ ruby -v
ruby 2.2.2p95 (2015-04-13 revision 50295) [x86_64-linux]
```

- きちんとインストールされてるので、不要になった Ruby のソースを消しておく。

```bash
$ cd ~
$ rm -rf ruby-2.2.2
```

### Bundler と Passengerを入れる
- [Bundler](http://bundler.io/)はGem管理ツール

```bash
$ gem install bundler --no-rdoc --no-ri
```

- [Phusion Passenger](http://www.modrails.com/)はApache上でRails アプリを動かすために使用する。

```bash
$ gem install passenger --no-rdoc --no-ri
```

- Passenger の Apache 用モジュールをインストール。

```bash
$ yum install gcc-c++ # 下記のコマンドで必要になる
$ passenger-install-apache2-module

This installer will guide you through the entire installation process. It
shouldn't take more than 3 minutes in total.

Here's what you can expect from the installation process:

 1. The Apache 2 module will be installed for you.
 2. You'll learn how to configure Apache.
 3. You'll learn how to deploy a Ruby on Rails application.

Don't worry if anything goes wrong. This installer will advise you on how to
solve any problems.

Press Enter to continue, or Ctrl-C to abort.

# ここでEnter押しておく
--------------------------------------------

Which languages are you interested in?

Use <space> to select.
If the menu doesn't display correctly, press '!'

   ⬢  Ruby
 ‣ ⬡  Python
   ⬡  Node.js
   ⬡  Meteor
```

- いろいろ設定し始めるので、3分ほど待つ

```bash
--------------------------------------------
Almost there!

Please edit your Apache configuration file, and add these lines:

### ↓から
   LoadModule passenger_module /usr/local/lib/ruby/gems/2.2.0/gems/passenger-5.0.11/buildout/apache2/mod_passenger.so
   <IfModule mod_passenger.c>
     PassengerRoot /usr/local/lib/ruby/gems/2.2.0/gems/passenger-5.0.11
     PassengerDefaultRuby /usr/local/bin/ruby
   </IfModule>
### ↑までコピーしてテキストファイルに保存

After you restart Apache, you are ready to deploy any number of web
applications on Apache, with a minimum amount of configuration!

Press ENTER when you are done editing.
```

- 終わると上の表示がでるので、指定部分をテキストファイルにコピーペして保存

```bash
Detected 0 error(s), 1 warning(s).
Press ENTER to continue.


--------------------------------------------

Deploying a web application: an example

Suppose you have a web application in /somewhere. Add a virtual host to your
Apache configuration file and set its DocumentRoot to /somewhere/public:

### ↓から
   <VirtualHost *:80>
      ServerName www.yourhost.com
      # !!! Be sure to point DocumentRoot to 'public'!
      DocumentRoot /somewhere/public    
      <Directory /somewhere/public>
         # This relaxes Apache security settings.
         AllowOverride all
         # MultiViews must be turned off.
         Options -MultiViews
         # Uncomment this if you're on Apache >= 2.4:
         #Require all granted
      </Directory>
   </VirtualHost>
### ↑までコピーしてテキストファイルに保存

And that's it! You may also want to check the Users Guide for security and
optimization tips, troubleshooting and other useful information:

  /usr/local/lib/ruby/gems/2.2.0/gems/passenger-5.0.11/doc/Users guide Apache.html
  https://www.phusionpassenger.com/documentation/Users%20guide%20Apache.html

Enjoy Phusion Passenger, a product of Phusion (www.phusion.nl) :-)
https://www.phusionpassenger.com

Phusion Passenger is a trademark of Hongli Lai & Ninh Bui.
```

- コピペした「最初」のテキストを、`passenger.conf`に記述する。

```bash
$ sudo vi /etc/httpd/conf.d/passenger.conf
```

```/etc/httpd/conf.d/passenger.conf
LoadModule passenger_module /usr/local/lib/ruby/gems/2.2.0/gems/passenger-5.0.11/buildout/apache2/mod_passenger.so
<IfModule mod_passenger.c>
  PassengerRoot /usr/local/lib/ruby/gems/2.2.0/gems/passenger-5.0.11
  PassengerDefaultRuby /usr/local/bin/ruby
</IfModule>
```

```bash
$ sudo vi /etc/httpd/conf.d/rails.conf
```

- 「次に」コピペしたテキストを、`rails.conf` に記述する。
- 【超重要】テキストは修正が必要
	1. ServerName にIPアドレスを記述
		- 独自ドメインを取ったらそっちを書いておく 
	2.  `/somewhere/public`となっている箇所を全部、Railsアプリを設置するディレクトリ付きで書き換える
		- 今回は`/var/lib/sparta-tabelog/public`
		- 【上級知識】Passengerを組み込んだApacheではDocumentRootを`/railsまでのパス/$RAILS_ROOT/public`に設定することで、自動的にRailsアプリケーションが認識される

```/etc/httpd/conf.d/rails.conf
<VirtualHost *:80>
   ServerName 153.122.55.166
   # !!! Be sure to point DocumentRoot to 'public'!
   DocumentRoot /var/lib/sparta-tabelog/public
   <Directory /var/lib/sparta-tabelog/public>
      # This relaxes Apache security settings.
      AllowOverride all
      # MultiViews must be turned off.
      Options -MultiViews
      # Uncomment this if you're on Apache >= 2.4:
      #Require all granted
   </Directory>
</VirtualHost>
```

- Apache の設定チェックと再起動をおこない、`passenger.conf` と `rails.conf`を読ませる。

```bash
$ sudo apachectl configtest
Syntax OK
$ sudo service httpd restart
httpd を停止中:                                            [  OK  ]
httpd を起動中:                                            [  OK  ]
```

### gitインストール
- GithubからRailsプロジェクトをプルするために、そもそも`git`のインストールが必要

```bash
$ yum install git
（中略）
$ git --version
git version 1.7.1
```

### MySQL 上に Railsアプリ 専用のデータベースとユーザーを作成
- まずは`root`ユーザーでログイン

```bash
$ mysql -u root -p
Enter password: 
```

- 今回は`tabelog`という名前のデータベースを作る
- Railsアプリの内容に沿った名前をつける

```mysql
mysql> CREATE DATABASE tabelog DEFAULT CHARACTER SET utf8;
Query OK, 1 row affected (0.00 sec)
```

- これでRailsアプリ`tabelog` 専用のデータベースができた
- 次は`tabelog`データベースの管理者ユーザーを作る
	- `nashirox@localhost`はユーザー名
	- 'spartanashiro'はパスワード
	- 自分のID／パスワードに変える

```mysql
mysql> GRANT ALL PRIVILEGES ON tabelog.* TO nashirox@localhost IDENTIFIED BY 'spartanashiro';
Query OK, 0 rows affected (0.00 sec)
mysql> exit;
```

- ミドルウェア周り終了。

## Railsプロジェクトの設定＠ローカル
次に、サーバから手元に戻って、サーバで設定した項目をRailsに書き加えていく

### `database.yml`の修正
- データベースとの接続設定を管理するファイル
- 本番環境でのMySQL設定内容を記載する

```yaml:config/database.yml
# SQLite version 3.x
#   gem install sqlite3
#
#   Ensure the SQLite 3 gem is defined in your Gemfile
#   gem 'sqlite3'
#
default: &default
  adapter: sqlite3
  pool: 5
  timeout: 5000

development:
  <<: *default
  database: db/development.sqlite3

# Warning: The database defined as "test" will be erased and
# re-generated from your development database when you run "rake".
# Do not set this db to the same as development or production.
test:
  <<: *default
  database: db/test.sqlite3

# 以降で追加：GMOクラウド用設定
production:
  adapter: mysql2
  encoding: utf8
  database: tabelog
  port: 3306
  username: nashirox
  password: spartanashiro
  host: localhost
```

- MySQLのポート番号はデフォルトで`3306`
- `username`と`password`は自分で設定した情報

### `Gemfile`の修正
- 本番環境用のgemに`mysql2`を追加
- 本番環境で不用意なgemアップデートによる不具合を招かないためにも、gemバージョンは固定する方がよい

```Gemfile
（中略）
# therubyracerのコメントアウトも外す
gem 'therubyracer', platforms: :ruby
（中略）
group :production do
  gem 'mysql2', '= 0.3.16'
end
```

- `gem sqlite3`が全環境設定になっていないか再チェック！！！！
	- `mysql2`と衝突する
	- `sqlite3`は`:development, :test`下の環境に押し込める

### git push
- 手元の変更をすべてコミットして、masterにpushする

## Railsプロジェクトをサーバにクローンする
### 公開鍵を作ってGithubに登録
- 再度サーバにログインする
- Githubからクローンするには、要求元の公開鍵を作って登録しなければいけない
	- 初日にやったことと同じ
- 公開鍵と秘密鍵のペア作成は`ssh-keygen`コマンド
	- コマンドを打ったら、`Enter`連打

```bash
$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
f4:5f:50:b4:f7:be:7b:79:8e:04:4a:34:e0:03:ea:f4 root@sub0000541132.hmk-temp.com
The key's randomart image is:
+--[ RSA 2048]----+
|      . .    .o  |
|     . o .   . . |
|    o   + o . . .|
|   o . . + . . ..|
|    . E S o . . .|
|         . o o . |
|          . . . o|
|             . o+|
|              .+=|
+-----------------+
```

- 自分の公開鍵をコピーする
	- `/root/.ssh/id_rsa.pub`が公開鍵

```bash
$ cat /root/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAxBSHdxGSb4TigS5NOu2PvZLT/q9szm/iHxL/nBadN76HNve280cxSqzb0Nvg0lCBtP2xDtpVRLJ6Uuh+XiJI62rYvq7E2ovQ+vCEJgAG8+Yau9KkK2vEDrcKssbikH6jE6FVcS/woNMNkWEPN6ptqEvWRUk4CIvTxEDnXRSIOAPBIxUm7fG8EHbLTzGSjeNfGV3RtiXnjgc0wEIyuPzu7RThGjvzkA/lYqua3yu6DC+My+QrN/1l7F8AAZr7es7Yhtm0L0ZF48lBdeQbHgyOugEeevtEb1/vs9mwuX4Q7BSzkhnVfK4SQAtK9RU2WEk72ScuoddJwP2pTsL2lVsC1w== root@sub0000541132.hmk-temp.com
```

- GithubのRailsリポジトリへいく
- Settings -> Deploy keys -> Add deploy key -> Keyに公開鍵をコピペし、Add keyボタンを押す
- これで準備完了
- あと、右端にある`SSH clone URL`をコピーしておく

### サーバにリポジトリをクローンする
- サーバの`/var/lib/`にいく。ここにRailsプロジェクトを設置する
- 移動したら、`git clone リポジトリ`でサーバ上にダウンロードする

```bash
$ cd /var/lib/
$ git clone git@github.com:sparta-camp/sparta-tabelog.git
Warning: Permanently added the RSA host key for IP address '192.30.252.128' to the list of known hosts.
remote: Counting objects: 170, done.
remote: Compressing objects: 100% (114/114), done.
remote: Total 170 (delta 43), reused 170 (delta 43), pack-reused 0
Receiving objects: 100% (170/170), 38.82 KiB, done.
Resolving deltas: 100% (43/43), done.
```

- これでサーバ上にRailsアプリがダウンロードされた
- もう少し設定しないと動かない

## Railsプロジェクト設定＠サーバ
### bundler経由でgemをインストール
- とりあえずアプリルートフォルダに移動

```bash
$ cd sparta-tabelog
```

- そして`bundle install`で、`Gemfile`に記載されたgemを全てインストールする
	- `--without`オプションで`development`と`test`はよけることに注意！！

```bash
$ bundle install --without development test
```

- しかし、失敗してしまう。本来はこのコマンド通るはずなのに。
	- `nokogiri-1.6.6.2 for inspection.`のエラー 
- 現在のライブラリバージョン構成が原因。
- 環境構築はこういったエラーが手を変え品を変え出てくるので、原因究明をがんばってほしい
- 今回は答えを探し当てることが出来た
	- http://qiita.com/chobi9999/items/7b66677b3bf3da8acc14

```bash
$ yum -y install libxml2 libxslt libxml2-devel libxslt-devel
$ gem install nokogiri -- --use-system-libraries
```

- 気を取り直してもう一回`bundle install`

```bash
$ bundle install --without development test
（中略）
Bundle complete! 21 Gemfile dependencies, 54 gems now installed.
Gems in the groups development and test were not installed.
Use `bundle show [gemname]` to see where a bundled gem is installed.
```

- 成功！

### セッション管理用の秘密鍵を生成
- `secrets.yml`というところで、環境変数で秘密鍵を設定しなければいけないので準備する

```yaml:config/secrets.yml
（中略）
production:
  secret_key_base: <%= ENV["SECRET_KEY_BASE"] %>
```

- `~/.tabelog_secret_key_base`というディレクトリ／ファイル名で秘密鍵を作る

```bash
$ ruby -e 'require "securerandom"; print SecureRandom.hex(64)' > ~/.tabelog_secret_key_base
```

- 確認してみるけど、一応管理者以外に見ちゃダメな値です！

```
$ cat ~/.tabelog_secret_key_base
9089965b78f99758fe015ad7ce9819526fd815d8d16e8df76466cedc69d4135129e0afc552ef7f043c722752b4b3e8631438166199a87f6f3f2184ae46e4ee71
```

- これを環境変数にセットする
- 設定は`~/.bash_profile`に書き込む

```bash
$ vi ~/.bash_profile
```

- ファイルが開いたら、一番下に追加する

```~/.bash_profile
# .bash_profile

# Get the aliases and functions
if [ -f ~/.bashrc ]; then
        . ~/.bashrc
fi

# User specific environment and startup programs

PATH=$PATH:$HOME/bin

export PATH
export SECRET_KEY_BASE=`cat ~/.tabelog_secret_key_base`
export DEVISE_SECRET_KEY=`cat ~/.devise_secret_key`
```

- 環境変数は再起動しないと読み込まれない

```bash
$ reboot

Broadcast message from root@sub0000541132.hmk-temp.com
	(/dev/pts/4) at 13:07 ...

The system is going down for reboot NOW!
[root@sub0000541132 sparta-tabelog]# Connection to 153.122.55.166 closed by remote host.
Connection to 153.122.55.166 closed.
```

### データベースを初期化
- データベースの中身は空っぽなので`db:migrate`する
- この辺、かなりこけやすいので気持ちに余裕を持って臨もう。

```bash
$ cd /var/lib/
$ rake db:migrate RAILS_ENV=production
```

### seedファイルの流し込み
- 今回は用意していないが、`seed.rb`を作ってある場合は `db:migrate` 成功後のタイミングで流し込む

```bash
# 今回は不要だよ
$ rake db:seed RAILS_ENV=production
```

### アセットプリコンパイル
- Asset Pipelineという機能により、RailsはCSSとJSファイルを一つ結合して圧縮している
- 開発環境では自動でやっているんだけど、本番環境では手動で動かさなければいけない

```bash
$ rake assets:precompile RAILS_ENV=production
```

### Rails再起動
- Railsは自身を再起動するまで変更を反映してくれない
	- サーバの再起動じゃないよ
- 未だになぜこんな再起動方法なのか疑問に思ってるけど、次のコマンドで再起動される

```bash
$ sudo touch tmp/restart.txt
```

再起動がうまくいけば、デプロイ完了。

## サイト上で確認
- 設定したIPアドレスか独自ドメインをブラウザで打って表示されたら終わり
- ダメだったら次の項を読む

## トラブルシューティング
- デプロイ作業はこけまくるので、各種ログを読んで対処していくことがめちゃ重要
- また、デプロイ後もこけるので、やっぱりログを読んで対処していく
- ログを読むためのコマンドは`cat` か `less`
- 以下に主要なログ置き場を記す

### Apacheのログ
#### エラーログ
- `/etc/httpd/logs/error_log`にある
- 一番基本。ここ読めばだいたい原因がわかる

#### アクセスログ
- `/etc/httpd/logs/access_log`にある
- トラブル関連というよりは、怪しいアクセスが記録されるので確認する


### Railsのログ
- `$Rails_Root/log/production.log`にある
- Railsアプリへのアクセスごとに情報が記録される

以上

### 以下、未整理

```bash
$ sudo ln -s /var/lib/sparta-tabelog/public /var/www/html/
```
