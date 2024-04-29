```
1. customersテーブルから、ageが28以上40以下でなおかつ、nameの末尾が「子」の人だけに絞り込んでください。
そして、年齢で降順に並び替え、検索して先頭の5件の人のnameとageだけを表示してください。
```
```
SELECT name, age FROM customers WHERE (age >= 28 AND age <= 40) AND name LIKE "%子" ORDER BY age DESC LIMIT 5;
```

```
2. receiptsテーブルに、「customer_idが100」「 store_nameがStore X」「priceが10000」のレコードを挿入してください。
```
```
INSERT INTO receipts VALUES (301, 100, "Store X", 10000);
```

```
3. 2で挿入してレコードを削除してください
```
```
DELETE FROM receipts WHERE id = 301;
```

```
4. prefecturesテーブルから、nameが「空白もしくはNULL」のレコードを削除してください
```
```
DELETE FROM prefectures WHERE name = '' OR name is NULL;
```

```
5. customersテーブルのidが20以上50以下の人に対して、年齢を+1してレコードを更新してください

(ただし、BETWEENを使うこと)
```
```
UPDATE customers SET age = age + 1 WHERE age BETWEEN 20 AND 50;
```

```
6. studentsテーブルのclass_noが6の人すべてに対して、1～5のランダムな値でclass_noを更新してください
```
```
UPDATE students SET class_no = CEILING(RAND()*5) where class_no = 6;
```

```
7. class_noが3または4の人をstudentsテーブルから取り出します。取り出した人のheightに10を加算して、その加算した全値よりも、heightの値が小さくてclass_noが1の人をstudentsテーブルから取り出してください。

(ただし、IN, ALLを使うこと)
```
```
SELECT * FROM students 
WHERE height < ALL (SELECT height+10 WHERE class_no IN (3,4))
AND class_no = 1;
```

```
8. employeesテーブルのdepartmentカラムには、「 営業部 」のような形で部署名の前後に空白が入っています。この空白を除いた形にテーブルを更新してください
```
```
UPDATE employees SET department = TRIM(department);
```


```
9. employeesテーブルからsalaryが5000000以上の人のsalaryは0.9倍して、5000000未満の人のsalaryは1.1倍して下さい。

(ただし、小数点以下は四捨五入します)
```
```
UPDATE employees SET salary = 
CASE
  WHEN salary >= 5000000 THEN salary * 0.9
  WHEN salary < 5000000 THEN salary * 1.1
END;
```

```
10. customersテーブルにnameが「名無権兵衛」、ageが0、birth_dayが本日日付の人を挿入してください。

（ただし、日付関数を使うこと）
```
```
INSERT INTO customers VALUES (101, '名無権兵衛', 0, CURDATE());
```

```
11. customersテーブルに新たなカラムとして、「name_length INT」を作成します。

name_lengthカラムをcustomersテーブルの各行の名前の文字数でアップデートしてください
```
```
ALTER TABLE customers
ADD
  name_length INT;

UPDATE customers SET name_length = CHAR_LENGTH(name);
```

```
12. tests_scoreテーブルに新たなカラムとして、「score INT」を作成します。

scoreカラムに、testsテーブルの各行のtest_score_1, test_score_2, test_score_3から、取り出したNULLでない最初の値で更新します。

ただし取り出したtest_score_〇が、900以上の人は1.2倍して小数点以下を切り捨てて、600以下の人は0.8倍して小数点以下を切り上げてください。
```
```
ALTER TABLE tests_score
ADD
  score INT;

UPDATE tests_score SET score = COALESCE(test_score_1, test_score_2, test_score_3);
UPDATE tests_score SET score =
CASE
  WHEN score >= 900 THEN FLOOR(score * 1.2)
  WHEN score <= 600 THEN CEILING(score * 0.8)
  ELSE CEILING(score)
END;
```

```
13. employeesテーブルを、 departmentが、マーケティング部 、研究部、開発部、総務部、営業部、経理部の順になるように並び替えて表示してください。
```
```
SELECT * FROM employees
ORDER BY 
CASE department
  WHEN 'マーケティング部' THEN 0
  WHEN '研究部' THEN 1
  WHEN '開発部' THEN 2
  WHEN '総務部' THEN 3
  WHEN '営業部' THEN 4
  WHEN '経理部' THEN 5
END
```