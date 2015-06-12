# オークションアプリ開発（２）
## 新規登録機能を開発する
データ送信のためのフォームと、フォームが投げたデータを受け取って保存するメソッドがあればよい。

### １．ルーティング開発
- フォーム用のURLと対応するアクションを先に決めておく

```ruby:config/routes.rb
get '/items/new' => 'items#new'
```

### ２．フォーム作成
- `app/views/items/new.html.erb`を新規作成し、コードを入力する。

#### 構文
```
form_for モデルオブジェクト [, オプション] do |f|
end
```

#### 記入
```html:app/views/items/new.html.erb
<h1>商品登録フォーム</h1>
<%= form_for @item do |f| %>
  <p>
    商品名：<br />
    <%= f.text_field :name %>
  </p>
  <p>
    商品詳細：<br />
    <%= f.text_area :description %>
  </p>
  <p>
    価格：<br />
    <%= number_field :price %>
  </p>
  <p>
    売り主ＩＤ：<br />
    <%= f.text_field :seller_id %>
  </p>
  <p>
    電子メール：<br />
    <%= f.text_field :email %>
    </p>
  <p>
    商品画像ＵＲＬ：<br />
    <%= f.text_field :image_url %>
  </p>
  <%= f.submit "登録" %>
<% end %>
```

- [Railsドキュメント フォーム](http://railsdoc.com/form#%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88%E3%83%9C%E3%83%83%E3%82%AF%E3%82%B9(text_field))に、どのようなフォーム要素が使用できるかは書いてある。
- そもそもHTMLのform要素に関する知識があいまいな場合は、まずそこから学習すること。

### ３．コントローラ開発
- この段階では、フォームで使っている@itemオブジェクトが存在しない（`nil`）
- だから、フォーム[http://localhost:3000/items/new](http://localhost:3000/items/new)にアクセスするとエラーになる。
- フォーム用のオブジェクトは、フォーム表示前に作成される必要がある
- `new`メソッドのあとにフォームが作成される
	- `new`メソッド内で`@item`オブジェクトを作らなくてはいけない
	- この`@item`オブジェクトはフォームの受け皿用なので、空っぽでOK
	
```ruby
def new
  @item = Item.new
end
```

- formがコントローラに投げた（Post）したデータを確認してみる。
- まずはformのソースを確認する

```
<form class="new_item" id="new_item" action="/items" accept-charset="UTF-8" method="post"><input name="utf8" type="hidden" value="&#x2713;" /><input type="hidden" name="authenticity_token" value="UUMv8XXjQCy4wb4oexvGnYS6crGkdLam+mv6JXswBWXc2MoGzbdClzpMt9kov5SPcwt20OHhSE+z+ZdAuJ1eVg==" />
  <p>
    商品名：<br />
    <input type="text" name="item[name]" id="item_name" />
  </p>
  <p>
    商品詳細：<br />
    <textarea name="item[description]" id="item_description">
</textarea>
  </p>
  <p>
    価格：<br />
    <input type="number" name="item[price]" id="item_price" />
  </p>
  <p>
    売り主ＩＤ：<br />
    <input type="text" name="item[seller_id]" id="item_seller_id" />
  </p>
  <p>
    電子メール：<br />
    <input type="text" name="item[email]" id="item_email" />
    </p>
  <p>
    商品画像ＵＲＬ：<br />
    <input type="text" name="item[image_url]" id="item_image_url" />
  </p>
  <input type="submit" name="commit" value="登録" />
</form>
```

- このformをsubmitすると、次のデータがPostされる（データ内容はその都度違う）

```json
{
	"utf8"=>"✓",
	"authenticity_token"=>"qyphPzsbBD/HAeZU/N6sY8Bh4OC09LqVzKSOsxa+bSUmsYTIg08GhEWM76Wvev5xN9DkgfFhRHyFNuPW1RM2Fg==",
	"item" => {
		"name"=>"スタンスミス",
		"description"=>"オールホワイトレザーのテニスシューズ",
		"price"=>"15000", "seller_id"=>"32",
		"email"=>"nashirox@gmail.com",
		"image_url"=>"http://img.abc-mart.net/img/goods/C/52016700011.jpg"
	},
	 "commit"=>"登録"
}
```

- このデータを、受け取り側のコントローラー（今回は`ItemsController`）は`params[:item]`で取得できる
- `params[:item]`はコントローラーで次のようなハッシュ形式で使える

```
{"name"=>"スタンスミス", "description"=>"オールホワイトレザーのテニスシューズ", "price"=>"15000", "seller_id"=>"32", "email"=>"nashirox@gmail.com", "image_url"=>"http://img.abc-mart.net/img/goods/C/52016700011.jpg"}
```

- コントローラーはこの`params[:item]`をモデルに渡して「保存させる」
	- `@item = Item.new(params[:item])`
	- ところが、それができたのはRails 3系まで
- Rails 4はStrong Prametersを通して、値を渡すこと

```ruby
class ItemsController < ApplicationController

  def create
    @item = Item.new(item_params)
    # 上の行はインスタンスを作っただけ。saveメソッドで保存しないと、消える。
    @item.save
  end

  private

  # params.require(key).permit(filter)
  # key
  #   Strong Parameters を適用したい params の key を指定する。
  # filter
  #   Strong Parameters で許可するカラムを指定する。

  def item_params
    params.require(:item).permit(:name, :price, :seller_id, :description, :email, :image_url)
  end
end
```

## 編集機能を開発する（課題）
商品の内容編集を実装する

- コントローラー
	- `edit`アクションと`update`アクションを設定する
		- それぞれ`new`アクションと`create`アクションと似ている
	- 違うのは、編集する`item`の`id`をルートから取得し、既存データを検索する
	- 検索されたデータを更新する
- ビュー
	- `edit`のビューを作る。`update`アクション後は、`show`のビューにリダイレクトするので不要
	- `form_for`メソッドは、データ送信先に`update`アクションを指定しないといけない。
		- `<%= form_for @item, url: { action: :update } do |f| %>`

## 応用課題
- 売り主モデルを作成し、商品詳細ページに表示されるようにする
	- id、name、prefecture、tel
- 売り主の平均評価機能を実装する
	- `item`モデルに`rate:integer`を追加
	- 5段階評価なので、データベースは1〜5までしか受け付けないようにする
	- ビューで評価を登録できるようにする（プルダウンで選択）
	- 売り主の`show`ページでは下記の機能を実装する
		- 出品アイテム毎の評価を平均して表示させる
		- 平均が表示できたら、ビューヘルパーを自作し、評価値から★を表示する機能をつける
