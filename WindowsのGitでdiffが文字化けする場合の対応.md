## 環境

OS : Windows11  
シェル : Windowsターミナル + PowerShell  
ツール : Git for Windows  

## 現象

SJIS のファイルの diff を見ようとすると文字化けが発生する

```diff
PS C:\work\test> git diff
diff --git a/fruit_sjis.txt b/fruit_sjis.txt
index 3c85617..6a92bf9 100644
--- a/fruit_sjis.txt
+++ b/fruit_sjis.txt
@@ -1,2 +1,3 @@
 apple : <97>ь<E7>
 banana : <83>o<83>i<83>i
+chocolate : <83>`<83><87><83>R<83><8C><81>[<83>g^M
```

## 試したこと

[git diff や git status での日本語の文字化けを防ぐ](https://maku77.github.io/git/settings/garbling.html)
の通りにしてみたが、ダメだった

## 解決策

git diff の出力を nkf に通してみたところ、文字化けが発生しなくなった

※nkf とは、文字コードを良い感じに判別して変換してくれるツール

1. nkf を <https://www.vector.co.jp/soft/dl/win95/util/se295331.html> からダウンロードして展開
2. nkf のパスを通す
    1. 適当なディレクトリを作成する(何でもいいけど、ここでは `C:\Tools\nkf` とする)
    2. 展開したディレクトリから `nkfwin\vc2005\win32(98,Me,NT,2000,XP,Vista,7)Windows-31J` の `nkf32.exe` を `C:\Tools\nkf` にコピー
    3. `nkf32.exe` を `nkf.exe` にリネーム
    4. ユーザー環境変数の Path に `C:\Tools\nkf` を追加
    5. シェルを開きなおして、`nkf --version` でバージョンが表示されればOK
3. `git config --global core.pager "nkf -w8 | less"` を実行

対応後 SJISのファイルの日本語がちゃんと表示されるようになった!

```diff
PS C:\work\test> git diff
 diff --git a/fruit_sjis.txt b/fruit_sjis.txt
index 3c85617..6a92bf9 100644
--- a/fruit_sjis.txt
+++ b/fruit_sjis.txt
@@ -1,2 +1,3 @@
 apple : 林檎
 banana : バナナ
+chocolate : チョコレート^M
```


## 参考サイト

- [git diff の文字化け対応](https://mistymagich.wordpress.com/2013/04/20/git-diff-%E3%81%AE%E6%96%87%E5%AD%97%E5%8C%96%E3%81%91%E5%AF%BE%E5%BF%9C/)

## [Optional] 文字コードと改行コードについて

よほどの理由が無い限り、文字コードは UTF-8 で統一した方が良い  

SJIS を使用してしまうと、Azure DevOps 上で差分を見る際に日本語が全て文字化けしたりする  
(SJISはMicrosoftが作った規格なのだから、それくらい対応して欲しいのだが...)  

また、改行コードが CRLF だと Git で差分を見るときに `^M` が末尾に付いて鬱陶しいなどの弊害があるので、改行コードは LF に統一した方が良い  
.gitattributes をリポジトリ内に置いておくことで改行コードを LF に強制できる

```shell:.gitattributes
* text=auto eol=lf
*.{cmd,[cC][mM][dD]} text eol=crlf
*.{bat,[bB][aA][tT]} text eol=crlf
```

エディタ側でも改行コードの設定をしておくと良い

```json:.vscode/settings.json
{
    "files.eol": "\n",
}
```
