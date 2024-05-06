```
1. employeesテーブルとcustomersテーブルの両方から、それぞれidが10より小さいレコードを取り出します。
両テーブルのfirst_name, last_name, ageカラムを取り出し、行方向に連結します。
連結の際は、重複を削除するようにしてください。
```
```
SELECT first_name, last_name, age FROM employees WHERE id < 10
UNION
SELECT first_name, last_name, age FROM customers WHERE id < 10;
```

```
2. departmentsテーブルのnameカラムが営業部の人の、月収の最大値、最小値、平均値、合計値を計算してください。
employeesテーブルのdepartment_idとdepartmentsテーブルのidが紐づけられ
salariesテーブルのemployee_idとemployeesテーブルのidが紐づけられます。
月収はsalariesテーブルのpaymentカラムに格納されています
```
```
SELECT
  MAX(sa.payment) AS 月収の最大値, MIN(sa.payment) AS 月収の最小値, AVG(sa.payment) AS 月収の平均値, SUM(sa.payment) AS 月収の合計値
FROM
  employees AS em
INNER JOIN
  departments AS dp
ON
  em.department_id = dp.id
INNER JOIN
  salaries AS sa
ON
  em.id = sa.employee_id
WHERE
  dp.name = "営業部"
;
```
```
3. classesテーブルのidが、5よりも小さいレコードとそれ以外のレコードを履修している生徒の数を計算してください。
classesテーブルのidとenrollmentsテーブルのclass_id、enrollmentsテーブルのstudent_idとstudents.idが紐づく
classesにはクラス名が格納されていて、studentsと多対多で結合される
```
```
SELECT
  SUM(
   CASE
     WHEN cl.id BETWEEN 1 AND 5 THEN 1
   END
  ) AS class1_5,
  SUM(
   CASE
     WHEN cl.id NOT BETWEEN 1 AND 5 THEN 1
   END
  ) AS class6_10
FROM
  classes AS cl
INNER JOIN
  enrollments AS en
ON
  cl.id = en.class_id
INNER JOIN
  students AS st
ON
  st.id = en.student_id
;
```

```
4. ageが40より小さい全従業員で月収の平均値が7,000,000よりも大きい人の、月収の合計値と平均値を計算してください。
employeesテーブルのidとsalariesテーブルのemployee_idが紐づけでき、salariesテーブルのpaymentに月収が格納されています
```
```
SELECT
  SUM(sa.payment) AS 月収の合計値,
  AVG(sa.payment) AS 月収の平均値
FROM
  employees AS em
INNER JOIN
  salaries AS sa
ON
  em.id = sa.employee_id
WHERE
  em.age < 40 and payment > 7000000
;
```

```
5. customer毎に、order_amountの合計値を計算してください。
customersテーブルとordersテーブルは、idカラムとcustomer_idカラムで紐づけができます
ordersテーブルのorder_amountの合計値を取得します。
SELECTの対象カラムに副問い合わせを用いて値を取得してください。
```
```
select customer_id, sum(order_amount) from orders group by customer_id;
```

```
6. customersテーブルからlast_nameに田がつくレコード、
ordersテーブルからorder_dateが2020-12-01以上のレコード、
storesテーブルからnameが山田商店のレコード同士を連結します
customersとorders, ordersとitems, itemsとstoresが紐づきます。
first_nameとlast_nameの値を連結(CONCAT)して集計(GROUP BY)し、そのレコード数をCOUNTしてください。
```
```
SELECT
  CONCAT(tmp_cs.first_name, tmp_cs.last_name),
  COUNT(*)
FROM
 (
 	SELECT * FROM customers WHERE last_name like "%田%"
 ) AS tmp_cs
INNER JOIN
  (
    SELECT * FROM orders WHERE order_date >= "2020-12-01"
  ) AS tmp_or
ON
  tmp_cs.id = tmp_or.customer_id
INNER JOIN
  items
ON
  tmp_or.item_id = items.id
INNER JOIN
  stores
ON
  stores.id = items.store_id
GROUP BY
  CONCAT(tmp_cs.first_name, tmp_cs.last_name)
;
```

```
7. salariesのpaymentが9,000,000よりも大きいものが存在するレコードを、employeesテーブルから取り出してください。
employeesテーブルとsalariesテーブルを紐づけます。
EXISTSとINとINNER JOIN、それぞれの方法で記載してください
```
```
#EXISTS
SELECT
  *
FROM
  employees AS em
WHERE
  EXISTS (SELECT DISTINCT employee_id FROM salaries AS sa WHERE payment > 9000000 AND sa.employee_id = em.id);

#IN
SELECT
  *
FROM
  employees AS em
WHERE
  em.id IN (SELECT DISTINCT employee_id FROM salaries WHERE payment > 9000000);

# INNER JOIN
SELECT
  *
FROM
(
  SELECT
     *
  FROM
   salaries
  WHERE
   payment > 9000000
) AS tmp_sa
INNER JOIN
  employees AS em
ON
  tmp_sa.employee_id = em.id
;
```


```
8. employeesテーブルから、salariesテーブルと紐づけのできないレコードを取り出してください。
EXISTSとINとLEFT JOIN、それぞれの方法で記載してください
```
```
# NOT EXISTS
SELECT
  *
FROM
  employees AS em
WHERE
  NOT EXISTS (SELECT DISTINCT employee_id FROM WHERE sa.employee_id = em.id);

# NOT IN
SELECT
  *
FROM
  employees AS em
WHERE
  em.id NOT IN (SELECT DISTINCT employee_id FROM salaries);

# LEFT JOIN
SELECT
  *
FROM
  employees AS em
LEFT JOIN
  salaries AS sa
ON
  em.id = sa.employee_id
WHERE
  sa.id IS NULL;
```

```
9. employeesテーブルとcustomersテーブルのage同士を比較します
customersテーブルの最小age, 平均age, 最大ageとemployeesテーブルのageを比較して、
employeesテーブルのageが、最小age未満のものは最小未満、最小age以上で平均age未満のものは平均未満、
平均age以上で最大age未満のものは最大未満、それ以外はその他と表示します
WITH句を用いて記述します
```
```
WITH tmp AS(
   SELECT
     MIN(age) AS min_age,
     AVG(age) AS avg_age,
     MAX(age) AS max_age
   FROM
     customers
)
SELECT
  emp.id,
  emp.age,
  (SELECT min_age FROM tmp) AS 最小年齢,
  (SELECT avg_age FROM tmp) AS 平均年齢,
  (SELECT max_age FROM tmp) AS 最大年齢,
  CASE
    WHEN emp.age < (SELECT min_age FROM tmp) THEN '最小未満'
    WHEN emp.age < (SELECT avg_age FROM tmp) THEN '平均未満'
    WHEN emp.age < (SELECT max_age FROM tmp)
 THEN '最大未満'
    ELSE 'その他'
  END
FROM employees AS emp;
```

```
10. customersテーブルからageが50よりも大きいレコードを取り出して、ordersテーブルと連結します。
customersテーブルのidに対して、ordersテーブルのorder_amount*order_priceのorder_date毎の合計値。
合計値の7日間平均値、合計値の15日平均値、合計値の30日平均値を計算します。
7日間平均、15日平均値、30日平均値が計算できない区間(対象よりも前の日付のデータが十分にない区間)は、空白を表示してください。
```
```
SELECT
  od.order_date AS 注文日,
  SUM(od.order_amount * od.order_price) AS payment,
  AVG(SUM(od.order_amount * od.order_price)) OVER(ORDER BY od.order_date ROWS BETWEEN 6  PRECEDING AND CURRENT ROW) AS avg_7,
  AVG(SUM(od.order_amount * od.order_price)) OVER(ORDER BY od.order_date ROWS BETWEEN 14  PRECEDING AND CURRENT ROW) AS avg_15,
  AVG(SUM(od.order_amount * od.order_price)) OVER(ORDER BY od.order_date ROWS BETWEEN 29  PRECEDING AND CURRENT ROW) AS avg_30
FROM(
  SELECT
    *
  FROM
    customers AS cs
  WHERE
    age > 50
  ) AS tmp_cs
INNER JOIN
  orders AS od
ON
  od.customer_id = tmp_cs.id
GROUP BY od.order_date;
```