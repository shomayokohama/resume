スパルタ顧客管理システム（３）
## 対応履歴を追加
### 要件
- `user`が`customer`の対応履歴を残せるようにする
	- コメントをつけるだけ
- 各`customer`の`show`で履歴の登録や閲覧ができるようにする

### 設計
#### ビュー設計
- `app/views/customers/show.html.erb`の中だけでコメント投稿・閲覧ができるようにする

#### モデル設計
- `comment`モデルという名前にする
- 一人の`customer `は複数の`comment`を持つ
	- `customer`からは`has_many :comments`
	- `comment`から見ると`belongs_to :customer`

#### コントローラー設計
- `comments`コントローラーに必要なメソッド
	- `create`：コメントを保存する
	- `destroy`：コメントを削除する
- とりあえずこの２つ。コメント編集はおいておく。

## 実装
### モデル開発
- コメント本文は`body`とし、`customer`を参照するようにする

```bash
$ rails g model comment body:text customer:references
$ rake db:migrate
```

### コントローラー開発

```bash
$ rails g controller comments
```

- まずは空のメソッドを作る

```ruby:app/controllers/comments_controller.rb
class CommentsController < ApplicationController
  def create
    
  end

  def destroy
    
  end
end
```

### ビュー開発
- `customer`の`show`にコメント投稿フォームを作る
- その前に、`customer`の`show`メソッドに、コメント用の空オブジェクトを作っておく


```ruby:app/controllers/customers_controller.rb
class CustomersController < ApplicationController
  def show
    @customer = Customer.find(params[:id])
    @comment = Comment.new # これをform_forで使う
end
```

- `<hr>`以降を追加している

```app/views/customers/show.html.erb
<div class="row">
  <div class="col-lg-12">
    <h1>顧客詳細</h1>
    <dl>
      <dt>お名前</dt>
      <dd><%= @customer.full_name %></dd>
      <dt>電子メール</dt>
      <dd><%= @customer.email %></dd>
    </dl>
  </div>
</div>

<hr />

<div class="row">
  <div class="col-lg-12">
    <%= form_for(@comment, url: { controller: :comments, action: :create }) do |f| %>
      <div class="form-group">
        <%= f.label :body , '対応内容' %><br>
        <%= f.text_area :body, class: "form-control" %>
      </div>
      <%= f.hidden_field :customer_id, value: @customer.id %>
      <%= f.submit %>
    <% end %>
  </div>
</div>
```

- `form_for`について
	- `form_for`は何もオプションを指定しないと、テンプレートが所属しているコントローラーのデータを新規作成しようとする
		- 上のテンプレートだと、新しい`comment`データが`customer#create`に投げられるわけですね
	- `comment`データの新規登録は`comment#create`で行われるべき！
	- そのため、`url`オプションで投げ先を指定してあげる
		- `url: { controller: :コントローラー名, action: :アクション名 }`
	- 参考：[フォーム(form) - Railsドキュメント](http://railsdoc.com/form)
- `f.hidden_field`について
	- `comment`の登録には、コメント本文である`body`と、「誰に対する」コメントであるかを示す`cusotmer_id`が必要
	- データを投げるためにはなにがしかの`xxx_field`を作らないといけないけど、`text_field`だと`cusotmer_id`が変更できちゃう。
	- そこで、隠しフィールドにしてデータを送信している
	- 参考：[INPUT type="hidden"－HTMLタグリファレンス](http://www.htmq.com/html/input_hidden.shtml)
	- ３日目のツアーガイド管理システムでやった

### ルート開発
- さっきのフォームは`comments#create`に投げるよう指定したけど、ルートを設定していない
	- formとしては、`/comments`に`POST`で投げようとしている
- ルートは二通り書ける

```ruby:config/routes.rb
post '/comments' => 'comments#create'
```

- 上より下にした方が、いろいろ便利
	- `comment_path`とか使える

```ruby:config/routes.rb
# post '/comments' => 'comments#create'
resources :comments, only: [:create]
```

- `resources`メソッドもそうだけど、Railsは便利な反面、いろいろ隠している。
- だから原理的に何やってるかを押さえておくかは悪いことではない

### `comments#create`メソッド開発
- フォームがデータ投げた先に何も処理を書いてないので、書いていく

```ruby:app/controllers/comments_controller.rb
class CommentsController < ApplicationController

  def create
    @comment = Comment.new(comment_params)
    if @comment.save
      redirect_to customer_url(@comment.customer_id)
    else
      redirect_to customer_url(@comment.customer_id)
    end
  end

  (省略)

  private

  def comment_params
    params.require(:comment).permit(:body, :customer_id)
  end
end
```

- これまで何回か書いてきた`create`メソッドなので、全体として解説することはあまりない
- 注意することは、保存後のリダイレクト先
- 何も指定しないと、保存後に`comments#create`の`create.html.erb`に行こうとする
- でも、`comments`コントローラーはビューがない
- コメントが投稿されたページにリダイレクトさせたい
- そのためにはまず、保存されたコメントが「どの`customer`のものか？」を調べなければいけない
	- `@comment.customer_id`でコメントが所属する`customer`を調べる
- `customer_url(customerのid)`で、`http://localhost:3000/customers/id`というURLが生成される
	- ちなみに同種で`customer_path(customerのid)`もあるけど、これは相対パスである`/customers/id`を作る
	- Viewの中で使うのは`xxx_path`、コントローラーで使うのは`xxx_url`

### ValidationとAssociationを設定
- 一人の`customer `は複数の`comment`を持つ
- これを忘れずに！

```ruby:app/models/customer.rb
class Customer < ActiveRecord::Base
  (省略)
  belongs_to :company
  has_many :comments # 追加
  (省略)
end
```

```ruby:app/models/comment.rb
class Comment < ActiveRecord::Base
  validates :body,        presence: true
  validates :customer_id, presence: true

  belongs_to :customer
end
```

## `customers#show`で`comments`を表示する
### あるべきビューの姿を考える
- こんなん

```app/views/customers/show.html.erb
<div class="row">
  <div class="col-lg-12">
    <h2>対応履歴</h2>
    <% @comments.each do |comment| %>
      <p>
        <%= comment.body %>
      </p>
    <% end %>
  </div>
</div>
```
- `customers#show`の中で、表示されている`customer`の全コメントが`@comments`に格納されている状態
- `@comments`の中身を`each`で取り出して、`body`を表示させれば全コメントが出力される
- `@comments`はまだどこにもない。どこに作る？何をどうやって入れる？

### コントローラー改変（１）
- `customers#show`に`@comments`を作る
 
```ruby:app/controllers/customers_controller.rb
class CustomersController < ApplicationController
  def show
    @customer = Customer.find(params[:id])
    @comment = Comment.new
    @comments = Comment.where(customer_id: params[:id].to_i) # 追加
  end
end
```
- 【復習】今いる`customer`の`id`は`params[:id]`でとれる
	- `http://localhost:3000/customers/305`
	- params[:id] => "305"
- クエリメソッド`モデル.where(カラム名: 値)`でデータベースを検索してオブジェクトを返す
	- ただし、`params[:id]`は文字列型、`customer_id `は数値型なので検索できない
	- `params[:id]`を`to_i`メソッドで数値型に変換する
- これで`customers#show`のビューを見ると、コメントが表示される

### コントローラー改変（２）
- しかし、Associationを設定しているので、もっと簡単で直感的に置き換えることが出来る
 
```ruby:app/controllers/customers_controller.rb
class CustomersController < ApplicationController
  def show
    @customer = Customer.find(params[:id])
    @comment = Comment.new
    # @comments = Comment.where(customer_id: params[:id].to_i)
    @comments = @customer.comments
  end
end
```

- `@customer.comments`で`@customer`に関連づけられている`comments`を呼び出すことが出来る
	- `customer`は`has_many :comments`なので、複数形で呼び出していることに注意

### ビュー開発
- Bootstrapを使って、ビューは次のようにする
	- 参考：[Media object - Bootstrap公式](http://getbootstrap.com/components/#media)
- `customer`の詳細表示とコメント投稿フォームの間

```app/views/customers/show.html.erb
(省略)
<div class="row">
  <div class="col-lg-12">
    <h2>対応履歴</h2>
    <% @comments.each do |comment| %>
      <div class="media">
        <div class="media-body">
          <p><%= simple_format(comment.body) %></p>
          <p class="text-muted">
            投稿日時：<%= comment.created_at.strftime('%Y/%m/%d %H:%M') %>
          </p>
        </div>
      </div>
    <% end %>
  </div>
</div>
(省略)
```

## コメント削除
### ルート開発
- `only`オプションに追加

```ruby:config/routes.rb
resources :comments, only: [:create, :destroy]
```
### ビュー開発
- `link_to`メソッドに`method`オプションで`delete`をつける

```app/views/customers/show.html.erb
(省略)
<div class="media-body">
  <p><%= simple_format(comment.body) %></p>
  <p class="text-muted">
    投稿日時：<%= comment.created_at.strftime('%Y/%m/%d %H:%M') %>
    <%= link_to "削除する", comment_path(comment), method: :delete %>
  </p>
</div>
(省略)
```
### コントローラー開発
- 基本的にこんな感じ

```ruby:app/controllers/comments_controller.rb
class CommentsController < ApplicationController
(省略)
  def destroy
    @comment = Comment.find(params[:id])
    @comment.destroy
    # リダイレクト先はどうする？
  end
(省略)
end
```

- ここで問題は、削除後のリダイレクト先
- このままでは`app/views/comments/destroy.html.erb`をRailsは探しに行ってしまう
	- そんなのないし
- 元の`customer`ページに戻したい
- でも、`@comment.destroy`でコメントデータが消去されるから、誰のコメントだったかがわからなくなっちゃう

```ruby:app/controllers/comments_controller.rb
class CommentsController < ApplicationController
(省略)
  def destroy
    @comment = Comment.find(params[:id])
    # @commentがdestroyされる前に、commentが誰のものかを変数に保存する
    customer_id = @comment.customer_id
    @comment.destroy
    # さっき保存したcustomer_idをここで使う
    redirect_to customer_url(customer_id)
  end
(省略)
end
```
- 上記のようにして解決

## 課題
- 一覧画面の各行にコメント数が表示されるようにする
	- ヒント：[count (Array) - Rubyリファレンス](http://ref.xaio.jp/ruby/classes/array/count)
- `comment`モデルに`user_id`を追加し、誰が投稿したコメントかわかるようにする
- `user`のイメージ画像`image_url`を登録できるようにする
- 投稿フォームや、`user`の投稿したコメントの横に、イメージ画像が表示されるようにする
- 一覧画面でコメントも検索できるようにする

以上
