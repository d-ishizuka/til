# 演算子一覧

|演算子|使い方|意味|例|
|-|-|-|-|
|+|数値+数値|数値の和を計算する|SELECT 1+1 #2|
|+|日付+数値|日付を指定日数進める||
|-|数値-数値|数値の差を計算する|SELECT 2-3 #-1|
|-|日付-数値|日付を指定日数戻す||
|-|日付-日付|日付の差の日数を計算する||
|*|数値*数値|数値の積を計算する|SELECT 3*3 # 9|
|/|数値/数値|数値の商を計算する|SELECT 4/3 # 1.3333|
|%|数値%数値|数値の余りを計算する|SELECT 4%3 # 1|
|\|\||文字列\|\|文字列|文字列同士を連結する|SELECT "Hello" \|\| "A" # Hello A|

※ 文字列の結合はMySQLの場合は、concatを用いる

# 日付に関する関数
```
# NOW():現在時刻を表示する
SELECT NOW() #2022-02-10 19:01:12
```

```
# CURDATE():現在日時を表示する
SELECT CURDATE();
```

```
DATE_FORMAT(date, format)
SELECT DATE_FORMAT(NOW(), '%Y/%m/%d/%H:%i:%S') #2022/02/10/19:10:11
```

# 文字列演算子
## LENGTH, CHAR_LENGTH
- 文字列のバイト数(LENGTH), 文字数(CHAR_LENGTH)を取得する
```
SELECT LENGTH("ABC"); #3(3バイト)
SELECT LENGTH("あいう"); # 9(9バイト)
SELECT CHAR_LENGTH("ABC"); 3(3文字)
SELECT CHAR_LENGTH("あいう"); 3(3文字)
```

## TRIM, LTRIM, RTRIM
- 両側、左側、右側から空白を除去する
```
# LTRIMで左側の空白を文字列から除去する
SELECT LTRIMI(" ABC "); #「ABC　」と表示
SELECT RTRIM(" ABC "); #「 ABC」と表示
SLELCT TRIM(" ABC "); #「ABC」と表示
```

## REPLACE
- 文字列を置換する
```
SELECT REPLACE("I like apple", "apple", "ringo"); # I like ringoと表示
SELECT REPLACE(name, "Ms.", "Mrs") FROM users; # usersテーブルのnameカラムに含まれる"Ms.”を"Mrs”に置換する
```

## UPPER, LOWER
- 文字列を全て大文字、小文字にする
```
SELECT UPPER("apple"); # 「APPLE」と表示される
SELECT LOWER("Apple"); # 「apple」と表示される
SELECT UPPER(name) FROM users; #usersテーブルのnameカラムを大文字にして表示する
SELECT LOWER(name) FROM users; #usersテーブルのnameカラムを小文字にして表示する
```

## SUBSTRING, SUBSTR
- 文字列から一部を抽出する
- ```SUBSTRING(文字列, 抽出を開始する位置, 抽出する文字数)```
```
SELECT SUBSTR("apple", 2, 3); # 「ppl」と表示される
SELECT SUBSTRING(name, 3, 5) FROM users; # usersテーブルのnameカラムの３文字目から５文字分取り出す
```

## REVERSE
- 文字列を逆順にする
```
SELECT REVERSE("apple"); #「elppa」と表示される
SELECT REVERSE(name) FROM users; # usersテーブルのnameカラムを逆順にして表示する
```

# 数学関数
## ROUND
- ```ROUND(数値, 桁数)```
```
# 桁数を指定せず、小数点1桁を四捨五入
SELECT ROUND(3.14) # 3と表示

# 小数点以下1桁目まで残し、小数点２桁目を四捨五入
SELECT ROUND(3.14, 1); # 3.1と表示

# 整数１桁目を四捨五入
SELECT ROUND(956, -1); # 960と表示

# 小数点以下を切り捨て
SELECT FLOOR(3.14); # 3と表示

# 小数点以下切り上げ
SELECT CEILING(3.14) # 4と表示
```

# RAND
- 0 ~ 1までの小数のランダム値を取得する(0, 1は含まない)
```
# 0~1のランダム値
SELECT RAND(); # 0.405

# 0~9のランダム値
SELECT FLOOR(RAND() * 10);

# 2~11のランダムな値
SELECT FLOOR(RAND() * 10) + 2;
```

## POWER
- べき乗を計算する
- ```POWER(数値, 何乗するかの指定)
```
# 3の4乗する
SELECT POWER(3, 4) # 81と表示

# usersテーブルからBMI(体重/身長の2乗)を計算する
SELECT weight / POWER(height , 2) FROM users;
```

## COALESCE
- 最初に登場するNULLでない値を返す関数
```
SELECT COALESCE('A', 'B', 'C'); # Aと表示される
SELECT COALESCE(NULL, 'B', 'C'); # Bと表示される
# usersテーブルから取得して、column1, column2, column3のうちNULLでない最初の文字列を表示
SELECT COALESCE(column1, column2, column3) FROM users
```