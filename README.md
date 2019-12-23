# 入門Unixシェルプログラミング

# 一章
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

### for文

```bash
for variable in word-list
do
  command
  ・・・
done
```

```bash
for i in a b c d
do
  echo $i
done
```

### while

```bash
while command-list
do
  command
  ・・・・
done
```

```bash
a=1
while test $a -lt 3
do
  echo $a
  a=`expr $a + 1`
done
```

### case文

```bash
case string in
  pattern1) command-list ;;
  pattern2) command-list ;;
  pattern3) command-list ;;
  ・・・・
esac
```

```bash
STRING=abc
case "$STRING" in
  ABC) echo "STRING is ABC" ;;
  abc) echo "STRING is abc" ;;
  xyz) echo "STRING is xyz" ;;
esac
```

### testコマンド

testコマンドは、ある条件を判定し、その条件が正しい場合には真(0)を返し、誤っている場合には偽(0以外)を返す。ifやwhileのような条件判定文で使いやすコマンド

```bash
if test -r file
then
  echo "the file exists and i can read it"
fi

# もっと読みやすいように鉤括弧で置き換えることができる
if [ -r file ]
then
  echo "the file exists and i can read it"
fi
```

# ２章

## シェル変数

- アルファベット、数字、アンダースコアを使うことができる
※1文字目は数字はNG
- シェルの変数名は大文字を使うことが慣例になっている

```bash
# 正しい変数のセット
variable1=value

# =の両脇にスペースを入れるとえらーになる
variable1 = value

# 複数の変数を1行にセットする場合はスペースで区切る
variable1=value1 variable2=value2
```

## シェル変数の中身の確認

`$variable` もしくは `${variable}`のようにする。$や{}はこれが変数であることを示すためのもの。{}で括ったほうが分かりやすい

```bash
$ FOO=abc
$ echo $FOOBAR
$
$ echo ${FOO}BAR
abcBAR
```

## シェル変数の初期設定

### =によるシェル変数の設定

```bash
# 変数variableを展開するとき、未使用がヌル値であればvalueを使用する
${variable:=value}
# ヌル値が入っている場合は既存のものを使用する
${variable=value}
```

```bash
$ echo ${ABC:=xyz}
xyz
$ echo $ABC
xyz
$ echo ${ABC:=abc}
xyz
$ ABC=""
$ echo ${ABC=123}
$
$ echo ${ABC:=123}
$ 123
```

### -によるシェル変数の設定
= と違うのは変数が未使用、未設定の状態の時値を代入しないまま、指定した値を返す

```bash
# 変数variableを展開するとき、未使用がヌル値であればvalueを使用する
${variable:-value}
# ヌル値が入っている場合は既存のものを使用する
${variable-value}
```

```bash
$ echo ${ABC:-xyz}
xyz
$ echo $ABC
$
$ echo ${ABC:=xyz}
xyz
$ echo $ABC
xyz
```

もう一つ=とことなるのは、-だと位置パラメタが使えること

`echo The variable ${1:-abc} will be used`

$1に何かしらの値がセットされていればそのまま使う。$1が未定義、未使用ならばabcという結果を返す。つまり、変数が書き込み禁止であっても置き換えて使用しているように見える。

### ?によるシェル変数の設定

?は、変数がこれまで未使用、未設定であるかどうかを確認するときに使う。変数がこおれまで未使用、未設定のときにmessageの部分が標示される。さらに、シェルスクリプトの場合は、そこで処理が終了する。

```bash
# 変数variableを展開するとき、未使用がヌル値であればmessageを使用する
${variable:?message}
# ヌル値が入っている場合は既存のものを使用する
${variable?message}
```

```bash
$ echo ${ABC:?"ABC is no set"}
-bash: ABC: ABC is no set
```

### +によるシェル変数の設定

-を使用したときによく似ている。変数に何らかの値が設定されている時、値を取り替えて標示する。もとの変数の値は変更しない。つまり、変数の値を変更せずにそのときだけ結果を変えたい場合に利用する

```bash
${variable:+value}
${variable+value}
```

```bash
$ echo ${ABC:+zzz}
$
$ ABC=www
$ echo ${ABC:+zzz}
zzz
$ echo $ABC
www
```

## 位置パラメタ

`./nnn.sh a b c d e f g h i` を実行すると、順に$0 〜 $9に値が入る($0は./nnnが入る。コマンド自身)

`$#`はコマンドに渡される引数の数が入る。この値は実際にそのコマンドに与えられて引数の数であり、$0に該当するコマンド自身は数に入らない。

引数全体を表現するには、`$*`と`$@`という2種類ある。2つの違いはダブルクォートで囲んで使用した場合の展開の方法にある。

- `$*`をダブルクォートで括ると、引数全体を1個のダブルクォートで囲んだ状態で展開する
- `$@`をダブルクォートで括ると、引数をそれぞれ1個ずつダブルクォートで囲んだ状態で添加する

## 特殊な変数

### $? 変数

コマンド実行時の終了ステータスを表す変数。&を使いバックグラウンドで実行させたコマンドに対して無効です。

コマンドは、通常実行終了時に、正常終了の場合は真(0)、異常時終了(0以外)の値がセットされる。

```bash
command
if [ $? -eq 0 ]
then
  ・・・
fi

# 上記は次と同じ
if command
then
  ・・・
fi
```

### $$変数

変数$$には、現在動作しているコマンドの「プロセスID」がセットされる。

プロセスIDというのは、UNIX上で管理されるものであり、何かコマンド実行されたときに、必ずそのUNIX上で一意に決定されて割り当てられる。

```bash
$ echo $$
43722
```

### $!変数

&を使ってコマンドをバックグランドで走らせたとすると、そのコマンドのプロセスIDが$!にセットされる。

```bash
command &
・・・
wait $!
```

waitコマンドは、引数のPIDを持つバックグラウンドジョブが終了するのを待つということ。上の例は、commandというコマンドをバックグランドで実行させ、・・・部分で何か別の処理をする。そして先程のバックグラウンドの処理が確実に終わるのを待ってから、この先に進めるということ。

### $- 変数

$-という変数には、そのシェルの起動時フラグや、setコマンドを使って設定したフラグの一覧がセットされている。

```bash
$ echo $-
himBH
```

この場合は、今動作しているシェルはh,i,m,B,Hという3つのフラグが指定されている。`/bin/bash -himBH`という形で実行されている。

## コマンド行上での変数の設定

コマンドを実行するとき、そのコマンドだけに有効になるよう変数を設定することが可能

`$ CFLAGS=-g make`

このようにmakeコマンドを実行すると、CFLAGSという変数に値がセットされた状態で実行される。