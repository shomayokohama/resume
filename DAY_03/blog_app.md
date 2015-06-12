# 【実習】ブログアプリ`my_blog`
## モデル
### Post
- title - string
- body - text
- category - string

### Author
- name - string
- description - string
- image_url - string

## コントローラー
### Post
- index
	- ブログのタイトル一覧を表示
- show
	- ブログ記事を表示
- new
	- ブログ記事を書く
- create
	- ブログ記事を保存
- edit
	- ブログ記事の編集
- update
	- 編集したブログ記事の保存
- delete
	- ブログ記事の削除

### Author
1人しかいない設定なので、データベースに直接書き込む。コントローラ不要。

```
$ rails c
irb(main):001:0> Author.create(name: 'sparta', description: 'hogehoge', image_url: 'http://www.nobodyeverdrownedinsweat.com/wp-content/uploads/2012/02/spartan-cam-logo1.png')
```

## ビュー
Bootstrapを利用すること。

- index
	- 左サイドの8カラム
		- 記事を若い順で表示
		- 記事は`truncate`メソッドを使って、50文字で切り捨て
			- メソッドは[Railsドキュメント](http://railsdoc.com/references/truncate)で詳細確認
		- 「記事の続きはこちら」でshowへのリンクを貼る
	- 右サイドの4カラム
		- 著者情報の表示
		- 最新記事を５つ表示
- show
	- 全12カラムで、記事タイトル、カテゴリ、記事本文の順番で表示する
	- 編集ボタンと削除ボタンを最後につける
- edit
	- `show`と同じ順番のフォームで編集する

以上