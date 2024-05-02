## ロックについて
- 作成したサービスを通してDBを利用する人が沢山いる
  - テーブルが全く同じタイミングで複数の場所から更新されたら？
  - テーブルからレコードを取得している最中に、レコードが削除されたら？
- ロックとは？
  - DBのテーブルや行を固定して、別のトランザクションからテーブルや行の参照・更新ができないようにすること
- ロックの種類
  - 共有ロック...他のトランザクション(ユーザー)から、参照はできるが更新はできなくなるロック
  - 排他ロック...他のトランザクション(ユーザー)から、参照も更新もできなくなるロック

## 複数のトランザクションがロックをかけた場合
- 共有ロックは、複数のトランザクションからかけられる
- 排他ロックをかけられると、別のトランザクションから共有ロック、排他ロックをかけられなくなる
  - 排他ロックは1つのトランザクションからのみかけられる

## 行ロックとテーブルロック
- 行ロックは、特定の行だけをロックする
- テーブルロックは、テーブル全体をロックする

```
# トランザクションの開始
START TRANSACTION;
UPDATE table SET name = "" WHERE id = 12; # id = 12の行だけロックがかかる
COMMIT / ROLLBACK; (ロックが解除される)
```

## SQLとロック
### SELECT
- 明示的にロックした場合や、主キーまたはユニークカラムで絞り込んだ場合のみ行ロックがかかる
```
テーブルロック
# 共有ロック(参照できるが、更新できない)
SELECT * FROM table_name FOR SHARE

# 排他ロック(参照も更新もできない)
SELECT * FROM table_name FOR UPDATE
```
```
行ロック(主キーまたはユニークカラムで絞り込んだ場合のみ行ロックがかかる)
# 共有ロック(参照できるが、更新できない)
SELECT * FROM table_name WHERE id = 1 FOR SHARE

# 排他ロック(参照も更新もできない)
SELECT * FROM table_name WHERE id = 1 FOR SHARE
```

### UPDATE, DELETE, INSERT
- **UPDATE, DELETE, INSERTを実行すると、自動的にテーブル・レコードが排他ロックされる**
- **主キーではないカラムで絞り込んでUPDATE, DELETEをかけようとすると、テーブル全体にロックがかかる！**
  - 実際には、トランザクションの中でUPDATE, DELETEしていない場合が多いので、その場合は一瞬ロックがかかるだけだが、トランザクションの中でやる場合だと全体を止めるのでクエリの性能が悪くなる
```
テーブルロック
# 排他ロック(参照も更新もできない)
UPDATE table_name SET name = "AA";
# 排他ロック(参照も更新もできない)
DELETE FROM table_name WHERE name = "AA"; # nameは主キーではないので、テーブル全体をロックしている！！
```
```
行ロック(主キーまたはユニークカラムで絞り込んだ場合のみ行ロックがかかる)
# 排他ロック(参照も更新もできない)
UPDATE table_name SET name = "AA";
DELETE FROM table_name WHERE id = 1;
INSERT INTO table_name VALUES(x, x, x)
```

## 明示的にテーブルをロックする
- DBにはトランザクション以外にも、テーブルをロックするコマンドが存在する
```
LOCK TABLE テーブル名 READ
・実行したセッションは、テーブルを読み込むことができるが、書き込むことはできない
・他のセッションも、テーブルを読み込むことはできるが、書き込むことはできない

LOCK TABLE テーブル名 READ
```
```
LOCK TABLE テーブル名 WRITE
・実行したセッションは、テーブルを読み込むことも書き込むこともできる
・他のセッションは、テーブルを読み込むことも書き込むことも**できない**

LOCK TABLE テーブル名 READ
```
```
# 現セッションの保有するテーブルロックを全て解除する
UNLOCK TABLES
```

## デッドロック
- 2つのセッションが互いの更新対象のテーブルをロックしていて、処理が進まない状態のこと
  - プログラムA ：テーブルAをロックして、次にテーブルBを更新する
  - プログラムB ：テーブルBをロックして、次にテーブルAを更新する
    - ロックするタイミングが重なるとデットロックが発生する
    - 対応として、ロックするテーブルの順番を揃える。MySQLの場合は、強制的にROLLBACKが発生するが、ロックして止まるものもある。