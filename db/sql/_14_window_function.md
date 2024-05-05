## ウィンドウ関数とは
- 一部の行に対して、集計した結果を表示するSQLのこと
- GROUP BYは列に対して集計を行うが、ウィンドウ関数は行に対して集計を行う

```
# ウィンドウ関数の書き方
WINDOW関数 OVER(
  [PARTITION BY 対象] -- 集計対象カラムを指定
  [ORDER BY 対象 [ASC | DESC]] -- 順番を並び替える
  [RANGE | ROW 集計範囲] -- 集計範囲を絞る
)
```
```
# OVERだけの場合
SELECT
  名前, 部署, 給料, AVG(給料) OVER() # オーバーがないと、GROUP BYがないのでエラーになる
FROM
  社員;
```
```
# PARTITION_BYを使った場合
SELECT
  名前, 部署, 給料, AVG(給料) OVER(PARTITION BY 部署) AS 部署ごとの平均給与 # 部署ごとにグループ化する
FROM
  社員;

# 年代毎の人数をカウントする
SELECT
  *,
  DISTINCT COUNT(*) OVER(PARTITION BY FLOOR(age/10)) AS age_count FROM employees;

# 月毎の売り上げを計算する
SELECT
  *,
  SUM(order_amount * order_price) OVER(PARTITION BY DATE_FORMAT(order_date, "%Y/%m"))
FROM
  orders;
```

## ORDER BYの利用
- パーティション...PARTITION BYによって、分割される行の塊のこと
- フレーム...パーティションの中で、集計の対処とするさらに小さな集合
  - UNBOUNDED PRECEDING...パーティションの始めの部分からフレームの対象にする
  - N PRECEDING...CURRENT ROWから見て、Nレコード分前をフレームの対象にする
  - CURRENT ROW...注目しているレコード
  - N FOLLOWING...CURRENT ROWから見て、Nレコード分後をフレームの対象にする
  - UNBOUNDED FOLLOWING...パーティションの終わりの部分までフレームの対象にする

```
# 指定した行で昇順か降順に順番を並び替えて、(デフォルトでは)パーティションの始めから現在の行と同じ値の行までをフレームとして気集計する
SELECT
  name, age, salary,
  SUM(salary) OVER(ORDER BY 年齢)
FROM
  employees;

Jiro,23,20000,50000
Yoshio,23,30000,50000 # 同じパーティションの始めから同じ年齢(23)の行までを足しているので、50000
Taro,24,10000,60000 #  同じパーティションの始めから同じ年齢(24)の行までを足しているので、60000
Saburo,25,20000,80000
Takashi,26,40000,120000
```

## PARTITION BY と　ORDER BYの両方を利用する
- PARTITION BYで分割してパーティション化する対象を決め、ORDER BYで行を並び替えて集計する
```
SELECT
  name, department, age, salary,
  SUM(salary) OVER(PARTITION BY department ORDER BY age) # 部署ごとに分割して、年齢で並び替える
FROM
  employees;
↓
Jiro,営業,23,20000,20000 # 同じ部署をパーティション(営業)として、ageでフレーム(~23)
Taro,営業,24,10000,30000 # 同じ部署をパーティション(営業)として、ageでフレーム(~24)
Yoshio,経理,23,30000,30000 # 同じ部署をパーティション(経理)として、ageでフレーム(~23)
Saburo,経理,25,20000,50000
Takashi,総務,26,40000,40000
Mikiko,総務,27,50000,90000
```

## フレームの範囲を変更する
- 現在の行を基準として、集計する対象の行(フレーム)を変更する
- ROWS BETWEEN 最初の集計対象行 最後の集計対象行
```
# 集計する対象の行を、直前の行と現在の行にする
SUM(数量) OVER(ROWS BETWEEN 1 PRECEDING AND CURRENT ROW)
```

- RANGE BETWEENについて
  - 現在の行の値を基準として、集計する対象の行(フレーム)を変更する(ORDER BYと一緒に使う)
  - ROWS BETWEENは行を基準としているが、RANGE BETWEENは値を基準にとする
```
# 集計対象を現在の年齢と、1つ下の年齢の値を含めて足す(例えば、24歳なら、23と24歳の合計)
SUM(salary) OVER(ORDER BY age RANGE BETWEEN 1 PRECEDING AND CURRENT ROW)

# 7日間の平均を求める
WITH daily_summary AS(
  SELECT
    order_date, SUM(order_price * order_amount) AS sale
  FROM
    orders
  GROUP BY
    order_date
)
SELECT
  *,
  AVG(sale) OVER(ORDER BY order_date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) # 6行前から今日まで
FROM
  daily_summary;
```

## ウィンドウ関数一覧
- ROW_NUMBER...パーティション内での行番号(重複はカウントアップする)
- RANK...パーティション内で重複する値は同じ値にして、重複分だけカウントアップしてランキング表示する
- DENSE_RANK...パーティション内で重複する値は同じ値にして、1だけカウントアップしてランキング表示する

|PRICE|ROW_NUMBER|RANK|DENSE_RANK|
|-|-|-|-|
|7|1|1|1|
|7|2|1|1|
|8.5|3|3|2|
|8.5|4|3|2|
|9|5|5|3|
|10|6|6|4|

- PERCENT_RANK...ORDER_BYと使う。現在の行のランクが全体の何%に当たるか(RANK-1)/(全行数-1)
- CUME_DIST...ORDER BYと使う。現在の行の値以下の行の数が、全体の何%に当たるか

|price|PERCENT_RANK OVER(ORDER BY price)|CUME_DIST OVER(ORDER BY price)|
|-|-|-|
|100|0|0.2|
|150|0.25|0.4|
|200|0.5|0.8|
|200|0.5|0.8|
|300|1|1|

```
SELECT
age,
RANK() OVER(ORDER BY age) AS row_rank,  -- RANK
COUNT(*) OVER() AS cnt, -- 行数
PERCENT_RANK() OVER(ORDER BY age) AS p_age, -- (RANK - 1)/(行数 - 1)
CUME_DIST() OVER(ORDER BY age) AS c_age -- 現在の行より小さい行の割合
FROM employees;
```

- LAG(expr, offset, default) ... **現在の行の、前の行の値を取得する**。expr(取り出す対象の行・式)、offset(何行前の値を取り出すか)、defalut(値を取り出せなかった場合の値)
- LEAD(expr, offset, default) ... **現在の行の、後の行の値を取得する**。expr(取り出す対象の行・式)、offset(何行後の値を取り出すか)、default(値を取り出せなかった場合の値)

```
元のテーブル
```
|number|sale|
|-|-|
|1|500|
|2|300|
|3|400|
|4|100|
|5|500|

```
LAG(sale) OVER(ORDER BY number)
```

|実行結果|
|-|
|NULL|
|500|
|300|
|400|
|100|

```
LAG(sale, 2, 0) OVER(ORDER BY number)
```

|実行結果|
|-|
|0|
|0|
|500|
|300|
|400|

``` 
LEAD(sale) OVER(ORDER BY number)
```

|実行結果|
|-|
|300|
|400|
|100|
|500|
|NULL|

```
LEAD(sale, 2, 0) OVER(ORDER BY number)
```

|実行結果|
|-|
|400|
|100|
|500|
|0|
|0|

- FIRST_VALUE...並び替えられたパーティション内の対象フレームの、一番最初の行の値を取り出す
- LAST_VALUE...並び替えられたパーティション内の対象フレームの、一番最後の行の値を取り出す

```
元のテーブル
```
|名前|部署|年齢|
|-|-|-|
|Taro|営業|24|
|Jiro|営業|23|
|Saburo|経理|25|
|Yoshio|経理|23|
|Takashi|総務|26|
|Mikio|総務|28|

```
FIRST_VALUE(名前) OVER(PARTITION BY 部署 ORDER BY 年齢)
```
|実行結果|
|Jiro|
|Jiro|
|Yoshio|
|Yoshio|
|Takashi|
|Takashi|

- NTILE(n)...対象の分布が、最初から数えて何番目にあたるのかを表示する。nで分布を幾つの数に分けるのかを設定する。
```
元のテーブル
```
|ID|VALUE|
|-|-|
|1|A|
|2|B|
|3|C|
|4|D|
|5|E|
|6|F|
|7|G|
```
NTILE(4) OVER(ORDER BY value)
```
```
|ID|VALUE|NTILE|
|-|-|-|
|1|A|1|
|2|B|1|
|3|C|2|
|4|D|2|
|5|E|3|
|6|F|3|
|7|G|4|
```

```
SELECT
  age,
  LAG(age) OVER(ORDER BY age), -- 直前
  LAG(age, 3, 0) OVER(ORDER BY age), -- 3つ前、ない場合は0
  LEAD(age) OVER(ORDER BY age), -- 直後
  LEAD(age, 2, 0) OVER(ORDER BY age) -- 2つ後、ない場合は0
FROM
  customers;
```