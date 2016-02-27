<!-- MarkdownTOC -->

- gnuplotでファイル名やディレクトリパスを取得する
	- フルパス
	- ドライブ名
	- ドライブ名を除いたパス
	- バックスラッシュをスラッシュに変えたパス
	- ディレクトリパス
	- ファイル名
	- ファイルベースネーム
	- 拡張子
	- 関数をいつでも呼び出せるようにする

<!-- /MarkdownTOC -->







# gnuplotでファイル名やディレクトリパスを取得する

[gnuplotの便利な文字列関数](http://www.ss.scphys.kyoto-u.ac.jp/person/yonezawa/contents/program/gnuplot/string_function.html)で定義された関数を使って、gnuplot上でファイル名やディレクトリパスの取得を行う。
gnuplot version4.2以上で使用可能。
汎用的な使い方や関数の詳しい説明は上のページを見てください。
















```gnuplot:最終的な出力
# フルパスは   C:\home\gnuplot\substr.gp
# ドライブ名は   C
# ドライブ名を除いた部分は   home\gnuplot\substr.gp
# バックスラッシュをスラッシュに変えたパスは   C:/home/gnuplot/substr.gp
# ディレクトリパスは   C:\home\gnuplot
# ファイル名は   substr.gp
# ファイルベースネームは   substr
# 拡張子は   .gp
```



























## フルパス
**ARG0**
標準装備の標準的な変数。このファイルのフルパスを返す

```gnuplot:フルパス
print sprintf("フルパスは   %s",ARG0)

# __実行結果__________________________
#	フルパスは   C:\home\gnuplot\substr.gp
# ____________________________
```

































## ドライブ名
**substr_str_before(str, target)**
strの中でtargetよりも前の部分文字列を返す

```gnuplot:ドライブ名
substr_str_before(str, target)=substr(str, 1, strstrt(str,target)-1)

print sprintf("ドライブ名は   %s",substr_str_before(ARG0,":"))

# __実行結果__________________________
#	ドライブ名は   C
# ____________________________
```


































## ドライブ名を除いたパス
**substr_str_after(str, target)**
strの中でtargetよりも後の部分文字列を返す

```gnuplot:ドライブ名を除いたパス
substr_str_after(str, target)=substr(str, strstrt(str,target)+strlen(target), strlen(str))

print sprintf("ドライブ名を除いたパスは   %s",substr_str_after(ARG0,"\\"))

# __実行結果__________________________
#	ドライブ名を除いた部分は   home\gnuplot\substr.gp
# ____________________________
```





## バックスラッシュをスラッシュに変えたパス
**strsubst_sub(str, index, target, subst)**
再帰表現を利用して、strのindexより後の文字列のtargetをsubstに全て置換する

```gnuplot:バックスラッシュをスラッシュに変えたパス
strsubst_sub(str, index, target, subst)=\
	strstrt(str[index:],target)==0 ? str :\
	strsubst_sub( str[:index-1] . substr_str_before(str[index:],target) . \
	subst . substr_str_after(str[index:],target),\
	index+strstrt(str[index:],target)+strlen(subst)-strlen(target),target, subst)

print sprintf("バックスラッシュをスラッシュに変えたパスは   %s",strsubst_sub(ARG0,1,"\\","/"))

## __実行結果__________________________
##	バックスラッシュをスラッシュに変えたパスは   C:/home/gnuplot/substr.gp
## ____________________________
```

































## ディレクトリパス
**strstrlt(str,target)**
標準で備わっている関数`strstrt(str,target)`は最初の場所を返すのに対して
文字列strの中でtargetの最後に出てくる場所を返す

**max(x,y)= ( (x) > (y) ) ? (x) : (y)**
大きい方の値を返す関数を補助的に定義

**strstrlt_sub(str,index,target)**
strのindexより後の文字列の中でtargetが最後に出てくる場所を返す

**strstrlt(str,target)=strstrlt_sub(str,1,target)**
strの中でtargetが最後に出てくる場所を返す

```gnuplot:ディレクトリパス
# 大きい方の値を返す関数を補助的に定義
max(x,y)= ( (x) > (y) ) ? (x) : (y)
# strのindexより後の文字列の中でtargetが最後に出てくる場所を返す
strstrlt_sub(str,index,target)=(strstrt(str[index:],target)==0 ? \
	max(index-strlen(target),0) : \
	strstrlt_sub(str, index-1+strstrt(str[index:],target)+strlen(target),target))
# strの中でtargetが最後に出てくる場所を返す
strstrlt(str,target)=strstrlt_sub(str,1,target)




# __dirname(path)__________________________
# 文字列pathからディレクトリ名を返す。
# ____________________________
dirname(path)=strstrt(path, "\\")!= 0 ? substr(path, 1, strstrlt(path,"\\")-1) : ""

print sprintf("ディレクトリパスは   %s",dirname(ARG0))

# __実行結果__________________________
#	ディレクトリパスは   C:\home\gnuplot
# ____________________________
```





























## ファイル名
**remdirname(path)**
文字列pathからディレクトリ名を消したものを返す。

```gnuplot:ファイル名
remdirname(path)=substr(path,strstrlt(path,"\\")+1,strlen(path))

print sprintf("ファイル名は   %s",remdirname(ARG0))

## __実行結果__________________________
##	ファイル名は   substr.gp
## ____________________________
```












## ファイルベースネーム
**remext(*path*)**
文字列pathから拡張子を消したものを返す。

```gnuplot:ファイルベースネーム
# filenameからexentionを消したものを返す
remext(filename)=strstrt(filename, ".")==0 ? filename : substr(filename,1,strstrlt(filename, ".")-1)


print sprintf("ファイルベースネームは   %s",remext(remdirname(ARG0)))

## __実行結果__________________________
##	ファイルベースネームは   substr
## ____________________________
```
















## 拡張子
**substr_str_after(str, target)**


```gnuplot:拡張子
print sprintf("拡張子は   %s",substr_str_after(ARG0,remext(remdirname(ARG0))))

# __実行結果__________________________
#	拡張子は   .gp
# ____________________________
```








## 関数をいつでも呼び出せるようにする

以下のファイルを任意のディレクトリに置いて、gnuplot上で`load "<任意のパス>/substr.gp"`すればこれまで説明した関数が使えるようになる。

```gnuplot:substr.gp
## ____________________________
##gnuplotでファイル名やらディレクトリパスを取得する
##[gnuplotの便利な文字列関数](http://www.ss.scphys.kyoto-u.ac.jp/person/yonezawa/contents/program/gnuplot/string_function.html)で定義された関数を使って、gnuplot上でファイル名やディレクトリパスの取得を行う。
## ____________________________







## __substr_str_before(str, target)__________________________
## strの中でtargetよりも前の部分文字列を返す
## ____________________________
substr_str_before(str, target)=substr(str, 1, strstrt(str,target)-1)

# print sprintf("ドライブ名は   %s",substr_str_before(ARG0,":"))
##__実行結果__________________________
##	ドライブ名は   C
## ____________________________







## __substr_str_after(str, target)__________________________
## strの中でtargetよりも後の部分文字列を返す
## ____________________________
substr_str_after(str, target)=substr(str, strstrt(str,target)+strlen(target), strlen(str))

# print sprintf("ドライブ名を除いたパスは   %s",substr_str_after(ARG0,"\\"))
##__実行結果__________________________
##	ドライブ名を除いた部分は   home\gnuplot\WhereamI.gp
## ____________________________







## __strsubst_sub(str, index, target, subst)__________________________
## 再帰表現を利用して、strのindexより後の文字列のtargetをsubstに全て置換する
## ____________________________
strsubst_sub(str, index, target, subst)=\
	strstrt(str[index:],target)==0 ? str :\
	strsubst_sub( str[:index-1] . substr_str_before(str[index:],target) . \
	subst . substr_str_after(str[index:],target),\
	index+strstrt(str[index:],target)+strlen(subst)-strlen(target),target, subst)

# print sprintf("バックスラッシュをスラッシュに変えたパスは   %s",strsubst_sub(ARG0,1,"\\","/"))
## __実行結果__________________________
##	バックスラッシュをスラッシュに変えたパスは   C:/home/gnuplot/WhereamI.gp
## ____________________________







## __strstrlt(str,target)__________________________
## デフォルト関数`strstrt(str,target)`は最初の場所を返すのに対して
## 文字列strの中でtargetの最後に出てくる場所を返す
## ____________________________
#大きい方の値を返す関数を補助的に定義
max(x,y)= ( (x) > (y) ) ? (x) : (y)
# strのindexより後の文字列の中でtargetが最後に出てくる場所を返す
strstrlt_sub(str,index,target)=(strstrt(str[index:],target)==0 ? \
	max(index-strlen(target),0) : \
	strstrlt_sub(str, index-1+strstrt(str[index:],target)+strlen(target),target))
# strの中でtargetが最後に出てくる場所を返す
strstrlt(str,target)=strstrlt_sub(str,1,target)






## __dirname(path)__________________________
## 文字列pathからディレクトリ名を返す。
## ____________________________
dirname(path)=strstrt(path, "\\")!= 0 ? substr(path, 1, strstrlt(path,"\\")-1) : ""

# print sprintf("ディレクトリパスは   %s",dirname(ARG0))
## __実行結果__________________________
##	ディレクトリパスは   C:\home\gnuplot
##____________________________






##__remdirname(path)__________________________
##文字列pathからディレクトリ名を消したものを返す。
##____________________________
remdirname(path)=substr(path,strstrlt(path,"\\")+1,strlen(path))

# print sprintf("ファイル名は   %s",remdirname(ARG0))
## __実行結果__________________________
##	ファイル名は   WhereamI.gp
##____________________________



##__remext(path)__________________________
##文字列pathから拡張子を消したものを返す。
##____________________________
remext(filename)=strstrt(filename, ".")==0 ? filename : substr(filename,1,strstrlt(filename, ".")-1)

# print sprintf("ファイルベースネームは   %s",remext(remdirname(ARG0)))
## __実行結果__________________________
##	ファイルベースネームは   WhereamI
##____________________________
# print sprintf("拡張子は   %s",substr_str_after(ARG0,remext(remdirname(ARG0))))
## __実行結果__________________________
##	拡張子は   .gp
##____________________________
```
