
## 2024_R04
- 不足エンティティを回答する問題で回答に時間がかかり、かつ間違えた
  - 本来の解き方
    - 問題文を読みながら、逐一概念データモデルとテーブル構造の対応を確認していくべきだった
    - そうでないと、後でまた全部読み返しながら対応エンティティを探すことになる
- 外部キー参照の表記に関する認識が甘かった
  - 概念データモデルで外部キー参照になっているリレーションがテーブル構造でどう表現されているか認識する
  - 例. テーブルA(__key1__, __key2__, ...), テーブルB(__key1__, __key2__, __key3__, ...)となっている場合、テーブルBはテーブルAを参照している。
  - あるテーブルでxxIDとかxx名などが主キーになっていて、それが他のテーブルにあるなら外部キーとしては気づきやすいが、複合キーがそのままマルっと他のテーブルの要素になっている場合は気付きにくい。(これに気づかないと同じようなリレーションの矢印が重なって引かれるようになって概念データモデルの図がおかしなことになる)
- ちゃんとエンティティ名の関係を理解しにいく
  - 例1. 等級というワードが出てきた時に、他にはどんな等級という名前のついたエンティティが存在しているのか？確認することで関係性が理解できる
  - 例2. 明細という言葉は、あるテーブルの詳細情報だから、基本的に外部キーとして参照していないとおかしい。
    - "運行スケジュール"と"運行スケジュール明細"の関係
  - 例3. 多対多の中間テーブルは、参照しているテーブルの名前を組み合わせていることがある。
    - "船型"と"等級"を組み合わせた"船型別等級構成"
- 受け身ではなく、なぜこのリレーションにしたのか自分で理解しながら読み進める
  - 多対多の解消をするために追加されているエンティティを理解する。たくさんのエンティティが登場すると、多対多の関係性がたくさん出てくる。
  - 例1. 複数の"航路"が存在して、複数の"港"が存在する。ある航路の中で港を一つ選ぶと、航路の中での港の発着時間などが決められた明細が決まる。つまり、"航路"と"港"には多対多の関係があり、その中間テーブルとして"航路明細"が作られている。**これを一瞬で分かりやすくしたのが、概念データモデル。**
- **多対多の中間テーブルはわかりやすい**
  - 先ほど例に挙げた、"航路"と"港"で決定される"航路明細"はエンティティ図を見ると[航路]→[航路明細]←[港]という形で、矢印の向き先が集まっている形になる。（同様に、[等級]→[船型別等級構成]←[船型]もなっている）
  - 文章で解説を読むと分量が多くてわかりにくくても、概念データモデルで「あー、このエンティティとエンティティは関係あるのか。その関係はここで定義されるのか。」がパッと見で分かる。
- 安易に1対多にしない。1対1の場合もあるので注意する。
  - わかりやすいのが、"予約客"と"乗船客"の関係で、予約した人が乗ったら乗船客になる。この場合は1対1なので、勢いで矢印を書かない。
- あとは、短い時間で要件を高いレベルで理解すること
  - ちゃんと話に入り込んで理解する
  - 結構理解してないと回答できないものも多い。頑張るしかない。
  - 今回初めて解いて時間配分結構ミスったけど、それでも頑張りゃなんとかなりそうだった。時間がかかるかどうかは理解度次第な印象。
- 関係スキーマとテーブルの違いを意識する
  - 今回の概念データモデル＋テーブルという出題は珍しいらしい(関係スキーマではなかった)
  - 概念データモデル上ではサブタイプで表現されているものも、今回のテーブルでは実装上区別しない → つまりスーパータイプだけしかテーブル上には存在しない。
    - 例. 予約あり乗船、予約なし乗船がサブタイプとして表現されていたが、テーブルにはスーパータイプの乗船しかない