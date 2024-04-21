# Tutorial

## mysqlコマンドクライアントの呼び出し
- 基本構文
  - ```mysql [OPTINS] [databasename]```
#### OPTIONSの一部
| short | long | description |
| ---- | ---- | ---- |
| -u | --user=ユーザー名 | MySQLサーバーに接続するためのユーザー名を指定する |
| -p | --password=パスワード文字列 | MySQLサーバーに接続するためのパスワードを指定する。通常-pまで入力したら、パスワードは別途入力する |
| -h | --host=ホスト名 | 接続先のMySQLサーバーのホスト名を入力する |
| -p | --port=# | 接続先のMySQLサーバーのポート番号を指定する(TCP接続時)。省略時は、デフォルトポート3306になる。|

※ その他のOPTIONSは、リファレンス参照

- サーバー接続
  - ```mysql -umyuser -p -hlocalhost```
  - localhostで立ち上がっているMySQLサーバーへ接続する場合

## mysqlコマンドラインクライアントでの操作
- 命令の書き方
  - 命令の終わりは必ず「;」で終了し、Enterを押すと実行される
  - 命令文中の空白は、幾つ入れても動くし、改行を複数行入れても動く
- 結果を縦に表示する
  - 「;」の代わりに```\G```を使うことで、縦向きに表示できる
    - ```SELECT * FROM pref_info LIMIT 2\G```
- クライアントの終了
  - ```exit```, ```quit```, ```\q```
- 対話モードとバッチモード
  - 対話モード : ```mysql>```プロンプトに対してコマンドを入力していく使い方
  - バッチモード : 与えられたSQL文などをmysqlクライアントを通して実行し、終了する利用方法。あらかじめファイルに記述しておいたSQL文を実行したり、コマンドラインに直接SQLを記述する方法がある。
    - ```echo "SELECT * FROM mypref LIMIT 2;" mysql -uroot -p mydatabase```
    - ```cat sql.txt | mysql -urrot -p mydatabase```
    - ```mysql -uroot -p mydatabase < sql.txt```
    - ```echo "SELECT * FROM mypref LIMIT 2;" mysql -uroot -p mydatabase > myoutput.txt```

## SQLチュートリアル
- データベースオブジェクトの生成と破棄


## 参照
- MySQL徹底入門 第4版　MySQL 8.0対応　1-２章
