## テーブルの結合について
### 1対1の結合
- 複数のテーブルが１レコードに付き１レコードに紐づく結合
  - e.g. 国に対して、首都が１レコードずつ紐づく

Countriesテーブル
|id|country|
|-|-|
|1|Japan|
|2|China|

Capitalsテーブル
|id|capital|country_id|
|-|-|-|
|1|Tokyo|1|
|2|Beijing|2|

### 1対多の結合
- 一方のテーブルの１レコードに対して、もう片方のテーブルの複数レコードが紐づく

Ordersテーブル
|id|date|amount|
|-|-|-|
|1|20210101|5|

OrderDetailsテーブル
|id|order_id|quantity|
|-|-|-|
|1|1|1|
|2|1|2|
|3|1|2|

### 多対多の結合
- 各テーブルがお互いのテーブルに対して複数のレコードに紐づく結合
- 以下の例は、本には複数の著者が存在し、著者は複数の本を執筆している場合

Bookテーブル
|id|name|
|-|-|
|1|緑の本|
|2|青の本|

中間テーブル
|book_id|author_id|
|-|-|
|1|1|
|1|2|
|2|1|
|2|2|

Authorsテーブル
|id|name|
|-|-|
|1|田中|
|2|佐藤|

## JOIN
- 特定のカラム同士が等しいレコード同士を、テーブル間で結合するSQL(複数のテーブルを列方向に結合する)

### INNER JOINについて
- 複数のテーブルを指定した条件が正しいレコードのみ結合して取り出すSQL
```
INNER JOIN 接続するテーブル名 ON 接続する条件
```
```
SELECT
  or.id,
  ct.customer_nmae,
  or.order_date
FROM
  orders AS or
  INNER JOIN
    customers AS ct
  ON
    or.customer_id = ct.customer_id; 

# ONでは、＝以外の条件でも結合可能(比較演算子とかでもOK)。trueが返ってきた時に結合できる認識。
# 結合相手がいない場合は、返ってこない → 結合相手がいない場合NULLで補完して返して欲しい場合は、LEFT(OUTER) JOINを使う。
```

### LEFT (OUTER) JOINについて
- 複数のテーブルを結合して、左のテーブルは全てのレコードを所得して、右のテーブルからは紐付けのできたレコードのみを取り出し、それ以外はNULL
-  LEFT JOINのLEFTは、左側のテーブルが基準・基本となると言う意味(左側のテーブルにデータをくっつけるイメージ)
```
SELECT
  or.id,
  ct.customer_name,
  or.order_data
FROM
  orders AS or　# LEFT JOINではこのテーブルが基本
  LEFT JOIN
    customers AS ct
  ON
    or.customer_id = ct.customer_id;

# 左のテーブルorders(or)は全てのレコードを取得し、customers(ct)はor.customer_id = ct.customer_idで紐づけられたものだけを取得して、それ以外はNULLとする。
```

### RIGHT (OUTER) JOINについて
```
SELECT
  or.id,
  ct.customer_name,
  or.order_data
FROM
  orders AS or
  RIGHT JOIN 
    customers AS ct # RIGHT JOINではこのテーブルが基本
  ON
    or.customer_id = ct.customer_id;
# 右のテーブルcustomers(ct)は全てのレコードを取得し、orders(or)はor.customer_id = ct.customer_idで紐づけられたものだけを取得して、それ以外はNULLとする。
# 基本的には、LEFT JOINをよく使う
```

### 3つのテーブルを紐づける
- 先に結合されたテーブルを1つのテーブルだとみなして、さらにもう1つのテーブルをくっつける
```
SELECT * FROM students AS std
LEFT JOIN
  enrollments AS enr
ON 
  std.id = enr.student_id # ここまでで1つのテーブルだと思ってさらに結合する
LEFT JOIN
  classes AS cs
ON
  enr.class_id = cs.id;
```

### FULL OUTER JOIN
- 複数のテーブルを結合して、結合できなかった行はNULLと表示する結合方法
```
SELECT
  or.id,
  ct.customer_name,
  or.order_date
FROM
  orders AS or
  FULL OUTER JOIN
    customers AS ct
  ON
    or.customer_id = ct.customer_id;
# 両方のテーブルorders(or)とcustomers(ct)から全てのレコードを取得し、or.customer_id = ct.customer_idで紐づけられなかったものはNULLとする
```

## 複雑な結合について
- 4つのテーブルを結合して
- customers.idで並び替えて
- coustomers.idが10〜19で
- orders.order_dateが2020-08-01よりあと

```
# WHEREに関しては、FROMの部分でサブクエリとして組み込むこともできる
SELECT
  *
FROM
  customers AS ct
INNER JOIN
  orders AS od
ON
  ct.id = od.customer_id
INNER JOIN
  items AS it
ON
  od.item_id = it.id
INNER JOIN
  stores as st
ON
  it.store_id = st.id
WHERE
  ct.id BETWEEN 10 AND 19
AND
  od.order_date > '2020-08-01'
ORDER BY
  ct.id;

# GROUP BYとの紐付け
SELECT * FROM customers AS ct
INNER JOIN
 (SELECT customer_id, SUM(order_amount * order_price) AS summary_price FROM orders GROUP BY customer_id) AS order_summary
ON
  ct.id = order_summary.customer_id;
```

## 自己結合(SELF JOIN)
- 同一のテーブルを結合する結合方法
- 同一のテーブルなので必ず名前をつけるようにする
```
SELECT
  CONCAT(emp1.last_name, emp1.first_name) AS "部下の名前",
  COALESCE(CONCAT(emp2.last_name, emp2.first_name), "該当なし") AS "上司の名前"
FROM
  employees AS emp1
LEFT JOIN
  employees AS emp2
ON
  emp1.manager_id = emp2.id;
```

## 交差結合(CROSS JOIN)
- 2つのテーブルのデータの全ての組み合わせを取得するSQL
 Colorsテーブル
 |COLOR|
 |-|
 |Blue|
 |Green|

 Sizesテーブル
 |SIZE|
 |-|
 |Small|
 |Medium|
 |Large|

 交差結合後
 |COLOR|SIZE|
 |-|-|
 |Blue|Small|
 |Blue|Medium|
 |Blue|Large|
 |Green|Small|
 |Green|Medium|
 |Green|Large|


```
# 書き方1
SELECT (カラム) FROM テーブル1 AS 別名1
CROSS JOIN テーブル2 AS 別名2
(ON 結合条件)

# 書き方2(古い結合の書き方)
select * from employees AS emp1, employees AS emp2;
```
```
SELECT
  *,
  CASE
    WHEN cs.age > summary_customers.avg_age THEN "○"
    ELSE "×"
  END AS "平均年齢よりも年齢が高いか"
FROM customers AS cs
CROSS JOIN(
  SELECT AVG(age) AS avg_age FROM customers
) AS summary_customers;
```

## WITH
- WITHの後に記述したSQLの実行結果を、一時的なテーブルに格納する。**可読性の高い構文**
- 計算量を下げるには、まず絞り込み(where)に関する一時的なテーブルを作ってからJOINやGROUP BYするテーブルを作るのがいい
  - SQLのオプティマイザーがいい感じにやってくれることもある
```
WITH 中間テーブル名1 AS (
  SELECT xx
), 中間テーブル名2 AS (
  SELECT 〇〇
)
SELECT *
FROM テーブル
INNER JOIN 中間テーブル1 ON テーブル.id = 中間テーブル1.id
INNER JOIN 中間テーブル2 ON 中間テーブル1.id = 中間テーブル2.id;
```

```
# WITHを使った場合との比較

# WITHを使わない場合
SELECT
  *
FROM
  employees AS e
INNER JOIN
  departments AS d
ON
  e.department_id = d.id
WHERE
  d.name = "営業部";

# WITHを使う場合
WITH tmp_departments AS(
  SELECT * FROM departments WHERE name = "営業部"
)
SELECT * FROM employees AS e INNER JOIN tmp_departments
ON e.department_id = tmp_departments.id;
```

## 参考
- [INNER JOINとOUTER JOIN(LEFT JOIN, RIGHT JOIN)の違いについて](https://qiita.com/ysda/items/0d473e644409e45965d7)
  - INNER JOINでは、NULLとなるものがあればどちらのテーブルでも削除される
  - OUTER JOINでは、どちらかのテーブルが基準となるので基準となった方にNULLが含まれていてもそれは残る
