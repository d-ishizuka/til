## WHEREの基本文
- SQLではtrueとなる条件の場合1が返ってきて、falseとなった場合０が返ってくる

|演算子|意味|
|-|-|
|=|左辺と右辺が等しいか|
|<|左辺が右辺より小さいか|
|>|左辺が右辺より大きいか|
|<=|左辺が右辺以下か|
|>=|左辺が右辺以上か|
|<>, !=|左辺と右辺が等しくないか|

```
SELECT * FROM users WHERE id = 1;
SELECT * FROM users WHERE age > 20;
SELECT * FROM users WHERE name <> "Taro";
UPDATE users SET name = "foge" WHERE name <> "Taro";
DELETE FROM users WHERE id < 3;
```

## NULLについて
- DBでは真か偽以外に、不明(UNKNOWN)であることを表すNULLが存在する
- NULLは直接＝では取り出せない
  ```
  # nameの値が本当にNULLだったとしても、NULL=NULLは不明＝不明となり、真(True)にはならない
  SELECT * FROM USERS WHERE name = NULL;
  ```
- NULLのものを取り出すには、IS NULL, IS NOT NULLを用いる
  ```
  # NULLのレコードを取り出す
  SELECT * FROM users WHERE name IS NULL;

  # NULLでないレコードを取り出す(IS NOT NULL)
  SELECT * FROM users WHERE name IS NOT NULL;
  ```

## BETWEEN, NOT BETWEEN
```
# ageが10以上17以下のレコードをpeopleテーブルから取り出す
SELECT * FROM people WHERE age BETWEEN 10 AND 17;

# ageが10未満、または17より大きいレコードをpeopleテーブルから取り出す
SELECT * FROM people WHERE age NOT BETWEEN 10 AND 17;

# UPDATEとともに利用する
UPDATE users SET generation = "平成世代" WHERE age BETWEEN 5 AND 30;

# DELETEとともに利用する
DELETE FROM people WHERE age NOT BETWEEN 5 AND 30;
```

## LIKE, NOT LIKE
- 「%」:任意の０文字以上の文字列
- 「_」:任意の１文字

```
# nameがTで始まるレコードをpeopleテーブルから取り出す
SELECT * FROM people WHERE name LIKE "T%";

# nameがRで終わるレコードをpeopleテーブルから取り出す
SELECT * FROM people WHERE name LIKE "%R";

# nameにpが含まれるレコードをpeopleテーブルから取り出す
SELECT * FROM people WHERE name LIKE "%p%";

# nameがあ○のレコードをpeopleテーブルから取り出す(あご、あしなど)
SELECT * FROM products WHERE name LIKE "あ_";
```

## IN , NOT IN
```
式(カラム名) IN (値1, 値2, 値3 •••)
()内に列挙した複数の値のいずれかに合致したものを取り出す

# customersテーブルからcountry列がJapan, US, UKのいずれかに当てはまるものを取り出す
SELECT * FROM customers WHERE country IN ('Japan', 'US', 'UK');

# suppliersテーブルからcountry列を取り出す。countryに該当するものをcustomersテーブルから取り出す
SELECT * FROM customers WHERE country IN (SELECT country FROM suppliers);
```

## ANY
- 取得した値のリストと比較して、いずれかが真であるものを取り出す
```
# goodsテーブルからidが5よりも大きなレコードのpriceを取り出す。取り出したpriceのいずれかの値より小さいレコードをproductsテーブルから取り出す。(全部と比較して、どれか一個でもTrueが返ってこればTrueになる)
SELECT * FROM products WHERE price < ANY (SELECT price FROM goods WHERE id > 5);
```
## ALL
- 取得した値のリストと比較して、全てが真であるものを取り出す
```
# goodsテーブルからidが5よりも大きなレコードのpriceを取り出す。取り出したpriceの全ての値より小さいレコードをproductsテーブルから取り出す。(全部と比較して、いずれの値よりも小さい場合Trueになる)
SELECT * FROM products WHERE price < ANY (SELECT price FROM goods WHERE id > 5);
```

## AND
- 2つの条件の両方が真の場合だけ、真となる
```
SELECT * FROM customers WHERE age > 20 OR name LIKE "A%";
```

## OR
- 2つの条件のうち、どちらかが真の場合に真となる
```
SELECT * FROM customers WHERE age > 20 OR name LIKE "A%";
```

## AND + OR
```
SELECT * FROM employees WHERE salary > 5000 AND (id=1 OR id=5);
```

## NULLについての利用上の注意点
- NULLは直感とはズレた結果を返すことがある
```
# 全レコードを返しそうだがnameがNULLのデータは返らない
# nameがNULLのものは、NULLが変える
SELECT * FROM people WHERE name = "TARO" or name != TARO";
```
## AND, OR, NULLと真理値表
WHEREの条件として書いた場合

- ANDの場合
  |式|結果|
  |-|-|
  |NULL AND true| NULL|
  |true AND NULL| NULL|
  |NULL AND NULL| NULL|
  |NULL AND false| false|
  |false AND NULL | false|

- ORの場合
  |式|結果|
  |-|-|
  |NULL OR true| true|
  |true OR NULL| true|
  |NULL OR NULL| NULL|
  |NULL OR false| NULL|
  |false OR NULL| NULL|

### ANDはfalseが優先・ORはANDが優先(NULLは中間)
|AND|true|NULL|false|
|-|-|-|-|
|t|t|N|f|
|N|N|N|f|
|f|f|f|f|

|OR|t|NULL|f|
|-|-|-|-|
|true|t|t|t|
|NULL|t|N|N|
|false|t|N|f|

### ALL利用で注意する例
```
SELECT * FROM users WHERE age < ALL(SELECT age FROM other_users)
```
この時、例えば、other_usersのageにNULLのデータが入っているとすると、あるユーザーの年齢と比較した時に
```
15 < ALL(17, 18, NULL)
```
になるが、```15 < NULL```はN```NULL```なので結果全体がNULLになる。

### IN, NOT IN利用で注意する例
```
# 以下の例はいずれも、想定した通りに動かない
SELECT * people WHERE name IN ("Taro", "Jiro", NULL)

SELECT * people WHERE name NOT IN ("Taro", "Jiro", NULL)
```