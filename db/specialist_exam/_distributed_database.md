# 分散データベース
- ネットワーク上に複数存在するデータベースを1つのデータベースのように使用できるもの

## 分散データベース機能
- 透過性が重要
  - データベースが分散していても、それを利用者が意識しなくて済むということ
- 分散データベースの12のルール(の一部)
  - 存在場所からの独立...利用者は実際のデータの格納場所を意識する必要がない（移動に対する透明性・位置に対する透明性）
  - 分割からの独立...表を列や行で分割し、分散して格納されていても、一つの表として利用できる(分散に対する透明性・分割に対する透明性)
  - 複製からの独立...分散サイトで、データを重複して持っていても、利用者は意識する必要がない(重複に対する透明性・複製に対する透明性)

## 分散問い合わせ処理
- 複数のサーバをまたがって結合する必要があるデータを問い合わせる時、結合処理性能を向上させる方法が4つある。

### 入れ子ループ法(ネストループ法)
- 表A(アウターリレーション)のデータを1行ずつ取って、表B(インナーリレーション)と結合可能かどうか確認していく方法
- 最もシンプルだが、全組み合わせの処理になるので、最悪処理量が、表Aの行数 ✖️ 表Bの行数となる
  - 表B（インナーリレーション）側にインデックスがあれば全件の組み合わせ処理は不要

### 準結合法(セミジョイン法)
- 結合対象となる列を片方のサイトに送る
- 受け取ったデータに対して結合して、元のサイトに返す
- 元のサイトは受け取った後、さらに加工して最終結果とする
- 結合演算の最適化をしている

### ソートマージ法(マージ結合法)
- 2つのテーブルをそれぞれ結合対象となる列でソートし、ソート済みのテーブルをマージ処理しながら結合する
- 結合対象が多い時に有効だが、等結合に関してはハッシュ関数の方が高速となる。
- 結合対象となる列にインデックスをつけておくことで、高速化される。

### ハッシュ法
- 結合する列に対してハッシュ関数を用いて、ハッシュ値の値に応じて結合する
- 等結合しか使えないが、結合列が多い時にまとめてハッシュ化できるので事前ソートが不要で早い。

## 分散トランザクション
- **1つのトランザクションを複数のサイトにまたがって処理**する。
- この時、トランザクションの原子性が失われないように**コミットメント制御**を行う。
  - 要するに、1つのサイトで1つのDBでトランザクションしている場合はトランザクションで処理の失敗をハンドリングできるが、分散だとどうしようって話

## 1層コミットメント制御
- 1. 主となるサイトAが、従サイト(B, C,...)に対して、更新処理を投げる。
- 2. 従サイト(B, C,...)が更新処理を行い、主サイトAに応答する
- 3. 主サイトAが従サイト(B, C,...)にコミット要求を出し、コミットする。

## ２層コミットメント制御
- 1層コミットとの差は、更新処理OKの返事があればコミットの指示をしていたが、その前にもう一度コミットOKかどうかの確認を挟む点
- 1. 主となるサイトAが、従サイト(B, C,...)に対して、更新処理を投げる。
- 2. 従サイト(B, C,...)が更新処理を行い、主サイトAに応答する
- 3. **更新の応答後、それぞれにコミットOKかどうかを確認する**
- 4. コミットOKの返事が来たタイミングでコミットの指示を行い、コミットする。

## レプリケーション
- 分散されたデータベースの内容の一部、または全部を一定間隔で複写することをレプリケーションという
- 複数のサイトがデータのコピーを持ちユーザーやアプリケションを分散させることで負荷低減ができる
- 2相コミットメントを採用するとネットワークのトラフィックが増大するが、レプリケーションはこれを低減することができる

### 同期レプリケーション
- オリジナルデータの更新が複製側のDBにリアルタイムに行われる
- データの更新をトリガーにして、複製が行われる(イベント型)
- メリット
  - 時間によるデータの不整合が少なくなる
- デメリット
  - ネットワークへの負荷が増大する

### 非同期レプリケーション
- オリジナルと複製側で一定の間隔で同期が行われる(バッチ型)
- まず、更新ログを送信し、複製サイト側でログ反映を行う
  - スナップショットを送ることもある
- 多くの場合は、読み取り専用で用いられる
- メリット
  - ネットワーク負荷の低減
- デメリット
  - データ不整合が起こる可能性が高まる