# コントローラー＆ビュー開発の基礎
## MVCとは
- Railsは、MVCパターン（Model - View - Controllerパターン）と呼ばれるアーキテクチャを採用しています。
- MVCパターンとは、アプリケーションを次の役割で明確に分離しようという設計モデルです。
	-  Model（ビジネスロジック）
	-  View（ユーザインターフェイス）
	-  Controller（ModelとViewの制御）

![](http://re.buildinsider.net/web/rubyonrails4/0101/1-3.gif)

## VCだけのRailsアプリ開発（１）
- いっきにやると消化不良に陥るので、VCだけのアプリ作ってみる
	- つまり、データの保存や取り出しはできないアプリ（Modelが担当なので）
- `http://localhost:3000/tests/message`にアクセスすると、挨拶してくれるアプリを作る。

### コントローラー作成
話をシンプルにするために、まずはModel抜きでアプリを作ってみる

```bash
$ rails new ctlr_test
$ cd ctlr_test
```

- まずは、コントローラーのみを生成する。
- `rails generate controller コントローラ名`が構文
- 詳しくは[Railsドキュメント](http://railsdoc.com/rails#%E3%82%B3%E3%83%B3%E3%83%88%E3%83%AD%E3%83%BC%E3%83%A9%E3%81%AE%E7%94%9F%E6%88%90(rails%20generate%20controller))を読んでください。

```bash
$ rails generate controller tests
```

なお、`rails generate`で自動生成したファイルは`rails destroy`でまとめて削除ができる。

```ruby
class TestsController < ApplicationController
  def message
    @message = "Testsコントローラの中のindexアクションからこんにちは！！！"
  end
end
```

### ビュー作成
- アクションと対応した名前のビューが必要。
- `app/views/tests/message.html.erb`を新規作成する。

```html
<h1>ご挨拶</h1>
<p><%= @message %></p>
```

- ビューの中から、コントローラのインスタンス変数を呼び出せる。
	- `@`のついていないローカル変数は呼び出せない。

### ルート開発
- `http://localhost:3000/tests/message`、つまり`/tests/message`にアクセスしたさいに、testsコントローラのmessageアクションが呼び出されるようにする。
- ルーティングを司るファイルは`config/routes.rb`。これ以降、ルート開発といったらここをいじります。

```ruby
Rails.application.routes.draw do
  get 'tests/message' => 'tests#message'
end
```

これで挨拶文が出る。

## VCだけのRailsアプリ開発（２）
- 実際のRailsアプリは、さまざまな変数がURL内に埋め込まれている
- Railsで開発されている[Cookpad](http://cookpad.com/recipe/2287349)の例
- ルートでシンボル指定することで、Railsはそれを解釈できる

```ruby
Rails.application.routes.draw do
  get 'tests/:id' => 'tests#show'
end
```

'tests#show'の中身

```ruby
  def show
    @id = params[:id]
  end
```

コントローラーはこうやってユーザーのリクエスト（＝URL）を受け取って、処理を行う。

## 演習
- `http://localhost:3000/name/nashirodaisuke`にアクセスすると次のように表示されるようにしましょう

```
こんにちは！nashirodaisukeさん
```

- 表示されたら、`http://localhost:3000/name/nashiro/daisuke`にアクセスすると次のように表示するよう変更しましょう

```
あなたの姓はnashiro、
名前はdaisukeさんですね
```
