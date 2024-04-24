## テーブルの作成
```
CREATE TABLE テーブル名(
  column1 column_type,
  column2 column_type,
  ...
)

例1）
CREATE TABLE users(
  id INT,
  name VARCHAR(10),-- 名前、可変長文字列
  age INT,
  phone_number CHAR(13),-- 固定長文字列
  message TEXT
)

例２）主キー付き
CREATE TABLE users(
  id INT PRIMARY KEY,
  name VARCHAR(10),-- 名前、可変長文字列
  age INT,
  phone_number CHAR(13),-- 固定長文字列
  message TEXT
)
```

## テーブルを削除
```
mysql > DROP TABLE users;
```

## テーブル確認
```
mysql > DESCRIBE users;
mysql > SHOW TABLES;
```

## テーブル定義の変更
```
# テーブル名の変更(RENAME TO)
mysql > ALTER TABLE users RENAME TO users_table;
```
```
# カラムの削除(DROP COLUMN)
mysql > ALTER TABLE users DROP COLUMN class_no;
```
```
# nameカラムの追加する
ALTER TABLE table_name
ADD
  new_column_name_column_definition
  [FIRST | AFTER column_name]

- FIRST : テーブルの一番最初の列にカラムを追加する
- AFTER : テーブルの指定した列の後にカラムを追加する
※ デフォルトだと、一番最後の列に追加される

mysql > ALTER TABLE users ADD message TEXT AFTER name;
```
```
# カラム定義の変更(MODIFY)
mysql > ALTER TABLE users MODIFY name CHAR(255)
```
```
# カラム名・場所定義の変更(CHANGE COLUMN)
ALTER TABLE table_name
CHANGE COLUMN original_name new_name definition
[FIRST | AFTER column_name]

mysql > ALTER TABLE users
CHANGE COLUMN id id_new INT
AFTER message;
```
```
# 主キーの削除(DROP PRIMARY KEY)
ALTER TABLE users DROP PRIMARY KEY
```

## 代表的なデータ型
|データタイプ名|データ種類|
|-|-|
|INT|整数値|
|DECIMAL|小数|
|BOOLEAN|真偽値|
|CHAR|固定長文字列(最大255バイト：MySQL8の場合)|
|VARCHAR|可変長文字列(最大16383バイト：MySQL8の場合)|
|DATE|日付|
|DATETIME|時刻|
|TEXT|65536文字まで|

## 固定長文字列(CHAR)と可変長文字列(VARCHAR)の違い
|CHAR|VARCHAR|
|-|-|
|指定した領域分ディスク領域が確保される|挿入するデータが指定したサイズよりも小さい場合は、ディスク領域が縮小される|
|ディスク使用量は**多くなる**|ディスク使用量は**少なくなる**|
|データベース操作の**パフォーマンスが良い**(決まった領域にアクセスすればいいため)|データベース操作の**パフォーマンスが悪い**|
|末尾にスペースをつけて、データを格納した場合、**そのスペースは削除される**|末尾にスペースをつけて、データを格納した場合、**そのスペースは削除されない**|
|データの長さがある程度固定されていて、変化が少ない場合に用いられる(電話番号、銀行コード)|データの長さが格納するデータによって異なる場合に用いられる(住所、テキストメッセージ)|