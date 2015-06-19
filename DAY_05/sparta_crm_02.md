# スパルタ顧客管理システム（２）
## gem を使ってみる
### gemとは
- Rubyの便利ツール
- 無料で公開されていて、簡単にRailsに取り込んで便利に使うことが出来る
- `gem やりたいこと`でググると、だいたい先人が作ってくれている
- 参考：[公式ページ](https://docs.rubygems.org/)

### ページネーションの`kaminari`を使ってみる
- 長いテーブルにページ送り機能をつけてくれる
- 唯一の日本人Railsコミッター、松田明さん作
- [Github amatsuda/kaminari](https://github.com/amatsuda/kaminari)

```Gemfile
gem 'kaminari'
```

- `Gemfile`に使いたいgemを記述したら、`bundle install`

```bash
$ bundle install --without production
```

- `$ gem list`で入ったか確認してみる
- 入っていたら、一回サーバを落として再起動する
- その後、下記を書き換えていく

```ruby:app/controllers/customers_controller.rb
class CustomersController < ApplicationController
  def index
    @customers = Customer.page(params[:page])
  end
end
```

- indexビューの末尾に一行追加

```html:app/views/customers/index.html.erb
<%= paginate @customers %>
```

- これでページネーションが実装された。
- デザイン機能もあって、Bootstrap v3のテンプレートを適用させる

```bash
$ rails g kaminari:views bootstrap3
```

### ダミーデータを作成するFakerを使ってみる
- ダミーの顧客名が鈴木太郎しかない
- それっぽい名前を生成してくれる
- 参考
	- [Faker Github](https://github.com/stympy/faker)
	- [Faker-japanese Github](https://github.com/tily/ruby-faker-japanese)

```Gemfile
group :development, :test do
  gem 'faker'              # 本物っぽいテストデータの作成
  gem 'faker-japanese'     # 本物っぽいテストデータの作成（日本語対応）
end
```

- groupで、そのgemが開発／テスト／本番のどれで使うものか指定できる
- 今回は開発／テスト用gem
- `--without`オプションで指定しないとダメ
	- 一回指定したら、`.bundle/config`ファイルに指定したオプションが記録される
	- それ以降は`--without`オプションいらない

```bash
$ bundle install
```

- seedファイルを書き換える

```ruby:db/seed.rb
# 前回のスクリプトはコメントアウト
# 100.times do |index|
#  Customer.create(family_name: "鈴木", given_name: "太郎", email: "customer_#{index}@sparta.com")
# end

100.times do |index|
  Customer.create(
    family_name: Faker::Japanese::Name.last_name,
    given_name: Faker::Japanese::Name.first_name,
    email: "customer_#{index}@sparta.com"
  )
end
```

- 元のダミーデータを消してseed.rbを流し込む

```
$ rails c
irb> Customer.destroy_all
irb> quit
$ rails db:seed
```

- 流し込まれているかDBビューワーで確認

## `company`モデルの追加と`customer`モデルとのアソシエーション
### `company`モデルの実装
```bash
$ rails g model company name:string url:string address:string
```

- 会社名だけはnull禁止

```ruby:db/migrate/__create_companies.rb
class CreateCompanies < ActiveRecord::Migration
  def change
    create_table :companies do |t|
      t.string :name, null: false
      t.string :url
      t.string :address

      t.timestamps null: false
    end
  end
end
```

```bash
$ rake db:migrate
```

### `customer`モデルに外部キー追加
- migrationで、customerがcompanyを参照していることを記述する
 
```bash
$ rails g migration AddCompanyIdToCustomers company:references
```


```ruby:db/migrate/_add_company_id_to_customers.rb
class AddCompanyIdToCustomers < ActiveRecord::Migration
  def change
    add_reference :customers, :company, index: true, foreign_key: true
  end
end
```

- 参考：[add_reference - Railsドキュメント](http://railsdoc.com/references/add_reference)

```bash
$ rake db:migrate
```

- DBをビューワーで確認すると、`customers`テーブルに、`company_id`が出来ているのがわかる
- `参照先モデル名_id`が外部キーの名前ルール

### ダミーデータの入れ直し
- 新設した`company_id`がnullなので、もう一度ダミーデータを作り直す
- seedファイルを書き換える

```ruby:db/seed.rb
Company.create(name: "スパルタ商事", url: "http://spartacamp.jp/", address: "東京都千代田区１−１−１")
Company.create(name: "株式会社ノーウォール", url: "http://nowall.co.jp/", address: "東京都新宿区西新宿6-15-1")
Company.create(name: "ヤフー株式会社", url: "http://www.yahoo.co.jp/", address: "東京都港区赤坂9-7-1")

100.times do |index|
  Customer.create(
    family_name: Faker::Japanese::Name.last_name,
    given_name: Faker::Japanese::Name.first_name,
    email: "customer_#{index}@sparta.com",
    company_id: rand(3) + 1
  )
end
```

- 元のダミーデータを消してseed.rbを流し込む

```
$ rails c
irb> Customer.destroy_all
irb> quit
$ rails db:seed
```

- 流し込まれているかDBビューワーで確認

### 各モデルでアソシエーション設定

```ruby:app/models/company.rb
class Company < ActiveRecord::Base
  has_many :customers
end
```

```ruby:app/models/customer.rb
class Customer < ActiveRecord::Base
  belongs_to :company
end
```

### indexで所属会社名を表示する
- `customer.company.name`で、`customer`が持っている`company_id`の情報を参照し、その`name`を引っ張ってこれる！

```app/views/layouts/index.html.erb
<div class="row">
  <div class="col-lg-12">
    <h1>顧客情報一覧</h1>
    <p>お客様情報が表示されます。</p>
    <table class="table table-striped">
      <thead>
        <th>姓</th>
        <th>名</th>
        <th>電子メール</th>
        <th>所属会社</th>
        <th>リンク</th>
      </thead>
      <tbody>
        <% @customers.each do |customer| %>
          <tr>
            <td><%= customer.family_name %></td>
            <td><%= customer.given_name %></td>
            <td><%= customer.email %></td>
            <td><%= customer.company.name %></td>
            <td><%= link_to '詳細', customer, class: "btn btn-default btn-xs" %></td>
          </tr>
        <% end %>
      </tbody>
    </table>

    <%= paginate @customers %>
  </div>
</div>
```

### 現在のアソシエーションを図でみてみる
- rails-erdというgemを使う

```bash
$ brew install graphviz
```

- Gemfileにrails-erdを追加

```Gemfile
group :development do
  gem 'rails-erd'
end
```
```bash
$ bundle install
$ rake erd
```

- ルートフォルダに`erd.pdf`というファイルができるので確認

## 課題
- `Customer`コントローラーに`before_action`を定義し、show/edit/destroy/updateの共通処理をまとめてください
- `Customer`モデルに`company_id`のバリデーションを追加してください（presence）
- `customer`が所属する会社を登録／編集できるようにしてください
	- `f.collection_select`を使う
- 役職を表す`Post`モデルを追加して`customer`に紐付け、CRUD操作できるようにしてください
	- 役職例：社長、部長、課長、平社員
	- こちらも登録/編集は`f.collection_select`を使う

## 検索機能を実装する
- `index`の一覧表示で、顧客を検索できるようにする
- ransackというgemで、簡単に検索機能を実装できる
- 参考
	- [公式ページ@Github](https://github.com/activerecord-hackery/ransack)
	- (Railsでransackを使って検索機能を作成する)[http://ruby-rails.hatenadiary.com/entry/20141008/1412774436]

### ransackの実装
```Gemfile
gem 'ransack'
```
```bash
$ bundle install
```

- controllerを書き換える

```ruby:app/controllers/customers_controller.rb
class CustomersController < ApplicationController
  def index
    # @customers = Customer.page(params[:page])
    @q = Customer.search(params[:q])
    @customers = @q.result.page(params[:page])
  end
end
```

- viewにransack用formを追加
ｰ 使い方
	- `search_form_for`を使う
	- `カラム名_start`、`カラム名_cont`などで検索条件を指定する
	- 名字と名前のOR検索なら`:family_name_or_given_name_cont`

```app/views/layouts/index.html.erb
<div class="row">
  <div class="col-lg-12">
    <h1>顧客情報一覧</h1>
    <p>お客様情報が表示されます。</p>

    <%= search_form_for @q do |f| %>
      <div class="row">
        <div class="col-sm-3 col-xs-12">
          <div class="form-group">
            <%= f.label :family_name_cont, '姓' %>
            <%= f.search_field :family_name_cont, class: "form-control input-sm" %>
          </div>
        </div>
        <div class="col-sm-3 col-xs-12">
          <div class="form-group">
            <%= f.label :given_name_cont, '名' %>
            <%= f.search_field :given_name_cont, class: "form-control input-sm" %>
          </div>
        </div>
        <div class="col-sm-3 col-xs-12">
          <div class="form-group">
            <%= f.label :email_cont, '電子メール' %>
            <%= f.search_field :email_cont, class: "form-control input-sm" %>
          </div>
        </div>
        <div class="col-sm-3 col-xs-12">
          <div class="form-group">
            <%= f.label :company_name_cont, '会社名' %>
            <%= f.search_field :company_name_cont, class: "form-control input-sm" %>
          </div>
        </div>
      </div>
      <div class="row">
        <div class="col-xs-12 text-center">
          <%= f.submit "検索", class: "btn btn-default" %>
        </div>
      </div>
    <% end -%>
    
    <hr />
（省略）
```

- これで検索ができるようになった。

### テーブルのソート機能
- `sort_link`ヘルパーをテーブルのヘッダーに設定することで、テーブルの列をソートすることができる

```app/views/layouts/index.html.erb
（省略）
    <table class="table table-striped">
      <thead>
        <th><%= sort_link(@q, :family_name, "姓") %></th>
        <th><%= sort_link(@q, :given_name, "名") %></th>
        <th><%= sort_link(@q, :email, "電子メール") %></th>
        <th><%= sort_link(@q, :company_name, "所属会社") %></th>
        <th>リンク</th>
      </thead>
（省略）
```

## ログイン機能の実装
- 今は誰でも顧客情報の操作ができるので、スタッフにしかできなようにする
- ログイン管理はdeviseというgemを使う
- 参考
	- [公式ページ Github](https://github.com/plataformatec/devise)
	- [Railsのログイン認証gemのDeviseのインストール方法 - Rails Webook](http://ruby-rails.hatenadiary.com/entry/20140801/1406907000)

### deviseの実装
```Gemfile
gem 'devise'
```

- gemのインストール後、devise関連ファイルをプロジェクトに追加

```bash
$ bundle install
$ rails g devise:install
```

- ログアウト時のリダイレクト先として、root_urlの設定が必要

```ruby:config/routes/rb
root 'customers#index'
```

- deviseで管理されるuserモデルの追加

```bash
$ rails g devise User
$ rake db:migrate
```

- 下準備はこれで完了。サーバーは再起動してください。
- 次はヘッダーにログインしていなければログインへのリンク、ログインしていたらログアウトへのリンクを表示する
- 使用されているビューヘルパー
	- `モデル名_signed_in?`で、ログインしているかをどうか返してくれる
	- `xxxx_xxx_xx_path`は、`rake routes`で確認できます

```app/views/layout/_header.html.erb
（省略）
    <div class="collapse navbar-collapse" id="navbar-collapse-menu">
      <ul class="nav navbar-nav navbar-right">
        <% if user_signed_in? %>
          <li><%= link_to "設定変更", edit_user_registration_path %></li>
          <li><%= link_to "ログアウト", destroy_user_session_path, method: :delete %></li>
        <% else %>
          <li><%= link_to "新規登録", new_user_registration_path %></li>
          <li><%= link_to "ログイン", new_user_session_path %></li>
        <% end %>
      </ul>
    </div><!-- /.navbar-collapse -->
（省略）
```

- これでビューを確認してみると、新規登録やログインができるようになっている

### アクセス制限をかける
- `customers`コントローラーの全アクションは、ログインしていないとできないようにする
- `:authenticate_モデル名!`というdeviseのメソッドを、コントローラー内の`before_action`で呼び出し、フィルタリングをする

```ruby:app/controllers/customers_controller.rb
class CustomersController < ApplicationController
  before_action :authenticate_user!

   （省略）
end
```

- `only`オプションを指定することで、ログインが必要なメソッドを絞ることもできる
	- ex. `before_action :authenticate_user!, only: [:new, :create, :edit, :update, :destroy]`
	- これで`index`と`show`は誰でも閲覧できるので、試してみる

### 注意書きを表示させる

```app/views/layout/application.html.erb
（省略）
<div class="container">
  <% if notice.present? %>
    <div class="alert alert-dismissable alert-success">
      <button type="button" class="close" data-dismiss="alert">&times;</button>
      <p><%= notice %></p>
    </div>
  <% end %>

  <% if alert.present? %>
    <div class="alert alert-dismissable alert-danger">
      <button type="button" class="close" data-dismiss="alert">&times;</button>
      <p><%= alert %></p>
    </div>
  <% end %>
  
  <%= yield %>
（省略）
```

### 日本語化する
- 公式ページの[ここ](https://github.com/plataformatec/devise/wiki/I18n#japanese-devisejayml)にバージョン毎の日本語化ファイルがある
- 3.4.0以上のリンクから、`devise.ja.yml`をダウンロード
- `config/locales`以下に設置する
- サーバーを再起動すると、日本語化している

### ログイン画面のカスタマイズ
- デフォルトの画面をカスタマイズしたい場合は、コマンド入力でテンプレートがプロジェクトに入る
- 以降はこっちを使ってくれる

```bash
$ rails g devise:views
```

- 生成されるファイルと使われる画面
	- ログイン画面
		- `app/views/devise/sessions/new.html.erb`
	- 新規登録画面
		- `app/views/devise/registrations/new.html.erb`
	- 登録情報変更画面
		- `app/views/devise/registrations/edit.html.erb`
	- パスワードを変更するためのメールを送信する画面
		- `app/views/devise/passwords/new.html.erb`
	- パスワード変更画面
		- `app/views/devise/passwords/edit.html.erb`
	- メール認証画面
		- `app/views/devise/confirmations/new.html.erb`
	- アカウントのアンロック画面
		- `app/views/devise/unlocks/new.html.erb`
- ログイン画面を修正してみる

```app/views/devise/sessions/new.html.erb
<div class="row">
  <div class="col-sm-8 col-sm-offset-2">
    <h2>ログイン</h2>

    <%= form_for(resource, as: resource_name, url: session_path(resource_name)) do |f| %>
      <div class="form-group">
        <%= f.label :email, "電子メール" %><br />
        <%= f.email_field :email, autofocus: true, class: "form-control" %>
      </div>

      <div class="form-group">
        <%= f.label :password, "パスワード" %><br />
        <%= f.password_field :password, autocomplete: "off", class: "form-control" %>
      </div>

      <% if devise_mapping.rememberable? -%>
        <div class="checkbox">
          <label>
            <%= f.check_box :remember_me %> ログイン情報を記録する
          </label>
        </div>
      <% end -%>

      <div class="form-group">
        <%= f.submit "ログイン", class: 'btn btn-default' %>
      </div>
    <% end %>

    <%= render "devise/shared/links" %>
  </div>
</div>
```

## 課題
- ログイン画面のように新規登録画面も同じようにカスタマイズしてください
	- `app/views/devise/registrations/new.html.erb`

## 答え
### 新規登録画面をカスタマイズ

```app/views/devise/registrations/new.html.erb
<div class="row">
  <div class="col-sm-8 col-sm-offset-2">
    <h2>ユーザー新規登録</h2>

    <%= form_for(resource, as: resource_name, url: registration_path(resource_name)) do |f| %>
      <%= devise_error_messages! %>

      <div class="form-group">
        <%= f.label :email, "電子メール" %><br />
        <%= f.email_field :email, autofocus: true, class: "form-control" %>
      </div>

      <div class="form-group">
        <%= f.label :password, "パスワード" %>
        <% if @minimum_password_length %>
        <em>(<%= @minimum_password_length %> 文字以上が必要です)</em>
        <% end %><br />
        <%= f.password_field :password, autocomplete: "off", class: "form-control" %>
      </div>

      <div class="form-group">
        <%= f.label :password_confirmation, "パスワード再入力" %><br />
        <%= f.password_field :password_confirmation, autocomplete: "off", class: "form-control" %>
      </div>

      <div class="form-group">
        <%= f.submit "新規登録" %>
      </div>
    <% end %>

    <%= render "devise/shared/links" %>
  </div>
</div>
```

## `user`にカラム追加する
### 新規登録情報の追加
- 現在、`user`は電子メールのみしか列がないので、`family_name`と`given_name`を追加

```bash 
$ rails g migration AddFamilyNameAndGivenNameToUsers family_name:string given_name:string
```

```ruby:db/migrate/20150619032957_add_family_name_and_given_name_to_users.rb
class AddFamilyNameAndGivenNameToUsers < ActiveRecord::Migration
  def change
    add_column :users, :family_name, :string
    add_column :users, :given_name, :string
  end
end
```

- 追加した列を、`application_controller.rb`に独自のメソッドで定義しないといけない
	- 参考: [Strong Parameters - 公式ページ](https://github.com/plataformatec/devise#strong-parameters)

```ruby:app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
  # Prevent CSRF attacks by raising an exception.
  # For APIs, you may want to use :null_session instead.
  protect_from_forgery with: :exception
  before_action :configure_permitted_parameters, if: :devise_controller?

  protected

  def configure_permitted_parameters
    devise_parameter_sanitizer.for(:sign_up) << :family_name
    devise_parameter_sanitizer.for(:sign_up) << :given_name
  end
end
```

- あとはviewに追加していくらか修正するのみ

```app/views/devise/registrations/new.html.erb
<div class="row">
  <div class="col-sm-8 col-sm-offset-2">
    <h2>ユーザー新規登録</h2>

    <%= form_for(resource, as: resource_name, url: registration_path(resource_name)) do |f| %>
      <%= devise_error_messages! %>

      <div class="form-group">
        <%= f.label :family_name, "姓" %><br />
        <%= f.text_field :family_name, autofocus: true, class: "form-control" %>
      </div>

      <div class="form-group">
        <%= f.label :given_name, "名" %><br />
        <%= f.text_field :given_name, class: "form-control" %>
      </div>

      <div class="form-group">
        <%= f.label :email, "電子メール" %><br />
        <%= f.email_field :email, class: "form-control" %>
      </div>

      <div class="form-group">
        <%= f.label :password, "パスワード" %>
        <% if @minimum_password_length %>
        <em>(<%= @minimum_password_length %> 文字以上が必要です)</em>
        <% end %><br />
        <%= f.password_field :password, autocomplete: "off", class: "form-control" %>
      </div>

      <div class="form-group">
        <%= f.label :password_confirmation, "パスワード再入力" %><br />
        <%= f.password_field :password_confirmation, autocomplete: "off", class: "form-control" %>
      </div>

      <div class="form-group">
        <%= f.submit "新規登録" %>
      </div>
    <% end %>

    <%= render "devise/shared/links" %>
  </div>
</div>
```

### アカウント情報変更を追加

```ruby:app/controllers/application_controller.rb
  def configure_permitted_parameters
    devise_parameter_sanitizer.for(:sign_up) << :family_name
    devise_parameter_sanitizer.for(:sign_up) << :given_name
    devise_parameter_sanitizer.for(:account_update) << :family_name
    devise_parameter_sanitizer.for(:account_update) << :given_name
  end
```

```app/views/devise/registrations/edit.html.erb
<div class="row">
  <div class="col-sm-8 col-sm-offset-2">
    <h2><%= resource_name.to_s.humanize %> 編集</h2>

    <%= form_for(resource, as: resource_name, url: registration_path(resource_name), html: { method: :put }) do |f| %>
      <%= devise_error_messages! %>
      <div class="form-group">
        <%= f.label :family_name, "姓" %><br />
        <%= f.text_field :family_name, autofocus: true, class: "form-control" %>
      </div>

      <div class="form-group">
        <%= f.label :given_name, "名" %><br />
        <%= f.text_field :given_name, class: "form-control" %>
      </div>

      <div class="form-group">
        <%= f.label :email, "電子メール" %><br />
        <%= f.email_field :email, class: "form-control" %>
      </div>

      <% if devise_mapping.confirmable? && resource.pending_reconfirmation? %>
        <div>Currently waiting confirmation for: <%= resource.unconfirmed_email %></div>
      <% end %>

      <div class="form-group">
        <%= f.label :password, "新しいパスワード" %> <i>（入力しない限り変更しません）</i><br />
        <%= f.password_field :password, autocomplete: "off", class: "form-control" %>
      </div>

      <div class="form-group">
        <%= f.label :password_confirmation, "新しいパスワード（再入力）" %><br />
        <%= f.password_field :password_confirmation, autocomplete: "off", class: "form-control" %>
      </div>

      <div class="form-group">
        <%= f.label :current_password, "現在のパスワード" %> <i>（設定を変更する場合は、現在のパスワード入力が必要です）</i><br />
        <%= f.password_field :current_password, autocomplete: "off", class: "form-control" %>
      </div>

      <div class="actions">
        <%= f.submit "更新する" %>
      </div>
    <% end %>

    <h3>アカウント削除</h3>

    <p>イケてない？ <%= button_to "アカウントを削除する", registration_path(resource_name), data: { confirm: "本当に退会しますか？" }, method: :delete %></p>

    <%= link_to "戻る", :back %>
  </div>
</div>
```

以上
