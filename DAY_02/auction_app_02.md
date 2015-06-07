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
- そもそもHTMLのform要素に関する知識があいまいな場合は、まずそこから理解すること。

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


