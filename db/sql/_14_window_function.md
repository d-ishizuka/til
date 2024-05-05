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
