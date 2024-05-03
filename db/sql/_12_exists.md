## EXISTSとは
- 他のテーブルに値の存在する行のみ抽出するSQL
- サブクエリ内でメインクエりの表や列を利用する相関副問い合わせの1つ
```
SELECT
  *
FROM
  a_table
WHERE
  [NOT] EXISTS(subquery);
```

- EXISTSでは、EXISTSの後のサブクエリが何らかの値を返すレコードだけを取り出す。(NOT EXISTSの場合は、**値を返さないレコードだけを返す**)
```
SELECT
  *
FROM
  customers AS ct
WHERE
  EXISTS(
    SELECT
      * # とりあえず何でもいいからWHEREの条件がtrueになった時に返す値を指定(1でもいい)
    FROM
      orders AS or
    WHERE
      ct.customer_id = or.customer_id
  );
```
- INを使って書けるが、こちらの方がパフォーマンスが悪い
```
SELECT
 *
FROM
 employees as em
WHERE
  em.department_id in (SELECT id FROM departments);
```
## NULLとEXISTS
- NULLは、EXISTS内で使用する場合にも複雑な挙動をするので注意が必要
  - 特にNOT EXISTSの場合
- EXISTS句では、サブクエリで値が返されるレコードのみを取得する(NULLのものは返されない)
  - where null = nullとしてもtrueにならない
```
# 以下の二つのクエリは同じ挙動をする
SELECT * FROM b_table
WHERE EXISTS(
  SELECT * FROM a_table WHERE b_table.name = a_table.name
);

SELECT * FROM b_table
WHERE name IN (SELECT name FROM a_table);
```

- 逆に、NOT EXISTSの場合はNULLのものが返される（存在しなかったものとして認識される）
  - 但し、NOT INとは挙動が異なる点に注意
```
# 以下の二つは同じ挙動にはならない。なぜなら、NOT INで判定した時にNULLは判定不能・不明となり偽を返さないため

# NOT EXISTSは、EXISTSで返らなかったものを返す 
SELECT * FROM b_table
WHERE NOT EXISTS(
  SELECT * FROM a_table WHERE b_table.name = a_table.name
);

# NOT IN は、INで比較している各要素1つずつ比較していって、全て一致したものは除外される
# 但し、NULLはINで比較しても不明の結果しか返らないので、他の要素がfalseであればNOT INで返るが、他がtrueでもNULLの部分ではfalseにならないので返らないことがある
SELECT * FROM b_table
WHERE name NOT IN (SELECT name FROM a_table);
```

## EXISTSでINTERSECTとEXCEPTを実現する
```
# NOT EXISTS句を用いて各カラムを比較し、特定のテーブルから別のテーブルに存在するレコードを取り除いて取得する
SELECT * FROM b_table WHERE
NOT EXISTS(
  SELECT * FROM a_table
  WHERE
  a_table.column1 = b_table.column1 AND
  a_table.column2 = b_table.column2 AND...
);

# EXISTS句を用いて各カラムを比較して、特定のテーブルから別のテーブルに存在するレコードのみ取得する。この際、NULLを含むカラムには注意が必要
SELECT * FROM b_table WHERE
EXISTS(
  SELECT * FROM a_table WHERE
  WHERE
  a_table.column_1 = b_table.column_1 AND
  (a_table.column2 = b_table.comlun_2 OR (a_table.column_2 IS NULL AND b_table.column_2 IS NULL)) AND... # NULLが両方入っていたらそれはOKとする。(NULL=NULLが不明になるのでその対応)
)
```