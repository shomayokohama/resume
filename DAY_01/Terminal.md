# ターミナルの基礎
## ターミナルとは

- コマンドと呼ばれる命令文を用いて、Macの操作や設定をおこなうためのアプリ
- ちょっと難しいことをいうと、キーボードでMacと対話するソフト
	- 皆さんは今、タッチパッドでMacと対話しています
- Railsに対して、ターミナルで命令を出して開発を進める

## ターミナルを起動する

[アプリケーション] → [ユーティリティ] → [ターミナル]


## ls コマンド
現在のディレクトリ下にあるファイルやフォルダの情報を表示

**l**ist **s**egmentsの略

```bash
$ ls
Applications			Music
Desktop					Pictures
Documents				Public
Downloads				Library
```

## pwd コマンド
現在のディレクトリを確認

**p**resent **w**orking **d**irectoryの略

```bash
$ pwd
/Users/nashirox
```

## cd コマンド
ディレクトリを移動

**c**hange **d**irectoryの略

```bash
$ cd Desktop # デスクトップへ移動
$ pwd
/Users/nashirox/Desktop
$ cd .. # 一つ上のディレクトリへ
$ pwd
/Users/nashirox
```

## mkdir コマンド
ディレクトリを作成

**m**a**k**e **dir**ectoryの略

$ mkdir + [新フォルダ名]

```bash
$ pwd
/Users/nashirox/Desktop
$ mkdir sample_folder
$ ls
sample_folder
```

## cp コマンド
ファイルやディレクトリをコピー

**c**o**p**yの略

```bash
$ cp sample.txt copy.txt # コピー元 コピー先の順番
$ ls
sample.txt	copy.txt
```

### ディレクトリをコピーするには「オプション」が必要

cpコマンドの後に -a オプションをつける。

```bash
$ cp -a sample_folder copy_folder
$ ls
sample_folder	copy_folder
```

## vi コマンド
ターミナル内で使えるエディタを起動

**vi**sual display editorの略

$ vi [ファイル名 ...］

- 起動後はコマンドモードだが、`i`を押すと編集モードにはいる。
- 編集モードで`esc`を押すと、コマンドモードに戻る
- コマンドモードで`：wq`を入力すると保存して終了、`:q`だと保存しないで終了
- 存在しないファイル名を指定すると、新しいファイルを作成して編集に入る

多くのコマンドがあるので、より詳しく知りたい方は[このサイト](http://net-newbie.com/linux/commands/vi.html)などを参照してください。


## その他便利な使い方
- パスを入力中、`tab`を押すと予測してくれる
- `control + a`で最初にカーソルが戻る
- `cd ~`でホームディレクトリに戻れる

