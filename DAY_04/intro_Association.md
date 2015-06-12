# ガイドツアー管理システム

- ２つのモデル間の連携を学ぶ
- 部分テンプレートの利用法を学ぶ

## プロジェクト作成

```bash
$ rails new sparta_tour
$ cd sparta_tour/
```

## モデル設計
### Course
- id:integer
- tour_date:date
- guide_name:string
- title:string
- description:text
- lower_limit:integer
- upper_limit:integer

### Participant
- id
- course_id
- name


```bash
$ rails g scaffold course tour_date:date guide_name:string title:string description:text lower_limit:integer upper_limit:integer
$ rails g scaffold participant name:string course_id:integer
$ rake db:migrate
```

## ビュー開発
- course/showでコースの内容確認と予約を一度にできるようにする
- ページ内に別ページを埋め込むためには、**部分テンプレート（パーシャル）**を利用する
- パーシャルはファイル名先頭に`_`がついているとRailsが自動認識する
- course/_new_participant.html.erbを作成し、partcipants/new.html.erbの内容をコピペする

```ruby:_new_participant.html.erb
<%= form_for(participant) do |f| %>
  <% if participant.errors.any? %>
    <div id="error_explanation">
      <h2><%= pluralize(participant.errors.count, "error") %> prohibited this participant from being saved:</h2>

      <ul>
      <% participant.errors.full_messages.each do |message| %>
        <li><%= message %></li>
      <% end %>
      </ul>
    </div>
  <% end %>

  <div class="field">
    <%= f.label :name %><br>
    <%= f.text_field :name %>
  </div>
  <div class="field">
    <%= f.label :course_id %><br>
    <%= f.number_field :course_id %>
  </div>
  <div class="actions">
    <%= f.submit %>
  </div>
<% end %>
```

- `course/show.html.erb`に`render`メソッドを追加する
- メソッドの詳細は[ドキュメント](http://railsdoc.com/references/render)で確認してください

```ruby
<p>
  <strong>Upper limit:</strong>
  <%= @course.upper_limit %>
</p>

<%= render partial: 'new_participant', locals: { participant: Participant.new() } %>

<%= link_to 'Edit', edit_course_path(@course) %> |
<%= link_to 'Back', courses_path %>
```

### 応用知識（わからなくてもOK）
- なお、`courses`コントローラー内の`show`アクションに`@participant`を記述しても動く。

```
def show
  @participant = Participant.new
end
```

- それをやらずに` locals: { participant: Participant.new }`とローカル変数を渡している理由は、コントローラーの数が増えてくると同じ記述をしなければいけないから
- パーシャルは複数のコントローラから呼ばれることが多い
- パーシャルとコントローラーを切り離しておくと、後々管理が楽になる

## 予約者一覧をコースページ内に表示する
- `courses/_participants_list.html.erb`を作り、`participants/index.html.erb`の中身を改変して挿入する

```
<table>
  <thead>
    <tr>
      <th>Name</th>
      <%# course_idは不要 %>
      <th colspan="3"></th>
    </tr>
  </thead>

  <tbody>
    <% participants.each do |participant| %>
      <tr>
        <td><%= participant.name %></td>
        <%# course_idは不要 %>
        <td><%= link_to 'Show', participant %></td>
        <td><%= link_to 'Edit', edit_participant_path(participant) %></td>
        <td><%= link_to 'Destroy', participant, method: :delete, data: { confirm: 'Are you sure?' } %></td>
      </tr>
    <% end %>
  </tbody>
</table>
```

- 上でつくったパーシャルを、`course/show.html.erb`で呼ぶ

```
<p>
  <strong>Upper limit:</strong>
  <%= @course.upper_limit %>
</p>

<%= render partial: 'participants_list', locals: { participants: Participant.where(course_id: @course.id) } %>

<%= render partial: 'new_participant', locals: { participant: Participant.new(course_id: @course.id) } %>

<%= link_to 'Edit', edit_course_path(@course) %> |
<%= link_to 'Back', courses_path %>
```

## アソシエーション
![](http://wiki.usagee.co.jp/index.php?plugin=ref&page=ruby%2Frails%2FRailsGuides%E3%82%92%E3%82%86%E3%81%A3%E3%81%8F%E3%82%8A%E5%92%8C%E8%A8%B3%E3%81%97%E3%81%A6%E3%81%BF%E3%81%9F%E3%82%88%2FActive%20Record%20Associations&src=has_many.png)
出典：[ruby/rails/RailsGuidesをゆっくり和訳してみたよ](http://wiki.usagee.co.jp/ruby/rails/RailsGuides%E3%82%92%E3%82%86%E3%81%A3%E3%81%8F%E3%82%8A%E5%92%8C%E8%A8%B3%E3%81%97%E3%81%A6%E3%81%BF%E3%81%9F%E3%82%88/Active%20Record%20Associations)

- テーブル間をidで紐つける仕組み
- 関連するモデルで宣言することで、アソシエーションが設定され、便利なメソッドが利用可能となる
	- `course.rb`は`hay_many :partiants`
	- `participant.rb`は`belongs_to :course`

```ruby
class Course < ActiveRecord::Base
  has_many :participants
end
```

- アソシエーションを指定すると、`@course.participants`で関連づけられたデータを引っ張ってこれる

```ruby
<%= render partial: 'participants_list', locals: { participants: @course.participants } %>
```

- `@course.participants.name`のようにして値にもアクセスできる

### 応用知識（わからなくてもOK）
- データの塊をＲＤＢＭＳで扱えるようテーブルに細かく切り出すことを、「正規化」と言います。
- 厳密さに応じて第５正規化までありますが、通常は第３正規化まで。
- 自分が作りたいシステムのデータを洗い出し、正規化することがモデル設計の第一歩。

![データベース正規化](http://image.itmedia.co.jp/ait/articles/0605/11/l_r3table01.gif)
出典：[データベースエンジニアへの道（3）：素早く正規形を見抜く実践テクニック ](http://www.atmarkit.co.jp/ait/articles/0605/11/news124.html)

## validateの実装
- 正しいデータの形式かどうか、デーベース登録前に判断する門番のような機能
- `save`メソッドを呼び出すと、`save`メソッドがバリデーションを実行する
	- バリデーションに引っかかると`false`が返る。
- 詳細は[Railsドキュメント](http://railsdoc.com/validation)
