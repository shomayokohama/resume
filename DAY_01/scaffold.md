# 初めてのRailsアプリ - Scaffold編

## サンプルアプリ
### チケット販売管理システム FUJI
このシステムができること一覧

1. 販売済チケットを一覧表示する
2. 新規のチケット情報を登録する
3. 各チケットの情報を表示する
4. 各チケットの情報を更新する
5. 各チケットを削除する

## Railsプロジェクトの新規作成
### `rails new` コマンド
```bash
$ rails new fuji
```

### 各ディレクトリの意味と重要度

重要度| ディレクトリ  | 意味
-----|------- | ------
★★★ |app | モデル、ビュー、コントローラ、ヘルパーなどを含む主要なアプリケーションコード
★☆☆ |bin | バイナリ実行可能ファイル
★★☆ |config | アプリケーションの設定
★★★ |db | データベース関連のファイル
★☆☆ |doc | マニュアルなど、アプリケーションのドキュメント
★☆☆ |lib | 自作のライブラリモジュール
★☆☆ |log | アプリケーションのログファイル
★☆☆ |public | 公開データ（エラーページなど）
★☆☆ |test | アプリケーションのテスト 
★☆☆ |tmp | 一時ファイル
★★☆ |vendor | サードパーティのプラグインやgemなど
★★★ |Gemfile | このアプリケーションに必要なGemの定義ファイル
★☆☆ |Gemfile.lock | アプリケーションのすべてのコピーが同じgemのバージョンを使用していることを確認するために使用されるgemのリスト

### bundlerでgemをインストール
```bash
$ bundle install
```

Gemfileで指定したgem（Rubyの便利ツール）がインストールされる。

### ローカルホストで確認
```bash
$ cd fuji # 必ずRailsアプリフォルダの中に入ること
$ rails server
```

- コマンド入力後、ブラウザで[http://localhost:3000/](http://localhost:3000/)にアクセス
- `control + c`でサーバはストップ
- `rails s`でもOK

## Rails アプリの設計
### データベース設計
今回のアプリが必要とする情報一覧

1. 購入者の氏名
2. 価格
3. メールアドレス
4. テントサイトの利用有無
5. 備考

### Railsで使用できるデータ型一覧
データ型  | 意味
------- | ------
string | 文字列
text |  長い文字列
integer | 整数
float | 浮動小数
decimal | 制度の高い小数
binary | バイナリデータ
date | 日付
time | 時間
datetime | 日時
timestamp | より細かい日時
boolean | 真偽値

## Railsアプリの構築
### `rails scaffold`コマンド
CRUDアプリを一気に構築してくれるコマンド。

#### 構文
```bash
$ rails generate scaffold モデルの名前 データ名:データの型
```

#### 実行結果
```bash
nashirox$ rails generate scaffold ticket name:string price:integer email:string tent:boolean note:text
      invoke  active_record
      create    db/migrate/20150528071442_create_tickets.rb
      create    app/models/ticket.rb
      invoke    test_unit
      create      test/models/ticket_test.rb
      create      test/fixtures/tickets.yml
      invoke  resource_route
       route    resources :tickets
      invoke  scaffold_controller
      create    app/controllers/tickets_controller.rb
      invoke    erb
      create      app/views/tickets
      create      app/views/tickets/index.html.erb
      create      app/views/tickets/edit.html.erb
      create      app/views/tickets/show.html.erb
      create      app/views/tickets/new.html.erb
      create      app/views/tickets/_form.html.erb
      invoke    test_unit
      create      test/controllers/tickets_controller_test.rb
      invoke    helper
      create      app/helpers/tickets_helper.rb
      invoke      test_unit
      invoke    jbuilder
      create      app/views/tickets/index.json.jbuilder
      create      app/views/tickets/show.json.jbuilder
      invoke  assets
      invoke    coffee
      create      app/assets/javascripts/tickets.coffee
      invoke    scss
      create      app/assets/stylesheets/tickets.scss
      invoke  scss
      create    app/assets/stylesheets/scaffolds.scss
```

### migrationを実行する
実際にデータベースを作らなければいけない。
#### 構文
```bash
$ rake db:migrate
```

#### 実行結果
```bash
$ rake db:migrate
== 20150528071442 CreateTickets: migrating ====================================
-- create_table(:tickets)
   -> 0.0021s
== 20150528071442 CreateTickets: migrated (0.0022s) ===========================
```

### ローカルサーバを立ち上げ、確認する

```bash
$ rails s
```

[http://localhost:3000/](http://localhost:3000/)にアクセス。