# スパルタ顧客管理システム（１）

## モデル初期開発
### `customer`モデル作成

```bash
$ rails g model customer family_name:string given_name:string email:string
```

- migrationスクリプトを編集

```ruby:db/migrate/yyyymmddhhss_create_customers.rb
class CreateCustomers < ActiveRecord::Migration
  def change
    create_table :customers do |t|
      t.string :family_name, null: false, length: { maximum: 20 }
      t.string :given_name,  null: false, length: { maximum: 20 }
      t.string :email,       null: false

      t.timestamps null: false
    end
    add_index :customers, :email, unique: true
  end
end
```

- 参考
	- [create_table - Railsドキュメント](http://railsdoc.com/migration#%E3%83%86%E3%83%BC%E3%83%96%E3%83%AB%E3%81%AE%E4%BD%9C%E6%88%90(create_table))
	- [add_index - Railsドキュメント](http://railsdoc.com/references/add_index)

```bash
$ rake db:migrate
```

### `customers`コントローラー作成
- コントローラー名の後にメソッドをつけると、コントローラーで空のメソッド定義、ビューのテンプレまで作ってくれるのでつける

```bash
$ rails g controller customers index new create edit update show destroy
```

### ルーティング修正

- 'rails g controller'でメソッドまで指定すると、ルーティングがgetのみになるので、修正する。

```ruby:config/routes.rb
Rails.application.routes.draw do
  resources :customers
end
```

### viewファイル削除

- create/update/edit/destroy/new.html.erbは削除

## 初期設定
### 基本設定ファイルをいじる
- `config/application.rb`はすべての環境で共通の設定ファイル
- `config.パラメータ名 = 値`の形式
- 設定系ファイルはすべて、サーバの再起動後に反映される

```ruby:config/application.rb
module SpartCrm
  class Application < Rails::Application
    config.time_zone = 'Tokyo'
    config.i18n.default_locale = :ja
  end
end
```

- 参考
	- [設定ファイル(config) - Railsドキュメント](http://railsdoc.com/config)

### 日本語化ファイルをいれる

- [gitHubからja.ymlをダウンロードする](https://github.com/svenfuchs/rails-i18n/blob/master/rails/locale/ja.yml)
- ダウンロードした`ja.yml`を`config/locale/`の以下に入れる。

```yaml
---
ja:
  date:
    abbr_day_names:
    - 日
    - 月
```

- 参考
	- [設定ファイル(config) - Railsドキュメント](http://railsdoc.com/config)

## 共通レイアウトを開発
### headerをパーシャル化

```app/views/layouts/_header.html.erb
<nav class="navbar navbar-default">
  <div class="container">
    <div class="navbar-header">
      <button type="button" class="navbar-toggle collapsed" data-toggle="collapse" data-target="#navbar-collapse-menu">
        <span class="sr-only">Toggle navigation</span>
        <span class="icon-bar"></span>
        <span class="icon-bar"></span>
        <span class="icon-bar"></span>
      </button>
      <a class="navbar-brand" href="/customers">顧客管理システム</a>
    </div>

    <!-- Collect the nav links, forms, and other content for toggling -->
    <div class="collapse navbar-collapse" id="navbar-collapse-menu">
      <ul class="nav navbar-nav navbar-right">
        <li><a href="#">ダミー</a></li>
      </ul>
    </div><!-- /.navbar-collapse -->
  </div><!-- /.container-fluid -->
</nav>

```

```html:app/views/layouts/index.html.erb
<%= render partial: 'layouts/header' %>

<div class="container">
  <%= yield %>
</div>
```

## `customer` のCRUD周りを開発
### `new`周りを作る
- まずは`new`メソッドから

```ruby:app/controllers/customers_controller.rb
class CustomersController < ApplicationController
  def new
    @customer = Customer.new
  end
end
```

- パーシャルのformを作る

```html:app/views/customers/_form.html.erb
<%= form_for(@customer) do |f| %>
  <div class="form-group">
    <%= f.label :family_name %><br>
    <%= f.text_field :family_name, class: "form-control" %>
  </div>
  <div class="form-group">
    <%= f.label :given_name %><br>
    <%= f.text_field :given_name, class: "form-control" %>
  </div>
  <div class="form-group">
    <%= f.label :email %><br>
    <%= f.text_field :email, class: "form-control" %>
  </div>
  <%= f.submit %>
<% end %>
```

- viewからは`render`で`_form.html.erb`を呼び出す

```html:app/views/customers/new.html.erb
<div class="row">
  <div class="col-lg-12">
    <h1>顧客新規登録</h1>
    <%= render 'form' %>
  </div>
</div>
```

- これでフォームからコントローラーにデータを投げられるが、それを受けて保存してくれるメソッドがない

### `create`周りを作る
- これで新規登録はOK

```ruby:app/controllers/customers_controller.rb
class CustomersController < ApplicationController
  def create
    @customer = Customer.new(customer_params)
    @customer.save
    redirect_to @customer
  end
  
  private

  def customer_params
    params.require(:customer).permit(
      :family_name,
      :given_name,
      :email
      )
  end  
end
```

- `redirect_to @customer`で`show`メソッドのviewにリダイレクトされる
	- 参考：[redirect_to の引数とかのメモ](http://qiita.com/kanpe777/items/c5154b58c852855deefc)
- でもまだ飛び先を作ってないから、このままだとエラー

### `show`周りを作る

```ruby:app/controllers/customers_controller.rb
class CustomersController < ApplicationController
  def show
    @customer = Customer.find(params[:id])
  end
end
```

```html:app/views/customers/show.html.erb
<div class="row">
  <div class="col-lg-12">
    <h1>顧客詳細</h1>
    <dl>
      <dt>お名前</dt>
      <dd><%= @customer.family_name %> <%= @customer.given_name %></dd>
      <dt>電子メール</dt>
      <dd><%= @customer.email %></dd>
    </dl>
  </div>
</div>
```

#### モデルのインスタンスメソッド
- 名字と名前を2つ載せるのではなく、`Customer`モデルに名前と名字を連結させるインスタンスメソッドを作ってそれを呼ぶ

```ruby:app/models/customer.rb
class Customer < ActiveRecord::Base
  def full_name
    full_name = family_name + given_name
    full_name
  end
end
```

- これによって、`Customerモデルのインスタンス.full_name`で、名字と名前が連結された文字列を呼び出せる
- `@customer`は`show`メソッドで作成されたCustomerモデルのインスタンスなので、`full_name`が利用できる

```html:app/views/customers/show.html.erb
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
```

### バリデーションを実装する
- migrationスクリプトで、null禁止指定をした
- バリデーション無しで投げたらどうなるかチェックしてみる
	- 内容確認したら、`rails c`で削除する
	- `irb> Customer.last.destroy`
- フォーム登録時に、チェックする
	- データが入力されているかどうか
	- データが規定の長さ以下か
	- データが規定の書式になっているか
- 参考：[検証(validation) - Railsドキュメント](http://railsdoc.com/validation)

```ruby:app/models/customer.rb
class Customer < ActiveRecord::Base
  validates :family_name,
    presence: true, length: { maximum: 20 }
  validates :given_name,
    presence: true, length: { maximum: 20 }
  VALID_EMAIL_REGEX = /\A[\w+\-.]+@[a-z\d\-.]+\.[a-z]+\z/i
  validates :email,
    presence: true, format: { with: VALID_EMAIL_REGEX }
end
```

- saveメソッドが呼ばれたタイミングで、モデルのバリデーションが実行される
	- バリデーションが通って保存されたら`true`
	- 通らなかったら保存しないで`false`
- 保存できなかったらnewページを再度表示させる

```ruby:app/controllers/customers_controller.rb
class CustomersController < ApplicationController
  def create
    @customer = Customer.new(customer_params)
    if  @customer.save
      redirect_to @customer
    else
      render :new
    end
  end
end
```

- バリデーションが通らず`new`に`redirec_to`された際、エラー内容が表示されるようにする
- `@customer.errors`でエラー情報にアクセスできる
- この辺はコピペで使い回すとこなので、「型」で覚えておきます

```html:app/views/customers/_form.html.erb
<%= form_for(@customer) do |f| %>
  <% if @customer.errors.any? %>
    <div class="alert alert-danger">
      <p><b><%= @customer.errors.count %>件のエラーがあります</b></p>

      <ul>
      <% @customer.errors.full_messages.each do |message| %>
        <li><%= message %></li>
      <% end %>
      </ul>
    </div>
  <% end %>

（省略）
```

### `edit`周りを作る

```ruby:app/controllers/customers_controller.rb
class CustomersController < ApplicationController
  def edit
    @customer = Customer.find(params[:id])
  end
end
```

- viewは`render`で`_form.html.erb`を使い回す

```html:app/views/customers/edit.html.erb
<div class="row">
  <div class="col-lg-12">
    <h1>顧客情報の編集</h1>
    <%= render 'form' %>
  </div>
</div>
```

### `update`周りを作る

```ruby:app/controllers/customers_controller.rb
class CustomersController < ApplicationController
  def update
    @customer = Customer.find(params[:id])
    if  @customer.update(customer_params)
      redirect_to @customer
    else
      render :edit
    end
  end
end
```

- `update`メソッドが呼ばれたタイミングで、モデルのバリデーションが実行される
	- バリデーションが通って保存されたら`true`
	- 通らなかったら保存しないで`false`
- 保存できなかったら`edit`ページを再度表示させる

### `destroy`周りを作る

```ruby:app/controllers/customers_controller.rb
class CustomersController < ApplicationController
  def destroy
    @customer = Customer.find(params[:id])
    @customer.destroy
    redirect_to customers_url
  end
end
```

- `edit`ページのみで削除できるようにする
- Bootstrapで確認ページ（モーダル）を出して、削除可否をユーザーが選べるようにする
	- 参考：[JavaScript · Bootstrap](http://getbootstrap.com/javascript/#modals-examples)

```html:app/views/customers/edit.html.erb
<div class="row">
  <div class="col-lg-12">
    <h1>顧客情報の編集</h1>
    <%= render 'form' %>
    <button type="button" class="btn btn-default btn-sm" data-toggle="modal" data-target="#delete-customer">顧客情報を削除する</button>
  </div>
</div>

<!-- 下記のモーダルは通常は隠れているので、どこに書いてもＯＫ -->
<div class="modal fade" id="delete-customer">
  <div class="modal-dialog">
    <div class="modal-content">
      <div class="modal-header">
        <button type="button" class="close" data-dismiss="modal" aria-label="Close"><span aria-hidden="true">&times;</span></button>
        <h4 class="modal-title">顧客情報の削除</h4>
      </div>
      <div class="modal-body">
        <p><%= @customer.family_name %>さんを削除しますか？</p>
      </div>
      <div class="modal-footer">
        <button type="button" class="btn btn-default" data-dismiss="modal">キャンセル</button>
        <%= link_to '削除する', @customer, method: :delete, class: "btn btn-danger" %>
      </div>
    </div>
  </div>
</div>
```

### ダミーの`customer`を流し込む
- seedファイルはrubyスクリプトなので、rubyのメソッドが使える

```ruby:db/seed.rb
# Staff.create(family_name: "鈴木", given_name: "テスト", email: "test@sparta.com", birth_day: "1980/1/1")

100.times do |index|
  Customer.create(family_name: "鈴木", given_name: "太郎", email: "customer_#{index}@sparta.com")
end
```

### `index`周りを作る

```ruby:app/controllers/customers_controller.rb
class CustomersController < ApplicationController
  def index
    @customers = Customer.all
  end
end
```

```html:app/views/customers/index.html.erb
<h1>顧客情報一覧</h1>
<p>お客様情報が表示されます。</p>
<table class="table table-striped">
  <thead>
    <th>姓</th>
    <th>名</th>
    <th>電子メール</th>
    <th>リンク</th>
  </thead>
  <tbody>
    <% @customers.each do |customer| %>
      <tr>
        <td><%= customer.family_name %></td>
        <td><%= customer.given_name %></td>
        <td><%= customer.email %></td>
        <td><%= link_to '詳細', customer, class: "btn btn-default btn-xs" %></td>
      </tr>
    <% end %>
  </tbody>
</table>
```

- 100件分のデータが一覧表示されて見にくい


以上
