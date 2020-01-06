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

# 3章

## シェル関数

下記が基本の形。nameという名前の関数ができあがる。普通のコマンドを実行するようにnameとすれば、中括弧で囲んだ部分が動作する。

関数名のあとの丸括弧は関数の定義として必要なもの。普通は、その中には何も書かない.

```bash
name()
{
  command
}
```

`return [n]`

関数では戻り値を設定できる。[n]と書いている部分に任意の番号を設定すれば、それがこの関数の戻り値となる。ここで返す値は`$?`で参照可能で、ifやwhileの条件判定に使用できる。returnb文を明示的に書いていない場合は、戻り値は「関数内の最後にじっこうさあれたコマンドの実行終了ステータス」となる。

### 関数をカレントシェルで利用すること

```bash
#!/bin/bash
pse()
{
  ps- ac |  sort -bn
}
```

```bash
$ ./aaa
$ pse
pse: not found

$ . ./aaa
$ pse
実行結果が標示される
```

ドットコマンドを使うことで、シェルスクリプト内で定義された関数を現在のシェルから利用できるようになる。

## 組み込みコマンド

### ヌルコマンド(:)

何もしないがいつも成功する(真の結果を返す)コマンド。実行終了ステータスで0を返す。`:`と書く。

条件判定式で必ず真にしたい場合などに使用することが多い

```bash
while :
do
  if ・・・
  then
    break
  fi
done
```

また、何かコマンドを書かなくてはならないような状態で、実際は何もしたくないときにも、使える。

```bash
# thenのあとにコマンドが必要なため、ヌルコマンド使用している
if command-listt
then
  :
else
  command
fi
```

### ドットコマンド(.)

`. file`

という形で使用するもの。ピリオドがそのコマンド。これは普通のコマンドとは違い、新しくプロセスを作らずに現行のシェルのプロセスを使って指定されたファイルを読み込み実行する。指定されたファイルで記述されている変数や関数が、現行のシェルで有効に使えるようになる。

### breakコマンド

`break`

forやwhileのループから抜け出るときに使用するコマンド。

デフォルトでは1個のループを抜ける。ループがネストしているうときに、複数を脱したときには引数を指定する。


`break number`

### cdコマンド

`cd [directory]`

引数で指定したディレクトリに移動する。


### continueコマンド

`continue`

forやwhileのループ内で、いったんループ内の処理を終えて次の繰り返しに意向させたい場合に使用する。

breakと同じようにループがネストしている場合は引数を設定できる

`continue number`

### echoコマンド

`echo parameter`

引数の部分を標準出力(画面)に表示するコマンド

### evalコマンド

```bash
$ VAR1=value
$ VAR2=VAR1

# このとき、VAR2という変数を使ってVAR1に代入したvalueを標示させたい

# 一回目しか展開されない
$ echo $"$VAR2"
VAR1

# $$というプロセス番号への変換処理が先に処理される
$ echo $$VAR2
43722VAR2

# 複数の変換処理を一度にする場合は、evalを使用する
$ eval echo \$$VAR2
value
```

evalコマンドは、変数の置換やコマンドの置換、ワイルドカードの展開などが複雑に絡んでいるときに、1行のコマンドでいっぺんに展開させてしまうのもの。

記述法は、`eval string`

### execコマンド

`exec command`

execコマンドは、新しくプロセスを作らず現行のカレンドシェルのプロセスと置き換えて、引数のコマンドを実行させる。このコマンドを実行したら、もうもとのシェルに戻ることはない。

```bash
#!/bin/sh
exec ls -l
echo abc
```

execコマンドでlsが起動された時点でシェルスクリプトのプロセス自体がlsのプロセスに置き換わるので、その後のechoコマンドは実行されない。

### exitコマンド

exitコマンドはシェルを終了するときに使用する。

シェルスクリプト実行中に何かしらのエラーが発生した時、その時点で実行をやめたい場合にこのコマンドを使う。

`exit number`

番号を引数にすることでシェルスクリプトの終了コード指定することができる。この終了コードは`$?`変数に渡される。

シェルスクリプトの終了で、exitコマンドで明示的に番号を指定することが望ましいた。

### exportコマンド

`export name`

このように書くことで、nameで指定した変数を、他のコマンドやシェルから利用することができる。

### pwdコマンド

カレントディレクトリの完全パスを標示する。

### readコマンド

`read variable`

このコマンドは、キーボードからの入力をvariable変数にセットする。

```bash
#!/bin/sh
echo -n "enter yes or no -->"
read ANSWER
```

### readonlyコマンド

`readonly name...`

このコマンドはnameで指定された変数をリードオンリーにする。

### returnコマンド

`return exitstatus`

シェルの関数から抜けるコマンド。exitstatusに指定した番号がその関数の終了コードとなる。番号が指定されなかった場合には、関数が終了する直前のコマンドの実行終了コードが返る。

### setコマンド

setコマンドは、シェルのオプションをオンにしたりオフにしたりする。

```bash
$ sh #シェルを起動
$ echo abc
abc
$ set -v #冗長モード
$ echo abc #コマンドを打つと、入力コマンドを表示後、結果がでる
echo abc
abc
$ set +v #オプションをもとに戻す
$ echo abc #入力コマンドが標示されない
abc
```

代表的なオプション

* -i 対話的に動作するようにする
* -n コマンドを読み取るが、実行はしない
* -s コマンドを標準入力から読む
* -v 入力されたコマンドを標準エラーに出力する
* -x コマンドの実行前に、実行するコマンドを標準エラーに出力する

注意点は-をともうなうとそのフラグがオンになり、+が伴うとオフになるということ

### shiftコマンド

このコマンドは、位置パラメタの値を飛騨アリにずらす。

`shift number`

引数としてnumberの所に数値(9まで)を書けば、その数値の文だけずらす。

```bash
#!/bin/sh
echo $0 $1 $2 $3 $4 $5 $6 $7 $8 $9
shift
echo $0 $1 $2 $3 $4 $5 $6 $7 $8 $9
shift 3
echo $0 $1 $2 $3 $4 $5 $6 $7 $8 $9
```

というスクリプトあった場合、下記のような挙動となる

```bash
$ ooo aa bb cc dd ee ff gg hh ii
./ooo aa bb cc dd ee ff gg hh ii
./ooo bb cc dd ee ff gg hh ii
./ooo ee ff gg hh ii
```

### testコマンド

testコマンドは、ある条件を判定し、正しいか正しくないかによって真(成功、0の値)か(失敗、0以外)を返す。ifやwhileの条件判定に使うのがもっとも多い利用。

書き方は次の二通りある

* test expression
* [ expression ]

下記、判定オプションの一礼

ファイルに関するテスト
```
-r file #fileが「読み取り可」なら真
-w file #fileが「書き込み可」なら真
-x file #fileが「実行可」なら真
-f file #fileが「普通のファイル」なら真
-d file #fileが「ディレクトリ」なら真
-s file #fileが「0より大きいサイズ」なら真
```

文字列に関するテスト
```
-z string #stringの「長さが0」なら真
-n string #stringの「長さが0より大」なら真
string    #stringが「ヌルでなければ」真
str1 = str2 #str1とstr２が「同じ」なら真
str1 != str2 #str1とstr2が「同じでない」なら真
```

数値に関するテスト
```
int1 -eq int2 #int1とint2が「等しい」なら真
int1 -ne int2 #int1とint2が「等しくない」なら真
int1 -lt int2 #int1がint2「より小さい(less than)」なら真
int1 -le int2 #int1がint2「以下(less than or equal)」なら真
int1 -gt int2 #int1がint2「より大きい(greather than)」なら真
int1 -ge int2 #int1がint2「以上(greather than or equaql)」なら真
```

その他
```
! #NOTの意味
-a # AND
-o # OR
( expr ) #グルーピング
```

### trapコマンド

`trap action signal...`

このシェルスクリプトが、signalに指定したシグナルを受け取ったときにどういう処理をするかをactionの所に指定する。

### typeコマンド

`type name`

nameには普通コマンドの名前を指定する。typeコマンドは、nameで指定したコマンドが、このシェルでどういう取り扱いなのかを出力する。

```bash
$ type echo date
echo is a shell builtin
date is /bin/date
```

### umaskコマンド

`umask mask`

ファイルを生成するときにどういうモードで作るかを決定する。

```bash
OLDMASK=`umask` #現在の値を保存
umask 002 #変更していろいろ操作
・・・・
・・・・
umask $OLDMASK #もとに戻す
```

### unsetコマンドは

unsetこまんどは、以下のようにして指定した変数や関数を消去する

`unset name`

### waitコマンド

`wait jobnumber`

引数に指定するのは動作しているプログラムのプロセスIDです。このコマンドは、そのプログラムが終了するまでずっとまちます。

# 第4章 リダイレクションによるファイル操作

## ファイルディスクリプタ

プロセスとそれが使用するファイルとの結びつけをする。何かプログラムがファイルを作成して書き込みを行う場合など、プロセスはファイルディスクリプタを使ってっ対象となるファイルへアクセスをする。

ファイルディスクリプタは、ファイルディスクリプタ0番、ファイルディスクリプタ5番というように、数値で表現される。

|ファイルディスクリプタ| 意味 |
|---|---|
| 0 | 標準入力(普通はキーボードからの入力) |
| 1 | 標準出力(普通は端末画面への出力) |
| 2 | 標準エラー(普通は端末画面への出力) |

## リダイレクション

「普通は」キーボードからの入力であり、「普通は」端末画面への出力であると書く。この「普通の入力元や出力先」を変えることをリダイレクションという。この機能をつかうことで、例えばファイルからデータを入力したり結果をファイルに書き出したりすることができる。

リダイレクト起動としては、「>」と「<」を基本的に用いる。

```bash
$ echo abcdefg      # echoは「標準出力」に書く
abcdefg      　　　  # 結果は画面に標示される
$ echo abcdef > abc # 出力をファイルabcに向ける
$                   # 画面に出力されない
$ cat abc           # ファイルabcにechoした内容が書き込まれた
abcdef

$ echo 123456 > abc # 別の出力を書き込んでみる
$ cat abc           # abcの内容を見る
123456
$ 
$ echo xyz >> abc   # >>というリダイレクト記号を使うと、
$ cat abc           # 結果は追加されている
123456
xyz
```

何らかの出力を > でリダイレクトし、ファイルに結果を書き込もうとしたとき、そのファイルが既に存在していたら中身はクリアされる。クリアせずに追加で書きたい場合には、リダイレクト記号として >> を使う。

```bash
$ cat abc xxx # abcとxxxというファイルをcatする
123456        # abcの中身は表示されるが、
xyz           # (↓)xxxというファイルは存在しないのでエラー
cat: xxx: No such file or directory
$ cat abc xxx > nnn # 同じことをリダイレクトすると
cat: xxx: No such file or directory #エラーだけが画面にでる
$
```

nnnというファイルにはabcの中身が書かれた。そしてエラーメッセージは画面にそのまま表示される。エラーメッセージは標準エラーに出力されたもの。

通常は標準出力も標準エラーも画面に出力されるので区別はつかないが、理大レクトを用いると別のものと分かる。

標準エラーが画面に標示される理由は、>というリダイレクト記号は標準出力の向きだけを変えたからである。

正確に書くと、`echo abcdef 1>abc` となる。これは、1番のファイルディスクリプタを経由するものを、abcというファイルに向けなさい、という意味。1は省略可能なので、通常は>と書き表す。

エラーメッセージだけをファイルに書くには、以下のようにする

```bash
$ cat abc xxx 2> nnn # 標準エラーの2番ファイルnnnに向ける
abcdef               # abcの内容は標準出力に書かれるから
xyz                  # このように画面に標示される
```

標準出力と標準エラーをひとまとめにリダイレクトする書き方。

```bash
# 標準出力をnnnというファイルに書くように指示。そして、標準エラーは標準出力に書くようにリダイレクトをしている。2>&1はファイルディスクリプト2番を1番と一緒にまとめるということ。
$ cat abc xxx > n 2>&1
```

リダイレクト記述法

```bash
>file   標準出力をfileというファイルに書く
>> file 標準出力をfileというファイルに追加で書き込む
>&m     標準出力をm番のファイルディスクリプタに書く
>&-     標準出力をクローズする

<file   標準入力をfileというファイルから読み込む
<&m     標準入力をmというファイルディスクリプタから読み込む
<&-     標準入力をクローズする
<<word  標準入力をヒアドキュメントから読み込む
```

上記の書き方は、デフォルトのファイルディスクリプタ(1と0)を省略できる。

リダイレクト記号はコマンド行のどこに置いてもよいが、下記のように最後に書くのが一般的

`$ command param1 param2 .... >/dev/null`

## リダイレクトを使った書き込み(>、>>)

```bash
# fileというファイルが既に存在していれば上書きする。
# ファイルが存在しなければ新規作成する
$ command > file

# ファイルを上書きするのではなく、追記をする
# ファイルが存在しなければ新規作成をする
$ command >> file

# 標準エラーをリダイレクトする
$ command 2>  file
$ command 2>> file
```

ファイルディスクリプタの番号を明示的に指定すれば、そのファイルディスクリプタが向いている出力先を別のファイルに変更することが可能

`command n>&m`

この場合、ファイルディスクリプタn番をm番に向ける。nが省略されたときは、標準出力の1がデフォルトを使う。

echoコマンドは標準出力にメッセージを標示する。よってシェルスクリプトの中で、エラーが発生したときにechoコマンドを使ってメッセージを出力させてもそれは標準エラーには書き込まれない。この場合、シェルスクリプト中で、「エラーメッセージをエラーとして出したい」ときには以下のように書けばいい。

`echo "Error messages." 1>&2`

## リダイレクトを使った読み込み(<、<<)

標準入力もリダイレクトすることができる。すなわち、キーボードから入力せずファイルから入力させることができるということ。

`command < file`

また、`<<`を使えば、ファイルを使わずに「そこに書いてあるテキスト」をそのまま入力することができる

```bash
command << word
 ・・・
 ・・・
word
```

これはヒアドキュメントという。<<のあとに任意のワードを書くと、次にその同じワードが出てくる行までに書かれたテキストを標準入力からのものだと解釈する。上の例では・・・の部分に書いた文字列が、commandへのキーボード入力として扱われる。

## 制御構文とリダイレクション

- グルーピングしたコマンドに対して：

```bash
{
  command
  ・・・
} > stdout
```

- if文：

```bash
if command-list
then
  command
  ・・・
fi > stdout
```

- for文：

```bash
for variable [ in word-list ]
do
  command
  ・・・
done > stdout
```

- while文：

```bash
while command-list
do
  command
  ・・・・
done > stdout
```

- case文：

```bash
case string in
  pattern) command-list ;;
  ・・・
esac > stdout
```

ifやwhileが完全にクローズしたところでリダレクトさせることで、その中で行われた処理の全部がひとまとめに渡されることになる。stdoutの所に出力ファイル名を指定する。

## execコマンドとリダイレクション

execコマンドは、現在のシェルのプロセスをそのまま置き換えて新しいコマンドを実行させるもの。この機能を逆手に取り、execコマンドに何も引数を与えず、リダイレクト記号だけを書くことで現行のシェルに対してリダイレクト処理を行わせることができる。

```bash
$ echo abc
abc
$ exec >/dev/null #シェルの標準出力を/dev/nullにリダイレクトする
$ echo abc #出力がリダイレクト先にいった
$ cat aaa # 存在しないファイルをオープンする
cat: aaa: No such file directory # 標準エラーは出力される
```

execを使ったシェルの入出力のリダイレクションには、以下のような用法がある

```bash
exec < file # fileから入力する
exec 1<&2   # 標準出力を標準エラーに向ける
exec 2< /dev/null # 標準エラーを捨てて画面に出さないようにする
exec > /dev/null 2>&1 #標準出力と標準エラーをすべて捨てる
exec >>file # 標準出力をfileに追加する
```

## ファイルからの読み込み

readコマンドは入力を読み込んで変数にセットするもの

```bash
read ANSWER
if [ "$ANSWER" = "yes" ]; then
  ・・・
else
  ・・・
fi
```

このようにすることで、シェルスクリプトの中で入力を待ち、この値がyesであれば・・・する処理をすると書ける。

リダイレクト記号を使用することでファイルから読み込ませることができる。readコマンドでは改行コマンドまでが渡るので、複数行あると2行目以降は取得できない。

しかし、readをwhile文と使用することで操作可能となる。

```bash
# fileから1行読み取りそれをLINEという変数に代入してdoからdoneの間で何かしらの処理をさせる
while read LINE
do
  ・・・
done < file
```

## ファイルのゼロリセット

ファイルの大きさを0にしたいときや、何も書かれていないファイルを作成したいとき、以下の方法で行うことができる

```bash
$ >file
$ :>file

$ cat /dev/null > file
$ cp /dev/null file
```

エディタでファイルを開いて中身を削除してしまっても1バイトは残るため、0にしたい場合は上記のようにする必要がある。

## ヒア・ドキュメント(Here Documents)

`<< word`

wordという部分にはどういう言葉を使ってもよい。この場合、このあとwordという言葉が「行頭に、しかもそのいちごだけの行として」出てくるまでの間に書かれたテキストがすべて標準入力からのものとして処理される。

「ここまでを入力のデータとして扱うよ」の意味をできるだけ明示的にするために、wordの部分にはEOFやENDという言葉をよく使う。

```bash
command << EOF
・・・・
・・・・
END
```

こう書くと、・・・・の部分がcommandの入力として扱われる。

```bash
$ cat
> this is a
> input data.
> ^D
this is a
input data.

# これを、<<を使って書き換えると次のようになる
$ cat << END
> this is a
> here document.
> END
this is a
here document
```

<<のあとに指定した文字列が現れるまでを、そのコマンドの入力として扱える。

```bash
$ DOC=document
# $DOCは展開されるｆｄｓ
$ cat << END
> this is a
> here $DOC
> END
this is a
here document.

# $DOCの文字列をそのまま処理させたい場合は、<<に続く文字列を\でクォートすれば良い
$ cat << \END
> this is a
> here $DOC
> END
this is a
here $DOC

# 個別にクォートすることもできる。
# クォートするもの、しないものが混在する場合に使用する
$ cat << END
> this is a
> here \$DOC
> END
this is a
here $DOC

# <<のあとに、-(ハイフン)を付けることができる。そうすると、行頭のタブは入力されていないものとして処理される
# インデントを使ってスクリプトを読みやすくする際に便利
$ cat <<- END
> <tab>this is a
> <space><space>here document.
> END
this is a
  here document
```

# 第５章: 環境

## 環境変数

環境変数はプロセスが作られたときに引き継がれる「名前とその値」が組になったものをいう。これはプロセスができるときに親の環境をコピーする。

これはあくまでコピーして利用されるものであり、そのプロセスで値を変更しても、元の(親の)値には影響を与えない。

`env` コマンドで今セットされている環境変数の一覧が得られる

## 子の環境を変更する

普通に変数の値を設定したり変更したりしても、その変数が環境変数になるわけではない。環境変数を追加・変更するには、exportコマンドを使う。このコマンドによって、他のプロセスが普通の変数の値を環境変数として認識できるようになる。

```bash
# このようにすることで、makeにこの値を認識できるようになる
$ CFLAGS=-g
$ export CFLAGS
$ make

# こうしたexportした変数は、このmake以外のコマンドやプロセスにも影響を与える。
# それを避けて、ある一部部分にだけに影響するようにしたい場合に下記のように丸括弧で囲みそのサブシェルだけで実行するようにする
(CFLAGS=-g; export CFLAGS; make)

# あるいは以下のように書くことも可能
CFLAGS=-g make
```

## 親の環境を変更する

基本的に、プロセスが異なるため、親の環境を変更することはできない。

2つのプロセスの間で情報を交換したいのなら、何かファイルを仲立ちにさせることが基本的なやり方である。このプロセスの方でセットしたり変更したものをファイルに書き込んで、そのあと親プロセスがそれを読み込み同じように変更させるもの。

## PATH変数

PATH変数にはディレクトリの一覧が入っている。何かコマンドを打った時、このディレクトリを順番にそのコマンドがあるかないかを調べていく。

```bash
$ echo $PATH
/bin:/usr/bin:/usr/local/bin:
```

このようにPATH変数が定義されれいる場合に、/binと/usr/local/binディレクトリの下に、それぞれabcという名称のコマンドがあったなら/bin/abcというコマンドが実行される。

自分の書いたシェルスクリプトや、自分の作ったコマンドを普通のコマンドとして使用したい場合、PATH変数にそれらがおいてあるディレクトリを追加すればよい。

```bash
PATH=$PATH:$HOME/mycmd; export $PATH
```

## ユーザ情報、マシン情報

### ユーザ名の取得

USERあるいはLOGNAMEという環境変数があり、現行ユーザの名前がこの変数にセットされている。

ユーザの名前を知るには、whoami、logname、id、whoといったコマンドから得ることができる

```bash
$ whoami
user
$ logname
user
$ id
uid=1001(user) gid=1001(usergrp)
$ who
user Nov 17 22:49
user ttys000  Dec 29 12:47
```

### マシン名称の取得

hostnameコマンド、そのマシンの名称を取ってくることができる。

```bash
$ hostname
host
```

## シグナルの処理

普通、プロセスはシグナルを受け取ると実行を中断する。シェルではtrapコマンドを使用して、シグナルを受け取ったときにいろいろな処理をさせることが可能になっている。

`trap command-list signal_naumber`

2番目の引数にしてした番号のシグナルを受け取った時、command-list部分のコマンドを実行する。

```bash
trap command-list signal_number # シグナルを受け取って処理する
trap '' signal_number # シグナルを無視をする
trap signal_number # シグナルをリセットする
```

trapコマンドはシェルスクリプトのどこに書いても構わないが、通常はできるだけ初めの方に書く。シェルスクリプトは上から下に順に実行されていくので、trapコマンドを書く以前になにかのシグナルを受け取ってしまうと、そこで中断されてしまうから。

# 第6章: コマンド行の解析、処理

## コマンド行の書き方

`command [options] [parameters]`

オプションは普通1文字で表記し、他の引数と区別するためハイフン(-)を前に置く。ハイフンとオプションの文字との間にはスペースは置かない。

`command -a`

# 第7章: フィルターの使用法

## フィルターとは

フィルタというのはコマンドの一種であり、標準入力からデータを受け取ってそれに変更を加えて標準出力に書き出すという働きをするもの。データの受け渡しには通常パイプライン(|)を利用する

```bash
# このように書けば、catコマンドで標準出力にファイルの内容を書き出してフィルタ役のコマンドに渡すことでfileのデータに手を加えることができる。
cat file |
    filter_1 |
    filter_2 |
    while read LINE
    do
      ・・・
    done
```

## sedコマンド

sedコマンドはフィルタを作るためにはとても有用なコマンド。標準入力からデータを受け取って編集後の結果を標準出力に書き出す。

### sedコマンドの基本的な使い方

`$ sed -e "s/01dText/NewText/" samplefile`

samplefileの中身が下記に左側になっていたとき、結果が右側のようになる。

```
01dTextaaaa01dText → NewTextaaaa01dText
mmmnnnn → mmmnnnn
hasjs01dTexthajsh → hasjhNewTexthajsh
```

samplefileの中から、01dTextという文字列を探してそれをNewTextという文字列に置き換え、標準出力に書き出す。

この結果をファイルに残したければ、リダイレクトさせる。

`$ sed -e "s/01dText/NewText/" samplefile > result`

また、次のように書いても結果は同じようになる

`$ cat samplefile | sed -e "s/01dText/NewText/" > result`

sedにつけた-eというオプションは、その後の文字列が編集用のコマンドだということを表している。

よって、`s/01dText/NewText/`を編集しなさいという意味となる。

sはsubstitute(置換)の意味。`s/01dText/NewText/`という編集コマンドは、/で囲んだ2つの文字列の左側の文字列を右側の文字列で置き換えるという意味。

１行の中で見つけた文字列をすべて置き換える場合には、globalのgを次のように指定する。

`s/01dText/NewText/g`

変換処理をさせるコマンドの中には変数を使うことも可能

```bash
$ OLDTEXT=OldText
$ NEWTEXT=NewText
$ sed -e "s/$OLDTEXT/$NEWTEXT/" samplefile
```

また、オプションとして-nを使う場合もかなりある。sedは処理した行も処理しなかった行もすべて標準出力に書き出すという動作である。

-nをつけると、指示した行だけを標準出力にだすという動作に変わる。このオプションを使用する場合には、「出力しなさい」と意味でprintのpを必ず一緒に使用する必要がある。

```bash
# これは2行目を標準出力に書き出すだけ、内容の変更は何もしません
$ sed -n '2p' < samplefile
mmmnnnn

# 先程の置換処理を組み合わせると置換処理を行った行だけを出力させることもできる
$ sed -n -e "s/OldText/NewText/gp" samplefile
NewTextjahjsaNewTextajs
hasjnNewTexthajsh
```

### sedコマンドをパイプでつなぐこと

次のように繋ぐことで、一度に複数の文字を置換できる

`sed -e "s/Old_1/g" | sed -e "s/Old_2/g" | ・・・`

```bash
# sedで繋いだパターン
cat file |
    sed -e "s/OldText1/NewText1/g" |
    sed -e "s/OldText2/NewText2/g" |
    while read LINE
    do
      ・・・
    done

# sedは同時に複数の変換処理の指定ができるので、次のようにも書ける
cat file |
    sed -e "s/OldText1/NewText1/g" \
        -e "s/OldText2/NewText2/g" |
    while read LINE
    do
      ・・・
    done
```

### sedコマンドのデリミタを変更する

区切り文字(デリミタ)として`/`を利用している。

`s/OldText/NewText/g`

では変換する文字に/という文字列が含まれている場合を考える。abc/efという文字列をABC-EFに変えたいという場合。

```bash
# /をエスケープする
s/abc\/ef/ABC-EF/g

# デリミタとして使用している/という文字は、単に慣習的に使われているだけなので、どんな文字をデリミタとして使用しても構わない
# デリミタとして使用するという宣言も不要で、sの直後の文字がデリミタとして使用される
s%abc/ef%ABC-EF%g
s.abc/ef.ABC-EF.g
sxabc/efxABC-EFxg
```

/の変わりには%や@という文字がよく使われる。

## sedを使っての編集

### 文字列の置換

`sed -e "s/OldText/NewText/g"`

最後のgがなければ、その行に出てくる最初の文字列だけを対象にする。

### 文字列の削除

`sed -e "s/TextToRemove//g"`

これはTextToRemoveという文字列を、何もない文字列に変換させること。

### 行頭の文字列を消す

`sed -e "s/^TextToRemove//"`

^は行の最初を表す。上記の例は行の最初に指定した文字列があった場合、何もないものに置換するため結果的に削除することになる。

### 行末の文字列を消す

`sed -e "s/TestToRemove\$//"`

$は行末を意味する文字。$はシェルの特殊文字のなので、sedより先にシェルに解釈させてはいけないため、エスケープさせる。

### 文字列を追加する

`sed -e "s/abc/abcxxx"`

文字列の置き換えと同じようなこと

### 行の先頭に文字列を追加する

`sed -e "s/^/TextToInsert/"`

このようにすれば対象ファイルの全部の行頭にTextToInsertで指定した文字列を書き込まれる。

### 行末の文字列を追加する

`sed -e "s/\$/TextToAppend/"`

### ドット(.)とアスタリスク(*)

.には「任意の1文字」の意味がある。

`sed -e "s/^...//"`

このように指定すると、行頭から3文字すべてを消去することになる。

*は「直前の文字が任意の個数連続した場合(0個も含む)」を表す。
a*という指定は、a,aa,aaaaなど、aが任意の個数続く文字列を表す。

.*は任意の文字列を表す。

`sed -e "s/.*/abcd"`

これは行の全部をabcdという文字列に置き換える。

### 文字列の切り詰め、切り取り

```bash
# Patternという文字列を削除する
sed -e "s/Pattern//"

# Patternという文字列とそれに続く任意の文字列をいみするため、Patternという文字列以降を削除する
sed -e "s/Pattern.*//"

# Pattern1で始まって、Pattern2で終わる文字列を削除する
sed -e "s/Pattern.*Pattern2//"
```

### 大文字小文字を入れ替える

アルファベットの大文字を小文字に変換したり、小文字を大文字に変換するにはtrコマンドが便利。

```bash
# ファイルの中身を全部小文字にする
$ cat file | tr '[A-Z]' '[a-z]' > lowerfile

# 全部大文字にする場合
$ cat file | tr '[a-z]' '[A-Z]' > upperfile
```

### タブをスペースに変換

`sed -e 's/<tab>/<space>/g'`

### 複数のスペースを1っ子のスペースに変換

複数のスペースは、<space>*で表現できる

`sed -e 's/<space><space>*/<space>/g'`

注意するのは<space><space>*と2個のスペースを並べるところ。<space>* とすると、0個か1個以上のスペースという意味になり、全部の文字の前にスペースが挿入される。

### ホワイトスペースを1個のスペースに変換

ホワイトスペースというのはタブかスペースのこと。タブが入っているのか、スペースが入っているのかわからないときや、タブやスペースが混在している場合には1個のスペースに変えるには以下のようにする。

`sed -e 's/[<space><tab>][<space><tab>]*/<space>/g'`

鉤括弧はシェルの鉤括弧と同じ用法で、どちらかを表現する

`[<space><tab>][<space><tab>]`

は、以下の4パターンになる

```
<space><space>
<space><tab>
<tab><space>
<space><tab>
```

### 行頭のホワイトスペースを削除

`sed -e 's/^[<space><tab>]*//'`

### 行末のホワイトスペースを削除

`sed -e 's/[<space><tab>]*$//'`

### 文字列指定による行の削除

Textとう文字列を含んだ行を削除するには以下のように指定する。

`sed -e "/Text/d"`

変換するわけではないので、substituteのsは不要。最後にdeleteの意味のdを指定する。

### 空白行の削除

何も書かれていない行の削除するには次のようにする。

`sed -e '/^$/d'`

ただし、この場合スペースやタブだけの行は削除できない。スペースやタブだけの行も含めた空白行を削除したい場合は、以下のように書く

`sed -e '/^[<space><tab>]*$/d'`

### sedによる行の指定

sedコマンドでは、そのファイルの中で処理させる行を指定することできる。

```bash
# fileの5行目から20行目を処理する。指定されていない行は何も処理しない
sed -e "5,20s/OldText/NewText/g" file

# 5行目から最終行まで処理をする
sed -e "5,$s/OldText/NewText/g" file
```

### 行の削除

```bash
# ファイルの1行目を削除
sed -e '1d' file

# 最初の4行を削除
sed -e '1,4d' file

# 最終行を削除
sed -e '$d' file
```

### 指定した行だけを表示する

```bash
# 1行目だけを表示する
sed -n '1p' file

# n行だけを標示させる
sed -n 'np' file

# n行目からm行目まで表示する
sed -n 'n,mp' file

# 最終行を表示する
sed -n '$p' file
```

### コメント行の削除

```bash
# コメント行を削除
# コメントが行頭にある場合のみ、削除できる。
sed -e '/^#/d' file

# 行の途中にコメントがある場合に、削除する
# これでは行そのものは削除できない
sed -e 's/#.*// file'
```

```bash
while read LINE
do
  case $LINE in
    \#* ) continue
          ;;
      * ) echo "$LINE"
          ;;
  esac
done
```

### ファイルを後ろから表示する

古く書き込まれたものより先に新しく書き込まれたものから順にファイルを見たい。

`grep -n '.*' | sort -n -r | sed 's/^[0-9]*://'`

grepコマンドに-nオプションを用いて各業に行番号を地ける。sortで逆順に並べて、sedで数字箇所を削除する。

# 第8章:シェルのいろいろな機能

## 数値の計算

シェルは数値であってもそれは文字として扱われている。計算するためには、exprコマンドを経由して行う

### 整数の計算

```bash
expr int1 + int2 # int1+int2を加える
expr int1 - int2 # int1からint2を引く
expr int1 * int2 # int1にint2を掛ける
expr int1 / int2 # int1をint2で割る
expr int1 % int2 # int1をint2で割った余り
```

```bash
$ expr 3 + 5
8
$ expr 3 - 5
-2
$ expr -4  + -2
-6
$ expr 3 '*' 5
15
$ expr 30 \* -2
-60
$ expr 10 / 2
5
$ expr 10 / 3
3
$ expr 10 / -4
-2
$ expr 30 % 2
0
$ expr 15 % 6
3
$ expr -20 % 3
-2
```

変数を使って計算させる場合は次のようになる

```bash
VALUE=5
VALUE='expr $VALUE + 1'
```

### 数値の比較

数値の大小を比較するには、testコマンドを利用する。

```bash
test int1 -eq int2 # int1とint2が等しいとき真
test int1 -ne int2 # int1とint2が等しくないとき真
test int1 -lt int2 # int1がint2より小さいとき真
test int1 -le int2 # int1がint2より小さいか等しいとき真
test int1 -gt int2 # int1がint2より大きいとき真
test int1 -ge int2 # int1がint2より大きいか等しいとき真
```

### 浮動小数点を含む計算

浮動小数点を含む計算を行う場合にはbcコマンドが利用できる。

scaleという値をセットすることで小数点第何位まで出力するかを指定する。

```bash
echo "scale=n; num1 + num2" | bc # num1にnum2を加える
echo "scale=n; num1 - num2" | bc # num1からnum2を引く
echo "scale=n; num1 * num2" | bc # num1にnum2を掛ける
echo "scale=n; num1 / num2" | bc # num1をnum2で割る
```

```bash
$ echo "scale=3; 10 / 3" | bc
3.333

$ echo "scale=4; 10 / 3" | bc
3.3333

$ echo "scale=3; 3.33 * 3.1234" | bc
10.4009

$ echo "scale=7; 3.33 * 3.1234" | bc
10.400922

$ echo "3.5678 * 3" | bc
10.7034

$ echo "scale=2; 3.5678 * 3" | bc
10.7034
```

### 数値化どうかの判定

NUMBERに代入されている文字列に強引に1を加えている。もしNUMBERが数値だけからなる文字列であれば、実行終了ステータスは0か1になる。(普通は0を返しますが、計算結果が0の場合には1を返す)。NUMBERに数値以外の文字が入っていればexprがエラーを出し、終了ステータスは2以上になる。

```bash
expr "$NUMBER +  1 > /dev/null 2>&1"
if [ $? -lt 2]; then
  echo "Numeric"
else
  echo "Not Numeric"
fi
```

## 文字列の操作

### フィルタを使った文字列処理

```bash
$ STRING="abc def ghi"
$ STRING=`echo "$STRING" | sed -e "s/def/xyz/g"`
$ echo "$STRING"
abc xyz ghi
```

### 文字列の連結

```bash
# 変数を並べて書けばよい
$ STRING1=abc
$ STRING2=xyz
$ VAR=$STRING1$STRING2
$ echo $VAR
abcxyz

# 変数に直接文字列を繋ぐ場合は中括弧で括る
$ STRING=abc
$ VAR=${STRING}xyz
$ echo $VAR
abcxyc
```

### 文字列の長さ

下記コマンドを使うと、文字列の長さを得られる

`expr "string" : '.*'`

exprtは引数に:(コロン)があると、その右側と左側の文字列を比較して「戦闘から何文字まで等しいか」という値を返す。右側の`.*`はどんな文字列でも表現できるので、左側の文字列が何であれ同じものを表す表現となる。

```bash
$ STRING=abcdefghijklmnopqrstuvwxyz
$ NUMCHARS=`expr "$STRING" : '.*'`
$ echo $NUMCHARS
26
```

STRINGに:や*が1文字だけ代入されている場合、exprコマンドはエラーとなるため、case文などで対応すればよい

```bash
case "STRING" in
  ? ) echo 1 ;;
  * ) expr "$STRING" : '.*' ;;
esac
```

### 文字列の中の文字列

grepコマンドはある文字列がその名k何含まれているどうかを判定し、含まれていればその行を書き出す、という処理をする。実行終了ステータスとして、文字列が含まれていたら真を、そうでなかったら偽を返す。

```bash
# xyzが含まれているかどうか
echo abcdefgxyzaaa | grep xyz > /dev/null
if [ $? -eq 0]; then
  echo "include it."
else
  echo "Not include it."
fi
```

### 文字列の中の一部分の切り出し

コマンドの記法は下記となる。

`expr "string" : "regrexp\(regrexp\)regrexp"`

コロンの左側に、元になる文字列を書く。コロンの右側には正規表現を使って取り出すべき文字列を書く。\\(と\\)で囲まれた部分が取り出すべき文字列となる。

```bash
$ expr "abcdefghijklmn" : "a.*\(e..h\)i.*"
efgh
```

正規表現の表し方

```bash
.* 何もないものを含め、どんな文字列も表す
.  何か1文字(ヌル文字にも該当する)
*  直前の文字が0個以上並んだ文字列
\. ドット文字そのもの
\* アスタリスクそのもの
```

```
abc.*   abcで始まる文字列ならなんでも
.*abc   abcで終わる文字列ならなんでも
ab*c    abやabbcというように、aとcの間にbが0個以上ある文字れる
ab.*c   abで始まってcで終わる文字れる(abcも該当)
a....b  aで始まってbで終わる6文字
..*\.c  .cで終わる文字列(.cでなく、.cの前に少なくとも1文字ある)
```

```bash
expr "string" : "pattern\(.*\)"  patternという文字列より後
expr "string" : "\(.*\)pattern"  patternという文字列より前
expr "string" : "\(...\).*"      初めの三文字
expr "string" : ".*\(...\)"      後ろの三文字
expr "string" : "...\(.*\)"      初めの三文字を取り去った残り
expr "string" : "\(.*\)..."      後ろの三文字を取り去った残り
expr "string" : "\(.*\)"         その文字列全部をそのまま
```

# 第9章:シェル関数の例

# 第10章:シェルスクリプトの例

# 第11章:デバッグの手順、手法

## デバッグオプション

シェルスクリプトは普通下記のように実行する。

`$ scriptfile param1 param2`

これは次のようにして実行することも可能。このshコマンドには、オプションを付けて実行することができる。

`$ sh scriptfile param1 param2`

下記のようなシェルスクリプトがあると想定

```bash abc.sh
#!/bin/sh
ABC=0
for i in 1 2 3 4
do
  ABC=`expr $ABC + $i`
done
  echo $ABC
```

```bash
$ sh ./abc.sh # 普通に実行する
10

$ sh -v ./abc.sh # 何を実行するか標示されて、最後の結果が出力
#!/bin/sh
ABC=0
for i in 1 2 3 4
do
  ABC=`expr $ABC + $i`
done
expr $ABC + $i
expr $ABC + $i
expr $ABC + $i
expr $ABC + $i
echo $ABC
10

$ sh -x ./abc.sh # 実行している途中結果が標示される
+ ABC=0
+ for i in 1 2 3 4
++ expr 0 + 1
+ ABC=1
+ for i in 1 2 3 4
++ expr 1 + 2
+ ABC=3
+ for i in 1 2 3 4
++ expr 3 + 3
+ ABC=6
+ for i in 1 2 3 4
++ expr 6 + 4
+ ABC=10
+ echo 10
10

$ sh -vx ./abc.sh
#!/bin/sh
ABC=0
+ ABC=0
for i in 1 2 3 4
do
  ABC=`expr $ABC + $i`
done
+ for i in 1 2 3 4
expr $ABC + $i
++ expr 0 + 1
+ ABC=1
+ for i in 1 2 3 4
expr $ABC + $i
++ expr 1 + 2
+ ABC=3
+ for i in 1 2 3 4
expr $ABC + $i
++ expr 3 + 3
+ ABC=6
+ for i in 1 2 3 4
expr $ABC + $i
++ expr 6 + 4
+ ABC=10
  echo $ABC
+ echo 10
10
```

-vオプションを付けると、そのシェルスクリプトがこれからしようとしていることを標示する。

-xオプションは、シェルスクリプトがその中で実際に行ったことを(変数なら値を置き換えた状態で)標示していく。

これらのオプションはシェルスクリプト中のsetコマンドで組み込むことが可能。例えば、詳細な情報を取りたい所だけ抜き出す場合は下記のようにすれば良い。

```bash
$!/bin/sh
・・・・・
set -xv
・・・・・
・・・・・
set +xv
・・・・・
```

デバックでよく使用するオプション
```bash
e : スクリプト内部のコマンドの終了で追うコードが0以外のとき、終了する
i : 対話的モードで動作する
n : スクリプトを読み込むが実行はしない(構文チェックを行うだけ)
u : 未設定の変数を利用したとき、エラーにする
v : 何をしようとしているかを標示する
x : 実行するコマンドの内容を表示する
```