## SQLの実行順序
- 実行順はSQLのチューニングを考える上でも重要
- WHERE句の中でウィンドウ関数が使えないのは、下記の処理順に反するから

|順番|処理|
|-|-|
|1|FROM, JOIN|
|2|WHERE|
|3|GROUP BY|
|4|集計関数(SUMなど)|
|5|HAVING|
|6|ウィンドウ関数|
|7|SELECT(レコードの取得)|
|8|DISTINCT|
|9|UNION/INTERSECT/EXCEPT|
|10|ORDER BY|
|11|OFFSET|
|12|LIMIT|