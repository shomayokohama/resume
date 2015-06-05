# Rubyの基礎（２）
## 制御構造
### １．条件判断系
#### if
- 条件には、値がtrueかfalseとなる式を書く。
- Railsで一番使う。

```ruby
if 条件 then
  条件がtrueの時に実行したい処理
end
```

```ruby
chikoku = 1
if chikoku < 3 then
  puts "セーフ"
end
=> セーフ
```

数値の比較

```ruby
p 5 == 5 # => true 
p 4 > 6  # => false
p 2 <= 3 # => true 
p 4 != 5 # => true
```

文字列の比較

```ruby
p "スパルタ" == "Sparta" # false
p "スパルタ" != "Sparta" # true
```

#### if + else

```ruby
if 条件１ then
  条件１がtrueの時に実行したい処理
elsif 条件２ then
  条件２がtrueの時に実行したい処理
else
  全条件がfalseの時に実行したい処理
end
```

```ruby
chikoku = 3
if chikoku < 3 then
  puts "セーフ"
else
  puts "アウト"
end
=> アウト
```

- 条件の合体も可能
	- `条件１ && 条件２`は、どちらも`true`の場合。
	- `条件１ || 条件２`は、どちらかが`true`の場合

```ruby
chikoku = 1
kesseki = 0

if chikoku >=3 || kesseki >= 1 then
  puts "アウト"
else
  puts "セーフ"
end

=> セーフ
```

#### unless
`if`とは逆の条件判断。あまり使わない。

```ruby
unless 条件 then
  条件がfalseの時に実行したい処理
end
```

### ２．繰り返し系
#### timesメソッド

```ruby
10.times do
  puts "10回この内容が出力されます。"
end

10.times { puts "10回この内容が出力されます。" }

```

#### eachメソッド
Railsで非常に多用するメソッド。配列やハッシュの要素に順番にアクセスできる。

#### 配列.each
```ruby
配列.each do |変数|
  繰り返し処理
end
```

- `do〜end`を**ブロック**と呼ぶ
- 配列が先頭から一つずつ取り出されて`|変数|`に格納され、それをブロック内で用いて処理をする

```ruby
yamanote = ["新宿", "池袋", "東京", "品川", "大崎"]

yamanote.each do |station|
  puts "次は#{station}駅です。"
end
=> 次は新宿駅です。
=> 次は池袋駅です。
=> 次は東京駅です。
=> 次は品川駅です。
=> 次は大崎駅です
```

次のように、`do〜end`を`{〜}`に置き換えて書くこともできる

```ruby
yamanote.each { |station| puts "次は#{station}駅です。" }
```

- Ruby界の慣習として、複数行に渡る処理の場合は`do〜end`、一行ですむ場合は`{〜}`で記述する。
- Railsはほぼ`do〜end`だけなので、これだけ覚えればとりあえずOK。

#### ハッシュ
```ruby
ハッシュ.each do |キーの変数, 値の変数|
  繰り返し処理
end
```

- 配列のeachとほぼ同じだが、キーと値の２つの変数を設定することができる。

```ruby
fruits = { "apple" => 200, "orange" => 150, "cherry" => 300 }

fruits.each do |fruit, price|
  puts "#{fruit}の値段は#{price}円です"
end
=> appleの値段は200円です
=> orangeの値段は150円です
=> cherryの値段は300円です
```

```ruby
all_user = {
	:nashirox => { :id => 372, :name => "Daisuke", :email => "n@wmegane.com", :DM => true },
	:mohira =>   { :id => 264, :name => "Ohira", :email => "ohira@nowall.com", :DM => false }
}

all_user.each do |name, info|
  puts "#{info[:name]}さんの情報"
  puts "メールアドレス：#{info[:email]}"
  puts "ＤＭ送信の可否：#{info[:DM]}"
  puts "----------------------"
end
=> Daisukeさんの情報
=> メールアドレス：n@wmegane.com
=> ＤＭ送信の可否：true
=> ----------------------
=> Ohiraさんの情報
=> メールアドレス：ohira@nowall.com
=> ＤＭ送信の可否：false
=> ----------------------
```
