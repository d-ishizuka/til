# IF
- 条件式をチェックして、真の場合と偽の場合とで表示内容を変える
```
IF(条件式、真の場合の値、偽の場合の値)
```

```
SELECT IF(100<200, "真", ”偽”); # 真と表示
SELECT IF(100<200 AND 300<200, "真", ”偽”); # 偽と表示
SELECT IF(name LIKE "T%", "Tで始まる名前です", "Tで始まらない名前です") FROM users;
SELECT *, IF(birth_place="日本", "日本人", "その他") AS "国籍" FROM users; # SELECT * で取得できる内容に加えて、カラムを追加できる
```

# CASE
```
CASE式1

CASE 評価する列
  WHEN 値1 THEN 値1のときに返す値
  WHEN 値2 THEN 値2のときに返す値
  (ELSE デフォルト値)
END

SELECT
  CASE country
    WHEN "Japan" THEN "日本人" # countryがJapanの時の表示内容
    ELSE "外国人" # countryがJapan以外の時の表示内容
  END AS "国籍"
FROM
  users;
```

```
CASE式2(こちらの方が一般的)

CASE
  WHEN 評価式1 THEN 評価式1が真のときに返す値
  (WHEN 評価式2 THEN 評価式2が真のときに返す値)
  (ELSE デフォルト値)
END

# SELECT * の内容に加えて、birth_placeが日本なら日本人、フランスならフランス人、それ以外なら外国人と表示する(かつidが30以上)
SELECT
  *,
  CASE birth_place
  WHEN "日本" THEN "日本人"
  WHEN "France" THEN "フランス人"
  ELSE "外国人"
  END AS "国籍"
FROM
  users
WHERE id > 30;

# WHENの中に、条件式を書くことができる
SELECT
 name,
 birth_day,
 CASE
   WHEN DATE_FORMAT(birth_day, "%Y") % 4 = 0 AND DATE_FORMAT(birth_day, "%Y" % 100 != 0) THEN "うるう年です"
   ELSE "うるう年でない"
 END AS "うるう年かどうか"
FROM users;

# THENの中は、特定のカラムを指定したり演算することもできる
SELECT
  *,
  CASE
    WHEN student_id % 3 = 0 THEN test_score_1
    WHEN student_id % 3 = 1 THEN test_score_2
    WHEN student_id % 3 = 2 THEN test_score_3 * 100
  END AS score
FROM tests_score;
```

## ORDER BYでCASEを使用する
```
ORDER BY CASE ••• END ASC(DESC)
```

```
# ORDER BYで、四国のものとその他のものを順番に並び替える
SELECT
  CASE
  WHEN name IN ("香川県", "愛媛県", "徳島県", "高知県") THEN "四国"
  ELSE "その他"
  END AS "地域名"
FROM prefectures ORDER BY CASE
  WHEN name IN ("香川県", "愛媛県", "徳島県", "高知県") THEN "四国"
  ELSE "その他"
  END DESC

# ORDER BYで、四国のものとその他のものを順番に並び替える
UPDATE
  prefectures
  SET name=CASE WHEN name IN ("香川県", "愛媛県", "徳島県", "高知県") THEN "四国"
  ELSE "その他"
  END
WHERE name = "香川";
```

## CASEでNULLを扱う場合
```
# 正しく動かないケース
SELECT
  CASE country
    WHEN "Japan" THEN "日本人"
    WHEN NULL THEN "不明"
  END
FROM
  users;

# 正しくは以下のように記述する
SELECT
  CASE country
    WHEN country = "Japan" THEN "日本人"
    WHEN country is NULL THEN "不明"
  END
FROM
  users;
```