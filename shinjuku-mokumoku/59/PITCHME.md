@snap[north-west]
### やったこと/わかったこと

- debian sidのマシンから、wifiのつなぎ方がわかる。  
- gitpitchの使い方を覚える。
- wifiのつなぎ方のLT書いたら時間がなくなってしまった...😓  

@snapend

---
# 某所のwifiの接続の仕組みについて
---
@snap[north-west]
### はじめに
最初debian sidから某所のwifiに接続する方法がわかりませんでした。  
ここでは無事接続できるようになりましたので、顛末を喋ります。  

@snapend
---
### よくあるSSID/PASSのみのwifi接続
wpa2+psk(よくあるSSID/Pass認証）の提供ならdebian sidの場合、何も苦労せずに繋がります。  
---
@snap[west span-40 text-center]
Gnome3のwifiメニュー  
![gnome3 wifi menu](/asset/59/gnome-wifi-modified.png)
@snapend
@snap[east span-40 text-center]
wpa supplicantで直接接続
```bash
cat /etc/network/interfaces
iface wlan0_wifi inet dhcp
	wpa-conf /etc/wpa_supplicant/wpa_supplicant_wifi.conf
cat /etc/wpa_supplicant/wpa_supplicant_wifi.conf
network={
     ssid="XXXXXX"
     psk="YYYYYYY"
}
sudo ifup wlp5s0=wlan0_wifi
```
@snapend
---
@snap[north-west]
### 某所のwifi
某所のwifiはwpa2+pskの他に、Web認証(captive portal)があります。
実はdebian sidはこのようなWeb認証はそのままではうまく動きません。  
つないでも、  
* ブラウザが立ち上がらない  
* ブラウザで適当なページをアクセスしても通信遮断されたままのように見える  
ということが起きなにもできません。  
@snapend
---
@snap[north-west]
### 最初に発見したつなぎ方
- Step1. ノートPCをwindows10に切り替えて一度wifiに接続する。  
- Step2. ノートPCをdebian sidに切り替えて、先述の通常のwifi接続をする。  

一応これでdebian sidはWeb認証がStep1.で完了しているので、Web認証無しでwifiにつながるようになります。  
@snapend
---
@snap[north-west]
### でも...
wifiが切れると、場合によってはStep1.からやり直しになることがあります。
いちいちwifi切れる度にwindows10へ切り替えるのは嫌ですよね。  
@snapend
---
@snap[north-west]
### web認証(captive portal)とは
認証の仕組みの大筋は次のとおりです。  
![](https://www.hitachi-solutions.co.jp/aruba/sp/guide/tech/img/img12_lan_guide.jpg)  　

wifi機材により流儀が異なるので、うまく動くとは限りません。  
手元のdebian sidではwifiに接続できるものの、直後のブラウザは何も通信しません。
@snapend
---
@snap[north-west]
### 解析開始
まず、Web認証をしている機材がどんな機材かを調べました。  
前項のやりかたでwifiをつないでおき、macアドレスから機材を引き当てます。  
```bash
netstat -rn
カーネルIP経路テーブル
受信先サイト    ゲートウェイ    ネットマスク   フラグ   MSS Window  irtt インタフェース
0.0.0.0         192.168.XX.XXX  0.0.0.0         UG        0 0          0 wlp5s0

sudo arp -a | fgrep 192.168.XX.XXX (↑のゲートウェイのIP)
? (192.168.XX.XXX) at ac:44:f2:XX:XX:XX [ether] on wlp5s0
```
このあと、googleってac:44:f2のMACアドレスはどこのメーカかを引き当てます。 
YAMAHA CORPORATIONが引き当たりました！  
@snapend
---
@snap[north-west] 
### 機材固有のWeb認証方式を調べる
今度はYAMAHAのルータ製品でWeb認証に関するマニュアルがないか？を引き当てます。  
ありました。  
[YAMAHA WLX313マニュアル](http://www.rtpro.yamaha.co.jp/AP/docs/wlx313/captive_portal.html)  
@snapend
---
@snap[north-west] 
wifiがつながったあと、HTTP通信を実施しないとWeb認証が始まらないとあります。  
![](http://www.rtpro.yamaha.co.jp/AP/docs/wlx313/images/captive_portal/cp_overview.png)  
@snapend
---
@snap[north-west]
昨今googleなど昨今のWebサイトはTLS1.2化が進んでおり、ページはHTTPS通信となってしまいます。このため、debian sid上ではHTTPの通信を行わかなかったので、Web認証が始まらなかったとなります。  
@snapend
---
@snap[north-west]
### なぜwindows10/mac/スマホはWeb認証できるのか？
これらのOSは、Web認証に対応するため、ひそかにHTTP通信をする機能が備わっていました。  
これはCaptive Portal Detection Strategyという手法名なのだそうです。  
@snapend
---
@snap[north-west]
### 各OSのCaptive Portal Detection Strategy
![](https://cdn-ak.f.st-hatena.com/images/fotolife/a/ao0780/20170221/20170221100614.png)  
@snapend
---
@snap[north-west]
### じゃあdebian sidでは...
http通信をすればRedirectが起き、Web認証画面まで遷移できるはずなので、
debian sidでやってみました。  
- Step1. wifiをSSID/Passで接続する。  
@snapend
---
@snap[north-west]
- Step2. コマンドラインから  
curl -v http://www.google.comを実行。  
```http
$ curl -v http://www.google.co.jp
...中略...
GET / HTTP/1.1
Host: www.google.co.jp
User-Agent: curl/7.65.1
Accept: */*
 
* Mark bundle as not supporting multiuse
HTTP/1.1 302 Moved Temporarily
Connection: close
Location: https://portal.hoge.fuga/logon?
wlan_id=XXXXXXX-YYYY-XXXX-YYYY-XXXXXXXXX&
ap_mac=ZZZZZZZZZZ&client_mac=AAAAAAAA&
url=http%3A%2F%2Fwww.google.co.jp%2F
```
@snapend
---
@snap[north-west]
- Step3. Locationで示されるURLをchromeにコピペ。無事Web認証画面が出てくるので必要事項を答える。  
- Step4. 無事debian sidでもWifiが利用できるようになりましたー🙆  
@snapend
---
### おわりに
debian sidでwifiとweb認証(captive portal)をうまく扱う方法が分かってよかったです。
