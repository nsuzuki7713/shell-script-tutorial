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
