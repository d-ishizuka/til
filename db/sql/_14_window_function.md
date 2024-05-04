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
```
