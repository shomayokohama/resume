# スパルタブクログ
- ペパボ社の[ブクログ](http://booklog.jp/)をプチクローン
	- ユーザーが本のレビューを投稿できるサービス
- 多対多のアソシエーションを学習する
- ネストされたルーティングを学習する

## プロジェクト開始
```bash
$ rails new sparta_booklog
$ cd sparta_booklog
```

## `book`と`user`周りを作る
### `book`モデル開発
- scaffoldでお手軽に
- 属性は本のタイトル、値段、出版社の３つだけ

```bash
$ rails g scaffold book title:string price:integer publisher:string
```

### `user`モデル開発
- deviseで作る

```Gemfile
gem "devise"
```

```bash
$ bundle install
$ rails g devise:install
```

- ログアウト時のリダイレクト先として、root_urlの設定が必要

```ruby:config/routes.rb
root 'books#index'
```

- deviseで管理されるuserモデルの追加

```bash
$ rails g devise User
$ rake db:migrate
```

- 下準備はこれで完了。サーバーは再起動してください。

### ビューを整える
- ヘッダーなどをつける
- 顧客管理システムと同じ
	- `app/views/layouts/application.html.erb`
	- `app/views/layouts/_header.html.erb`

## `review`周りを作る
- `book`は複数の`review`を持っている
- `customer`は複数の`review`を持っている

### モデル開発
#### migrationまで
- `book`は複数の`review`を持っている
	- ＝ `review`は`book`を参照している
- `user`は複数の`review`を持っている
	- ＝ `review`は`user `を参照している

```bash
$ rails g model review body:text book:references user:references
```

- 生成されるmigrationスクリプトに手を加える

```ruby:db/migrate/xxxx
class CreateReviews < ActiveRecord::Migration
  def change
    create_table :reviews do |t|
      t.text :body, null: false
      t.references :book, index: true, foreign_key: true
      t.references :user, index: true, foreign_key: true

      t.timestamps null: false
    end
    # 下記を追加
    add_index :reviews, [:book_id, :user_id], unique: true
  end
end
```

- `add_index`の設定がポイント
- 今回、一人のユーザーは一つのレビューしか投稿できない設計
- つまり、`reviews`テーブルでは`book_id`と`user_id`の組み合わせが一対しか存在してはいけない
- なお、一人のユーザーが複数レビューが投稿できる設計の場合、この設定は不要

```bash
$ rake db:migrate
```

#### アソシエーション設定
- `has_many` + `through`オプション
- 参考：[Active Record の関連付け (アソシエーション) — Rails ガイド](http://railsguides.jp/association_basics.html#has-many-through%E9%96%A2%E9%80%A3%E4%BB%98%E3%81%91)

```ruby:app/models/book.rb
class Book < ActiveRecord::Base
  has_many :reviews
  has_many :users, through: :reviews
end
```

```ruby:app/models/review.rb
class Review < ActiveRecord::Base
  belongs_to :book
  belongs_to :user
end
```

```ruby:app/models/user.rb
class User < ActiveRecord::Base
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :trackable, :validatable
  has_many :reviews
  has_many :books, through: :reviews
end
```

### ルート開発
- `review`のルーティングの設定は素直にいくと下記

```ruby:config/routes.rb
Rails.application.routes.draw do
  resources :reviews
end
```

- 上でもできるが、`book`と`review`の関係を考えると、下記がより適切
- 参考：[2.7 ネストしたリソース（Rails のルーティング） — Rails ガイド](http://railsguides.jp/routing.html#%E3%83%8D%E3%82%B9%E3%83%88%E3%81%97%E3%81%9F%E3%83%AA%E3%82%BD%E3%83%BC%E3%82%B9)

```ruby:config/routes.rb
Rails.application.routes.draw do
  resources :books do
    resources :reviews
  end
end
```

- ルーティングをネストする (入れ子にする) ことで、親子関係をルーティングで表す
- ルート例：`/books/105/reviews/new`
	- `id`が105の本のレビューを新規作成する
- 「何の本」のレビューかが、URLからわかる

### コントローラー開発
- 続いて、`review`のコントローラーを作る

```bash
$ rails g controller reviews
```

```ruby:app/controllers/reviews_controller.rb
class ReviewsController < ApplicationController

  def new
  end

end
```

### ビューを作る
- `new`から
- formはrenderする

```html:app/reviews/new.html.erb
<div class="row">
  <div class="col-lg-12">
    <h1>ニューレビュー登録</h1>
    <ul>
      <li>書籍名：<%= @book.title %></li>
      <li>価格：<%= @book.price %></li>
      <li>出版社：<%= @book.publisher %></li>
    </ul>

    <hr>
    <%= render 'form' %>
  </div>
</div>
```

- 空だった`new`メソッド作る
- ルーティングで、どの本に対する`review`を新しく作るのかがわかる

```ruby:app/controllers/reviews_controller.rb
class ReviewsController < ApplicationController
  def new
    @book = Book.find(params[:book_id])
    @review = Review.new
  end
end
```

- ネストしたルーティングのformは、`form_for`の指定で`[親モデル, 子モデル]`の配列で指定するのが肝

```html:app/views/reviews/_form.html.erb
<%= form_for([@book, @review]) do |f| %>
  <div class="form-group">
    <%= f.label :body, 'レビュー' %><br>
    <%= f.text_area :body, class: "form-control" %>
  </div>
  <%= f.submit %>
<% end %>
```

- `@book`と`@review`を同時に更新するフォームなので、`form_for`は２つのモデルオブジェクトを指定している
- `hidden_field`が送っている値に注意
- 受け取った後の`create`メソッドを作る

```ruby:app/controllers/reviews_controller.rb
class ReviewsController < ApplicationController
  before_action :authenticate_user!, only: [:new, :create]

  def new
    @book = Book.find(params[:book_id])
    @review = Review.new
  end

  def create
    @review = Review.new(
      book_id: params[:book_id],
      user_id: current_user.id,
      body: review_params["body"]
    )
    if @review.save
      redirect_to book_url(@review.book)
    else
      render :new
    end
  end

  private

  def review_params
    params.require(:review).permit(:body)
  end

end
```

- `create`されたら、`customers#show`にリダイレクト
- 戻された後、関連レビューをすべて表示させる

```html:app/views/reviews/show.html.erb
（省略）
    <%= link_to '編集', edit_book_path(@book) %> |
    <%= link_to '戻る', books_path %> |
    <%= link_to 'レビューを書く', new_book_review_path(@book) %> |

    <hr>

    <h2>レビュー一覧</h2>
    <% @book.reviews.each do |review| %>
      <p>
        <%= review.body %> <br>
        by <%= review.user.email %>
      </p>
    <% end %>
    <hr>
    <h2>レビューを残したユーザー一覧</h2>
    <%# throughオプションのおかげで、BookはReviewを介してUserにアクセスできる  %>
    <% @book.users.each do |user| %>
      <p><%= user.email %></p>
    <% end %>
  </div>
</div>
```

- これで完成
