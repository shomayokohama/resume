# 初めてのRailsアプリ（２）

## チケット情報の項目追加
連絡先として、「携帯電話番号」をデータベースに追加する。

### １．データベースの変更
データベースの変更には`migration`ファイルを作成し、`rake db:migrate`を実行する。

#### 構文

- `$ rails generate migration ファイル名 データ名:データ型 ...`
- ファイル名は、`Add...To...`にすること。

#### 実行

```bash
$ rails generate migration AddMobilePhoneToTickets mobile_phone:string
      invoke  active_record
      create    db/migrate/20150605064627_add_mobile_phone_to_tickets.rb
```

できあがった`migration`ファイルを確認。

```ruby:db/migrate
class AddMobilePhoneToTickets < ActiveRecord::Migration
  def change
    add_column :tickets, :mobile_phone, :string
  end
end
```

- `migration`のファイル名を規則に従わずに入力すると、Railsが開発者の意図を判断できず、`change`メソッドの中身が空っぽになる。
- 自分で入力すればいいけど、めんどくさいので規約には従った方がいい。

```bash
$ rake db:migrate
== 20150605064627 AddMobilePhoneToTickets: migrating ==========================
-- add_column(:tickets, :mobile_phone, :string)
   -> 0.0007s
== 20150605064627 AddMobilePhoneToTickets: migrated (0.0008s) =================
```

### ２．ビューの修正
- index.html.erb
- show.html.erb
- _form.html.erb

### ３．コントローラーの修正
Strong parameterに:mobile_phoneを追加。

```ruby
def ticket_params
  params.require(:ticket).permit(:name, :price, :email, :tent, :note, :mobile_phone)
end

```

## ステージ情報の追加
チケット管理だけではなく、ステージ情報を管理したい（バンドの出演情報）

### 仕様
- ステージ情報の一覧表示と、各情報の登録／編集／削除

### データ型
1. アーティスト名
2. 日付
3. 開始時間
4. 終了時間
5. ギャラ

### 日本語化
完成後、日本語化をしていく。

以上


