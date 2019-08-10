@snap[north-west text-08]
### やったこと/わかったこと

- gitpichの使い方を覚える  
	-めっちゃ便利だと思った。MarkDown作って、github.comで公開するだけでさらっとプレゼン作れる。  
- [Practical Binary Analysis](https://www.amazon.co.jp/gp/product/B07BPKWJVT)を第6章まで読む。だいたいkindleで220ページぐらい読んだ？  
	- ELFの.got/.plt/.got.pltの仕組みがあらためてわかる。  
	- GOTによるDynamicLinkingがセキュリティの要請からも必要であることがわかった。  
	- DisassemblyについてDynamic DisassemblyとかRecursive Disassemblyというのがあり、その利点がわかった。  
@snapend
---

![](https://images-fe.ssl-images-amazon.com/images/I/51SYed0K4NL.jpg)

---

### 書いたコード
- ELFのdebigging symbol tableを読み下しできるライブラリの写経をした。
- ptraceで/bin/lsのsystem callトレースして、Writeシステムコールにフックするサンプルを書いた。

