## 正確にデータを管理する
- DBはデータを正確に管理する必要がある
  - 停電して、DBが停止しても正確な値でなければならない(一部だけ更新されていてはいけない)
  - 間違ってレコードを消してしまっても、戻せなければならない

## トランザクションとは
- 複数のSQLをひとかたまりとして、扱うようにDBに指示する(DBの原子性を担保する)
  - 成功時：結果を反映する(コミット)
  - 失敗時：元に戻す(ロールバック)

## 原子性について
- トランザクションに含まれている複数のSQLが不可分のものであり、必ず「すべてが実行されている」か、「1つも実行されていない」状態に制御しなければいけないとするDBの性質
  - e.g. AさんがBさんに1万円振り込むためには、Aさんの口座から1万円差し引いた上で、Bさんの口座に1万円加算する

## トランザクションを開始するSQLコマンド
```
# トランザクションの開始
START TRANSACTION;

# トランザクション内の処理(複数のSQLはひとまとめにして実行させる)
INSERT ...
DELETE ...

# トランザクションが終了(実行結果が反映される)
COMMIT;
```

## ロールバックするSQLコマンド
```
# トランザクションの開始
START TRANSACTION;

# トランザクション内の処理(複数のSQLはひとまとめにして実行させる)
INSERT ...
DELETE ...

# トランザクションを破棄(明示的にロールバックを実行せずとも、DBとの接続が切断された場合もロールバックされる)
ROLLBACK;
```

## 自動コミットのON/OFF
- 多くのDBMSでは、明示的にトランザクションを開始しない限り、SQLを実行すると自動的にコミットされる。これを自動コミットモードという
- 自動コミットモードを解除するタイミングとしては、SQLの内容が正確に処理されるかを確認しながら行うために使われるが、その間DBにロックがかかるので注意！

```
# 自動コミットモードかどうか確認する
SHOW VARIABLES WHERE variable_name="autocommit";

# 自動コミットモードを解除する
SET AUTOCOMMIT = 0;
```