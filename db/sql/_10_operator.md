## 集合演算子(UNION, UNION ALL, EXCEPT, INTERSECT)
- 構造のよく似た複数のテーブルに対してSELECTでレコードを取得して、取得結果を組み合わせるSQL
  - UNION : 和集合...複数の検索結果を足し合わせる
    - UNIONは重複する行は1つにまとめ、UNION ALLは重複する行は重複したまま取り出す
    - 重複を弾く作業が入る分、UNIONの方が処理時間が長くなる(重複の排除が不要ならUNION ALLの方が良い)
  - EXCEPT(MINUS) : 差集合...検索結果のうち、重複するものを除く
    - SQL1とSQL2の結果を比較して、SQL1の結果のうちSQL2の結果に存在するものを差引く
  - INTERSECT...積集合 : 検索結果で重複したものを取り出す
    - SQL1とSQL2の結果を比較して、2つの結果に共通する行を表示する
- 書き方
  - 注意点
    - 各SQLで得られるデータのカラム数を合わせること
    - ORDER BYを使う場合は、1つ目のSQLのカラムを使うこと
```
# UNIONのSQL
SELECT * FROM table1
UNION
SELECT * FROM table2;

SELECT * FROM table1
UNION
SELECT * FROM table2
UNION
SELECT * FROM table3;

# カラムの数が一致していればUNIONすることができる(制約によってはエラーになる可能性ありそう)
SELECT id, name FROM students
UNION
SELECT name, name FROM new_students
ORDER BY id;
```

```
# EXCEPTで重複をしたデータを引く(Minusと考えた方が分かりやすい)
SELECT * FROM new_students
EXCEPT
SELECT * FROM students;
```

```
# INTERSECTで重複したデータを取り出す
SELECT * FROM new_students
INTERSECT
SELECT * FROM students;
```

## 集計関数(SUM, MIN, MAX, COUNT, AVG)
|関数名|数値型|文字列型|日付型|
|-|-|-|-|
|SUM|合計値|×(実行できない)|×|
|MIN|最小値|並び替えて最初の文字列|最も古い日付|
|MAX|最大値|並び替えて最後の文字列|最も新しい日付|
|AVG|平均値|×|×|
|COUNT|行数|行数|行数|

### 集計関数の書き方
```
# SUM
SELECT SUM(number) FROM table_name
# AVG
SELECT AVG(number) FROM table_name # NULLの場合は分子・分母から除れる
SELECT AVG(COALESCE(number, 0)) FROM table_name; # NULLの場合は0として平均に組み込みたい場合
# MIN, MAX
SELECT MIN(number), MAX(number) FROM table_name
# COUNT
SELECT COUNT(*) FROM table_name # 列指定でCOUNTする場合、NULLの行はカウントされない
```

## グループに分ける（GROUP BY）
- 例えば、employeesテーブルから、部署ごとに平均給与を計算する
```
SELECT department, AVG(salary) FROM employees GROUP BY department;
```

```
# column1でグループ化して集計する
SELECT column1, SUM(column2), AVG(column2) FROM table_name GROUP BY column1;

# column1とcolumn2でグループ化して集計する
SELECT column1, column2, SUM(column3), AVG(column3) FROM table_name GROUP BY column1, column2

# WHEREで絞り込んでから、グループ化する
SELECT column1, MIN(number) FROM table_name WHERE column1 < "xx" GROUP BY column1;

# 集計結果をORDER BYで並び替える
SELECT column1, COUNT(*) FROM table_name GROUP_BY column1, ORDER BY COUNT(*);
```

- GROUP BYでCASE文を利用する1
```
# GROUP BY内にCASE文を記述する
SELECT
CASE
  WHEN name IN ("香川県", "高知県", "愛媛県", "徳島県") THEN "四国"
　ELSE "その他"
END AS "district",
count(*)
GROUP BY
CASE
  WHEN name IN ("香川県", "高知県", "愛媛県", "徳島県") THEN "四国"
　ELSE "その他"
END;
```

- GROUP BYでCASE文を利用する2
```
select
case
    when birth_place="日本" THEN "日本人"
    else "その他"
  END AS "国籍",
count(*)
from
users
group BY
  case
    when birth_place="日本" THEN "日本人"
    else "その他"
  END;
```

## 集計結果の絞り込み(Having)
- グループ化して集計した結果に対して、絞り込みをする場合に用いるSQL
  - WHERE : GROUP BYをする前に絞り込む
  - HAVING : GROUP BYをした後に絞り込む 
```
HAVINGの書き方
# SUM(salary)が10000よりも大きい場合
SELECT department, SUM(salary), AVG(salary) FROM table_name GROUP BY department HAVING SUM(salary) > 10000;

# WHEREで絞り込んでから、グループ化して、HAVINGで絞り込む
SELECT column1, MIN(number) FROM table_name WHERE column1 < "xx" GROUP BY column1 HAVING MIN(number) < 10000

# 集計結果をORDER BYで並び替える
SELECT column1, COUNT(*) FROM table_name GROUP_BY column1 HAVING COUNT(*) < 100 ORDER BY COUNT(*);

# GROUP BYがなくてもHAVINGを使えるケース
select "重複なし" from students having count(id) = count(distinct id); # whereではない
```