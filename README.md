# 入門Unixシェルプログラミング

## コメント
`#`から行末までがコメントになる

```shell
#!/bin/sh
# この行はコメント1
# この行もコメント2
echo "Hello World!" # ここもコメント
```

## 1行の扱い
`\`の直後の改行は無視される

```shell
$ echo hello\
>world
helloworld

$ echo hello \
>world
hello word

# 複数のスペースは一つに置き換えられる
$ echo hello     \
>world
hello world

# 複数のスペースを入れたい場合はダブルクォーテーションでくくる
$ echo "hello   \
>world"
hello    world
```

## バックスラッシュ
直後の1文字をエスケープする

```bash
$ echo abc\def
abcdef

$ echo abc\\def
abc\def
```

## シングルクォーテーション
シェルのクォーテーションで一番強力なもの。すべて普通文字の扱いとなる。

シングルクォートでエスケープできないのはシングルクォートだけ

```bash
$ echo 'abc\def'
abc\def
```

## ダブルクォーテーション
ほとんどの特殊文字はエスケープできる。

しかし、$、`、\の3つの特殊文字はエスケープできない(シングルクォーテーションを使う)

注意点としては、`\` の直後にある$、`、\、"はエスケープされる。それ以外の場合は\が表示される

```bash
$ FOO=foo
$ echo $FOO
foo

$ echo "$FOO"
foo

$ echo "\$FOO"
$FOO

$ echo "F\OO"
F\OO

$ echo '$FOO'
$FOO

echo '\$FOO'
\$FOO
```

## クォーテーションの使い分け

ダブルクォートとシングルクォートの使い分けとして、「囲んだ文字列の中で、変数の置き換えやコマンド置換した結果を使いたいときはダブルクォートを使用する

```bash
$ FILE=foobar
$ echo "Cannot remove $FILE"
Cannot remove foobar

$ echo 'Cannot remove $FILE'
Cannot remove $FILE

$ echo "Tody is `date`"
Tody is Thu Dec 19 22:32:16 JST 2019

$ echo 'Today is `date`'
Today is `date`
```

## バッククォートによるコマンド置換

バッククォートは、その名kに書かれたコマンドを実行し、その結果をその１に書き込む

```bash
$ echo "Tody is `date`"
Tody is Thu Dec 19 22:32:16 JST 2019

# 最初にecho defの部分が添加され、defに置き換わります。次にecho abc def ghiが処理され、その値がSTRINGに入る
$ STRING=`echo "abc \`echo def\` ghi"`
$ echo $STRING
abc def ghi

# 上のやつをくどく書くと
$ TMPSTR=`echo def`
$ STRING=`echo "abc $TMPSTR ghi"`
$ echo $STRING
abc def ghi
```

## コマンドの終了ステータス

コマンドは普通、慣例として、正しく終了したときは0、そうでないときは1を終了時にセットする。

- 0 : 成功 : true
- 0以外 : 失敗 : false

## コマンドセパレート

### セミコロン

左から順に実行される。改行の箇所にセミコロンを置いたもの。

利点としては複数行になるコマンドを1行にまとめることができる

```bash
$ cd /tmp; ls

# ↑と全く同じ動作
$ cd /tmp
$ ls
```

### パイプ(|)

処理を左から右へ流していく意味でパイプライン処理と言う

| の左側で実行したコマンドの結果を、| の右側のコマンドの入力として処理する

```bash
$ echo abc | wc
1 1 4
```

### バックグラウンドでの実行

& をコマンドの後ろにつけると、そのコマンドはバックグラウンドで実行される

```bash
# 5秒間プロンプトが表示されない
$ sleep 5

# すぐにプロンプトが表示される
$ sleep 5 &
```

### OR演算子

||をOR制御演算子と呼ぶ。

コマンドの間に入れて実行すると、左側のコマンドが失敗したとき初めて右側のコマンドを実行する。

ORだからまたはと解釈すると混乱するので注意

```bash
# ファイルの削除に失敗したら、メッセージが表示される
$  rm file || echo "ERROR - Cannot remove file"
```

### AND演算子

&& をコマンドに間に入れて実行すると、左側のコマンド成功した(0を返した)ときだけ、右側のコマンドを実行する。

```bash
# ディレクトリを作成できた場合、ファイルをコピーする
$ mkdir dir && cp file dir
```

## コマンドのグルーピング

### 丸括弧()のグルーピング

括弧で囲まれたコマンドは現在の動作しているシェルとは別に、サブシェルのもとで動作する。サブシェルがあ終了するまで次の処理には移行しない。


親の環境は子の環境(サブシェル)に引き継がれるが、このシェルで変更した環境は親には影響しない

```bash
# 現在どこのディレクトリにいようが、makeが実行されるのはmakedir
# そして、makeが終了すると、makedirディレクトリではなく、もとのディレクトリにいる
$ (cd $HOME/makedir; make)
```

### 中括弧のグルーピング

現行のシェルで実行される。中括弧を使うのは普通、それぞれのコマンド結果をひとまとめにしたいようなとき。

```bash
# makeコマンドの出力結果の前に、その日の日付を挿入することができる
$ { date; make; } > make.list
```

## 制御文

### if文

```bash
if command-list
then
  command
  ・・・・
elif command-list2
then
  command
  ・・・・
else
  command
  ・・・・
fi
```

```bash
# fileがあるかないかで、メッセージを分ける
if test -f file
then
  echo "The file exist"
else
  echo "the file does not.exsit"

# ifと同じ行にthen書くものをよくやること。注意するのは、そのときはthenの前にセミコロンを付ける必要がある
if test -f file; then
  echo "The file exist"
else
  echo "the file does not.exsit"
```
