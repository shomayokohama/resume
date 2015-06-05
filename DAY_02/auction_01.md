# オークションアプリ開発（１）
## プロジェクト作成

```bash
$ rails new auction
$ cd auction
```

## モデル開発

### データ構造設計

商品名／商品詳細／価格／売り手ID／電子メール／イメージ画像URL

### モデル生成
モデルの名前には単数形を使用する。

```bash
$ rails g model item name:string description:text price:integer seller_id:integer email:string image_url:string
```

### 生成ファイルを確認
rails g modelで作成されるファイルは「２つ」

#### model
```ruby:app/models/item.rb
class Item < ActiveRecord::Base
end
```

#### migration
db/migrate/.._create_items.rb

### migration実行
```bash
$ rake db:migrate
```

### テーブル内容の確認
sqliteのビューアーをインストールして確認する。

id/created_at/updated_atの３つが**マジックカラム**

## コントローラ開発
モデルが生成したデータを扱うコード、それがコントローラ。

### コントローラ生成
モデルと同じ名前にする必要があるが、コントローラは複数形にする必要がある。

```bash
$ rails g controller items
```

### 生成ファイルを確認

```ruby:app/models/items_controller.rb
# rubyのクラス
class ItemsController < ApplicationController

# この間に処理内容を書いていく

end
```

- コントローラー名はキャメルケース
	- `ItemsController`
- ファイル名はスネークケース
	- `items_controller.rb`
- Railsは名前を頼りに処理をしているので、できるだけ触ってはいけない！

## ビュー開発
- ビューは埋込みRuby（ERB：Embedded Ruby）によってテンプレートが作成されている
- `app/views/items/show.html.erb`をに書いてみる

```html
<p>
  <b>商品名：</b>
</p>

<p>
  <b>商品詳細：</b>
</p>

<p>
  <b>価格：</b>
</p>

<p>
  <b>売り手ID：</b>
</p>

<p>
  <b>電子メール：</b>
</p>

```

- `$ rails s`で`http://localhost:3000/items/5`を確認

## ルート開発
- さきほどのviewはエラーとなる。
	- ルーティングが定義されていないため
- `config/routes.rb`でルーティングを定義する

```ruby
Rails.application.routes.draw do
  # 追加
  get '/items/:id' => 'items#show'

  # コメントがいっぱいあるけど省略
end
```

- `$ rails s`で`http://localhost:3000/items/5`を確認
- ページ表示されるようになったが、商品情報がない

## コントローラ開発（２）
- ビューにidと紐ついた商品情報を渡さないと、商品情報は表示されない

```ruby
class ItemsController < ApplicationController
  def show
    #Itemというモデルに対してfindメソッドでidを検索させている
    @item = Item.find(params[:id])
  end
end
```

- `items/show.html.erb`で、`@item.name`や`@item.price`のようにアクセスできる。

## ビュー開発（２）
- コントローラーが用意した`@item`オブジェクトの値を表示させるように修正する。
- 埋め込みスクリプト`<%= %>`を使用する

```html
<p>
  <b>商品名：<%= @item.name %></b>
</p>

<p>
  <b>商品詳細：<%= @item.description %></b>
</p>

<p>
  <b>価格：<%= @item.price %></b>
</p>

<p>
  <b>売り手ID：<%= @item.seller_id %></b>
</p>

<p>
  <b>電子メール：<%= @item.email %></b>
</p>

<p>
  <img src="<%= @item.image_url %>">
</p>
```

## indexページを作成する
一覧ページを作成する

### コントローラ開発
`app/controller/items_controller.rb`に追加

```ruby
def index
  @items = Item.all
end
```

### ビュー開発
`app/view/items/index.html.erb`に追加

```html
<h1>出品商品一覧</h1>
<li><a href="/items/<%= @items[0].id %>"><%= @items[0].name %></a></li>
<li><a href="/items/<%= @items[1].id %>"><%= @items[1].name %></a></li>
<li><a href="/items/<%= @items[2].id %>"><%= @items[2].name %></a></li>
```

- eachメソッドを使うとよりスッキリかける。
	- `<% %>`で囲われていることに注意
	- eachそのものはhtml出力の必要がないため

```html
<h1>出品商品一覧</h1>
<% @items.each do |item| %>
  <li><a href="/items/<%= item.id %>"><%= item.name %></a></li>
<% end %>
```

### ルーティング開発

```ruby
get '/items' => 'items#index'
```
