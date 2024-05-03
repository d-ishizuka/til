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
