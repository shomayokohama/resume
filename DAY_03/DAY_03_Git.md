# Git入門
## 準備
### スライド
- [デザイナのためのGit入門](http://www.slideshare.net/dsuket/git-16343460)

### SourceTreeのインストール
- 本来はターミナル上でgit操作するけど、GUIアプリもあるのでそっちを使います
- [こちら](https://www.atlassian.com/ja/software/sourcetree/overview)からダウンロード
	- app storeにもあるけど、バージョンが古すぎるので直接ダウンロード
	- bitbucketのアカウントもついでに登録

## 実習
### 基本操作

- [サルでもわかるGit入門 - 入門編](http://www.backlog.jp/git-guide/intro/intro1_1.html)

### リモートリポジトリの設定
リモートリポジトリをGithub上に作る。

- [SSHの公開鍵を作成しGithubに登録する手順 - mon_sat at Co-Edo（半年前の自分への教科書 / 別院）](http://monsat.hatenablog.com/entry/generating-ssh-keys-for-github)

### 応用操作
ブランチやマージなど。

- [サルでもわかるGit入門 - 発展編](http://www.backlog.jp/git-guide/intro/intro1_1.html)

## その他
- 日常業務のファイルもGitで管理すると便利
	- ただし、WordやExcelはバイナリファイルなので、差分は確認できない
	- コミットメッセージでわかるようにしておくと良い。
- リモートリポジトリを運用できるサービス（ホスティングサービス）はいくつかある。
	- [Bitbucket](https://bitbucket.org/)
		- 無料アカウント枠でもプライベートリポジトリが無制限に作れる。
		- 商用で少人数開発の場合はよく使われる
	- [Backlog](http://www.backlog.jp/)
		- 国産サービス。

		
	- [GitLab](https://about.gitlab.com/)
		- Gihubのクローン。自分でサーバにインストールして利用できる。
		- Railsで実装されている。
