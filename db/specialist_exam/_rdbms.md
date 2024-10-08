# RDBMSに関する基本的な知識の整理

## 1. 表領域、データページ、ページ
- 表領域
  - ディスク上の物理的なデータ格納領域
    - システム表領域
    - 一時領域
    - ログ領域
    - ロールバック表領域
    - ユーザー表領域
      - ユーザーが作成するテーブルおよび索引(インデックス)
  - 例えば、HDD1-5があって
    - HDD1:システム表領域、ログ表領域、ロールバック表領域(100G)
    - HDD2:一時表領域、インデックス領域(100G)
    - HDD3-5:ユーザーのテーブルのデータ(800G)
- データページ（またはブロック）
  - RDBMSとディスク(表領域)との間でデータの入出力を行う単位
    - 1回で読み取られるデータの塊
  - テーブル・索引のデータが格納される。
  - 表領域ごとにページサイズ(ブロックサイズ)と空き領域を指定する
    - ページサイズ=1データページの長さを意味していて、2KB,4KB, 8KB,16KBなどが一般的
  - 同じデータページの中に、違うテーブルのデータは入らない(テーブルで分離されている)

### 1-2. データ所要量の見積もり
- 見積もり行数：そのテーブルが何レコード(何行)くらいになりそうか
- １ページあたりに格納できるレコード数(行数)を計算する
  - 1ページあたりのレコード数(行数)=(1ページのサイズ - ヘッダー部分のサイズ)　✖️ (1 - 空き領域率) ÷ （1レコードあたりの平均サイズ ※平均行長） = (16000 - 100) × (1 - 0.3) ÷ 1080 = 10.35行
  - つまり、1ページあたり10行格納することができる
  - 必要なデータ領域は、見積もり行数 ÷ 1ページあたりの行数(レコード数) × 1ページあたりのサイズ = 10,000行 / 10行 × 16,000バイト　= 16,000,000バイト

### 1-3. バッファサイズ
- 別の言い方をすると、キャッシュの領域
- 物理的にDBにアクセスせず、バッファエリア(=キャッシュ)に問い合わせてヒットすると高速で結果を返すことができる
- バッファヒット率などで表現される
  - インデックスに対するバッファとテーブル上のデータに対するバッファがある
  - もしインデックスのバッファヒット率が100%でも、テーブルデータに対してのヒット率が0だと、インデックスごとにテーブルを1ページ読み出す必要が出る

## 2. テーブルに関する記述
- 制約
  - NOT NULL制約
  - 主キー制約
  - 参照制約
    - 参照先の親レコードがupdateやdeleteされた時に子レコードに対する操作を指定できる
      - NO ACTION
        - 例えば、親レコードが削除されても子レコードに何もしない
        - つまり、存在しない親レコードを参照する子レコードが存在することになり、参照制約に反する
        - Railsでいう、restrict_with_exceptionのように、参照されていたら削除を許さない
      - CASCADE
        - 親レコードを削除したときに、参照している子レコードも一緒に削除する
        - 但し、親レコードが消せても子レコードも消えるので注意
          - 例. 社員テーブルが参照する部署テーブルのレコードを消したら社員レコードまで消える
        - Railsでいう、dependent_destroy
      - ちなみに、この制約では外部キーとして存在するレコードのIDを入れるかNULLを許容するものであり、NOT NULLにする場合は、別途NOT NULL制約が必要
  - 検査制約
    - check制約のこと
    ```
    CREATE TABLE users (
      id INT PRIMARY KEY,
      name VARCHAR(50),
      age INT,
      CHECK (age >= 0)  -- 年齢は0以上
    );
    ```

## 3. 索引とオプティマイザの使用に関する記述
- 索引
  - ユニーク索引
    - 同じ値が存在しない索引・インデックス
  - 非ユニーク索引
    - 同じ値が存在する索引・インデックス
  - クラスタ索引
    - キーの値の順番とキーの値が指す行の物理的並びが一致する
  - 非クラスタ索引
    - キーの値の順番とキーの値が指す行の物理的並びが一致せず、ランダムである

### クラスタ索引に関するChatGPTの解説

1. クラスタ索引 (Clustered Index)
- 定義: クラスタ索引は、データベーステーブルの行が物理的にソートされる索引です。つまり、索引のキーの順番と実際のデータ行の物理的な並びが一致します。
- 例: 例えば、次のようなテーブルがあるとします。
学生テーブル（students）

|student_id|name|age|
|-|-|-|
|1|Alice|20|
|2|Bob|22|
|3|Carol|21|

クラスタ索引をstudent_idに設定した場合、テーブル内の行は常にstudent_idの順番で物理的に並べられます。例えば、新しい学生として4: Daveが追加された場合、student_id順に並び替えられます。

アクセスの効率: クラスタ索引では、データをキーに基づいて順序通りに物理的に並べるので、範囲クエリ（例えばstudent_id BETWEEN 1 AND 3）では非常に効率よくデータを取得できます。ディスクの連続した部分からデータを読み込むため、アクセスが高速です。

クエリ例
```sql
SELECT * FROM students WHERE student_id BETWEEN 1 AND 3;
```
このクエリでは、データが物理的にstudent_id順に並んでいるため、効率的に連続してデータを読み込めます。

2. 非クラスタ索引 (Non-clustered Index)
- 定義: 非クラスタ索引では、キーの順番と行の物理的な順番は一致しません。索引は別の場所に保存され、そこから対応するデータ行を探しに行きます。
- 例: 例えば、同じ学生テーブルにnameに基づく非クラスタ索引を作成した場合、索引自体はnameでソートされていますが、実際のデータ行はstudent_idに基づいて物理的に並んでいます。
学生テーブル（students）

|student_id|name|age|
|-|-|-|
|1|Alice|20|
|3|Carol|21|
|2|Bob|22|

nameに基づいた非クラスタ索引は次のようにソートされています：

|name|student_id (pointer)|
|-|-|
|Alice|1|
|Bob|2|
|Carol|3|

非クラスタ索引を使用すると、まず索引からnameに対応するstudent_idを見つけ、そのstudent_idを使って実際のデータ行を取得します。

アクセスの効率: 非クラスタ索引は、データの検索や特定の列に対するクエリでは便利ですが、テーブル全体のスキャンや範囲クエリではクラスタ索引ほど効率的ではありません。
クエリ例
```sql
SELECT * FROM students WHERE name = 'Bob';
```

このクエリでは、name列に基づく非クラスタ索引を使用し、まずBobに対応するstudent_id（2）を見つけ、それからデータ行を取得します。

クラスタ索引と非クラスタ索引の違い
クラスタ索引: データが索引順に物理的に並ぶ。範囲クエリや順序付きクエリで高速。
非クラスタ索引: データは索引順に物理的に並ばない。検索クエリで役立つが、範囲クエリでは少し遅くなる可能性がある。

### 3-2 アクセスパスとオプティマイザ
- アクセスパス
  - SQLを実行した時にデータベースから対象のデータを取得する手順のこと
    - 表探索(テーブルスキャン)
      - 全データページを探索する
    - 索引探索(インデックススキャン)
      - 検索条件に適した索引によって対象行を絞り込んだ上で、データページを探索する
  - 決定方法
    - 統計情報もとにRDBMSによって表探索、索引探索に決められる
    - この決定の仕方がオプティマイザの役割

## 4. テーブルの物理分割に関する記述
- 前提：パーティション化と物理分割と区分キー・区分化は同じこと
- 