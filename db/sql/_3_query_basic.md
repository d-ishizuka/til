## INSERT
- INSERT:1
  - カラム名を指定せずに、一番目のカラムから順に値を格納していく
  - ```INSERT INTO テーブル名 VALUES (value1, value2...);```
- INSERT:2
  - カラム名を指定して、値を格納する
  - ```INSERT INTO テーブル名(column1, column2, ...) VALUES (value1, value2, ...);```

## SELECT
- 全カラムを取得するSELECT文
  - ```SELECT * FROM テーブル名```
- カラムを指定したSELECT文
  - ```SELECT column1, column2, ... FROM テーブル名;```
- カラムを別名にして表示するSELECT文(AS)
  - ASを利用するとカラム名を別名で表示できる
  - **特に別のテーブルと結合する時に使用する**
  - ```SELECT column1 AS other_column1, column2, ... FROM テーブル名;``` 

## WHERE
- WHERE1
  - テーブルからnameがTaroのものだけを取得する
    - ```WHERE name = "Taro;"```
- WHERE2
  - テーブルからageが10より大きいものだけを取得する
    - ```WHERE age > 10;```

## UPDATE
- UPDATE1
  - テーブルの**全データ**の特定カラムを更新する(危険!)
    - ```UPDATE テーブル名 SET column1 = xx, column2 = yy;```
- UPDATE2
  - テーブルのnameがTaroのデータの特定カラムを更新する
    - ```UPDATE テーブル名 SET column1 = xx, column2 = yy WHERE name = "Taro"```

## DELETE
- DELETE1
  - テーブルの**全データ**を削除する(危険!)
    - ```DELETE FROM テーブル名;```
- DELETE2
  - テーブルのnameがTaroのデータのレコードを削除する
    - ```DELETE FROM テーブル名 WHERE name = "Taro";```

## DISTINCT
- column1から重複を排除して表示する
  - ```SELECT DISTINCT column1 FROM テーブル名;```
- 複数カラムのDISTINCT
  - column1, column2の組み合わせから重複を排除して表示する
  - ```SELECT DISTINCT column1, column2 FROM テーブル名;```

## ORDER BY
- ORDER BY1
  - column1で昇順に並び替える
    - ```ORDER BY column1;```
- ORDER BY2
  - column1, column2で昇順に並び替える
    - ```ORDER BY column1, column2;```
- ORDER BY3
  - column1で降順に並び替える
    - ```ORDER BY column1 DESC;```

## LIMIT, OFFSET
- 指定した行数分だけレコードを取り出す
  - ```SELECT * FROM user_name LIMIT 10;```
- 特定の行数だけ飛ばしたLIMIT
  - 5行飛ばして、10行分レコードを取り出す
  - ```SELECT * FROM user_name LIMIT 5, 10```
- 特定の行数だけ飛ばしたLIMIT
  - 5行飛ばして、10行分レコードを取り出す
  - ```SELECT * FROM user_name LIMIT 10 OFFSET 5```

## TRUNCATE
- 特定のテーブルを空にするSQL文
- TRUNCATE文
  - 全部削除する場合はこちらを使うのが一般的
  - ```TRUNCATE テーブル名 #テーブルの中のデータを全て削除する```
- DELETE文
  - ```DELETE FROM テーブル名 #テーブルの中のデータを全て削除する```
- ハイウォーターマーク
  - テーブルにどこまでレコードが入っているか示すマークのこと
  - レコードをDELETEしてもハイウォーターマークが動かないのでディスク解放されないが、TRUNCATEするとハイウォータマークが最初に戻る

|DELETE|TRUNCATE|
|-|-|
|データをロールバックできる|データを完全削除するのでロールバックできない|
|処理速度は遅い|処理速度はDELETEよりも速い|
|使用しているディスク領域が解放されない|使用しているディスク領域が解放される|


