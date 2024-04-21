# MySQL
## 特徴
- デュアルライセンス
  - オープンライセンス
    - Community Edition(CE)
    - GPLv2
  - 商用ライセンス
    - GPLの制限なしに利用可能。Standard Edition(SE) / Enterprise Edition(EE)がある
  - ライセンス形態による差分
    - 独自機能の追加
    - サポートサービス
    - (EE) 運用監視ツール、バックアップツール、セキュリティ機能、スケーラビリティ、高可用性サポート
- ストレージエンジンアーキテクチャ
  - データファイルへアクセスする部分を設計上分離したアーキテクチャ
  - 内部システム情報の管理に利用するテーブルも含め、すべてInnoDBがデフォルトで採用されている
- 簡易で強力なレプリケーション機能
- 様々な種類のデータが扱える
  - 文字列、数値、日付、JSON、地理情報
- 豊富な情報源
  - マニュアルのURL構造
    - ```https://dev.mysql.com/doc/refman/8.0/en/```
      - vesion部分を変更したり、enをjaにすることで日本語のリファレンスも確認可能。但し、英語のリファレンスに比べて公開が遅いので、英語での参照が必要な場合もある。
    - [MySQL Reference Manual](https://dev.mysql.com/doc/)
    - [www.MySQL.com](https://www.mysql.com/)
    - [www.Oracle.com](www.Oracle.com)

## プロダクションレベル
- 開発途上版(機能変更の可能性あり) : -dmr(Development Milestone Release)
- リリース直前 : RC(Release Candidate)
- 製品版(安定版) : GA(General Availability)

## リリーススケジュール
- かつて不定期にリリースされていたが、2020年時点では年4回1, 4, 7, 10月にリリースが行われている

## インストール
- プラットフォームごとに選択
  - 各Linuxディストリビューション向けのパッケージやmacOS用のpkgインストーラなどがある
- macOS向けの場合
  - DMG Archive
    - インストーラがついていて、マウスクリックするだけでインストール完了する
    - インストール先は```/usr/local```に固定される(```/usr/local/mysql```)
    - 複数バージョンを共存させて、切り替えることはできない
  - Compressed TAR Archive
    - tarでアーカイブされていて、```/usr/local``` 以外の任意のディレクトリでも展開可能
    - 異なるディレクトリに複数のMySQLを展開し、共存させることができる
  - [ダウンロードページ](https://dev.mysql.com/downloads/mysql/)
    - CPUアーキテクチャによってもダウンロードするものが変わる
    - 例えば、M2Macなら(ARM, 64-bit)、Intel製なら(x86, 64-bit)
- PATHを通す
  - mysqlのコマンド類は```/usr/local/mysql/bin```ディレクトリに存在
  - 毎回フルパスで```/usr/local/mysql/bin/mysql```とか入力するのは面倒なので、まずは/usr/local/mysql/binディレクトリにパスを通す。
  - ```export PATH=$PATH:/usr/local/mysql/bin```
  - ```mysql -uroot -p```でpasswordを入力したらログインできる

### 参照
- MySQL徹底入門 第4版　MySQL 8.0対応　1-２章
- [インストール手順参考記事](https://it-jog.com/db/install-mysql-onmac)