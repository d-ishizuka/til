## Rackとは
- WebアプリケーションサーバとWebアプリケーションフレームワークの間をインターフェイスを共通化した仕様であり、実装となっているもの
  - 具体的には、**WebアプリケーションフレームワークであるRailsがWebアプリケーションサーバであるPumaやUniconなどとスムーズにやりとりが行えるようにするためのもの**
- Rackができた背景1 : 共通インターフェイス
  - Rackが登場する以前は、サーバとフレームワークが密結合しており、AというフレームワークはXというサーバだと動かないといったことがあった
  - Rackインターフェイスにより、サーバとフレームワークの組み合わせが柔軟に選択できるようになった
- Rackができた背景2 : デプロイ
  - 様々なフレームワークとサーバの組み合わせが選択可能となったが、組み合わせによってデプロイ方法が複雑になってしまう問題が発生した
  - RackはPythonのWSGIという規格をもとに提案されたもので、サーバとフレームワークの中間に位置することで、サーバ・フレームワーク間を疎結合にする役割がある
    - 各アプリケーションサーバやフレームワークはRackに対応することでそれぞれの組み合わせを変更することが容易となった
- 処理の挿入
  - RackにはRack層からフレームワークに到達するまでの間に処理を追加することができるRackミドルウェアというものがあり、独自の処理を差し込む場合や、よく使う機能についてはRackミドルウェアとしてライブラリ化されるメリットもある。

## railsコマンド
### ```bin/rails server```コマンド
- このコマンドが実行されると、```Rack::Server```オブジェクトを作成し、Webサーバーを起動する
  - Webアプリケーション → Rack → Webサーバ起動
```
Rails::Server.new.tap do |server|
  require APP_PATH                  # APP_PATHで指定された場所にあるファイル(Railsアプリケーションのコード)をロード
  Dir.chdir(Rails.application.root) # カレントディレクトリをRailsアプリケーションのルートディレクトリに変更
  server.start                      # サーバーを開始。これにより、RailsアプリケーションがWebサーバーとして起動し、リクエストを受け付ける準備が整う
end
```

## ミドルウェアスタック
### ミドルウェアスタックの調べ方
```
# ミドルウェアのスタックを調べる
bin/rails middleware

# 出力結果
use ActionDispatch::HostAuthorization
use Rack::Sendfile
use ActionDispatch::Static
use ActionDispatch::Executor
use ActionDispatch::ServerTiming
use ActiveSupport::Cache::Strategy::LocalCache::Middleware
use Rack::Runtime
use Rack::MethodOverride
use ActionDispatch::RequestId
use ActionDispatch::RemoteIp
use Sprockets::Rails::QuietAssets
use Rails::Rack::Logger
use ActionDispatch::ShowExceptions
use WebConsole::Middleware
use ActionDispatch::DebugExceptions
use ActionDispatch::ActionableExceptions
use ActionDispatch::Reloader
use ActionDispatch::Callbacks
use ActiveRecord::Migration::CheckPending
use ActionDispatch::Cookies
use ActionDispatch::Session::CookieStore
use ActionDispatch::Flash
use ActionDispatch::ContentSecurityPolicy::Middleware
use Rack::Head
use Rack::ConditionalGet
use Rack::ETag
use Rack::TempfileReaper
run MyApp::Application.routes
```

| ミドルウェアスタック名 | 役割 |
| ---- | ---- |
|ActionDispatch::HostAuthorization| リクエストの送信先ホストを明示的に許可することで、DNSリバインディング攻撃から保護します |
|Rack::Sendfile|X-Sendfile headerを設定します。config.action_dispatch.x_sendfile_headerオプション経由で設定を変更できます。|
|ActionDispatch::Static|静的ファイルの配信に使います。config.public_file_server.enabledをfalse`にするとオフになります。|
|Rack::Lock|env["rack.multithread"]をfalseに設定し、アプリケーションをMutexでラップします。|
|ActionDispatch::Executor|スレッドセーフのコードを開発中にリロードするときに使います。|
|ActionDispatch::ServerTiming|リクエストのパフォーマンスメトリクスを含む Server-Timingヘッダーを設定します。|
|ActiveSupport::Cache::Strategy::LocalCache::Middleware|メモリキャッシュで用います。このキャッシュはスレッドセーフではありません。|
|Rack::Runtime|X-Runtimeヘッダーを設定します。このヘッダーには、リクエストの処理にかかった時間が秒単位で表示されます。|
|Rack::MethodOverride|params[:_method]が設定されている場合に（HTTP）メソッドが上書きされるようになります。HTTPのPUTメソッド、DELETEメソッドを実現するためのミドルウェアです。|
|ActionDispatch::RequestId|レスポンスでX-Request-Idヘッダーを有効にしてActionDispatch::Request#request_idメソッドが使えるようにします。|
|ActionDispatch::RemoteIp|IPスプーフィング攻撃をチェックします。|
|Sprockets::Rails::QuietAssets|アセットリクエストでのログ書き出しを抑制します。|
|Rails::Rack::Logger|リクエストの処理が開始されたことをログに出力します。リクエストが完了すると、すべてのログをフラッシュします。|
|ActionDispatch::ShowExceptions|アプリケーションが返すすべての例外をrescueし、例外処理用アプリケーション （エンドユーザー向けに例外を整形するアプリケーション） を起動します。|
|ActionDispatch::DebugExceptions|例外をログに出力します。ローカルからのリクエストの場合は、デバッグ用ページも表示します。|
|ActionDispatch::Reloader|development環境でコードの再読み込みを行うために、prepareコールバックとcleanupコールバックを提供します。|
|ActionDispatch::Callbacks|リクエストをディスパッチする直前および直後に実行されるコールバックを提供します。|
|ActiveRecord::Migration::CheckPending|未実行のマイグレーションがないか確認します。未実行のものがあった場合は、ActiveRecord::PendingMigrationErrorを発生させます。|
|ActionDispatch::Cookies|cookie機能を提供します。|
|ActionDispatch::Session::CookieStore| セッションをcookieに保存する役割を担当します.|
|ActionDispatch::Flash|flashのキーをセットアップします（訳注: flashは連続するリクエスト間でメッセージを共有表示する機能です）。これは、config.session_storeに値が設定されている場合にのみ有効です。|
|ActionDispatch::ContentSecurityPolicy::Middleware|Content-Security-Policyヘッダー設定用のDSLを提供します。|
|Rack::Head|すべてのHEADリクエストに対して空のbodyを返します。その他のリクエストは変更しません。|
|Rack::ConditionalGet|「条件付きGET（Conditional GET）」機能を提供します。"条件付き GET"が有効になっていると、リクエストされたページで変更が発生していない場合に空のbodyを返します。|
|Rack::ETag|bodyが文字列のみのレスポンスにETagヘッダを追加します。ETagはキャッシュのバリデーションに使われます。|
|Rack::TempfileReaper|マルチパートリクエストをバッファするのに使われる一時ファイルをクリーンアップします.****|

## ミドルウェアスタックの設定を変更する
- Railsが提供している ```config.middleware```を用いることで、ミドルウェアスタックにミドルウェアを追加・削除・変更可能
- 記述する場所は、```application.rb```か、環境ごとのファイルである```environments/<環境名>.rb```ファイルで記述する
  - 全体に適用したいか、環境ごとかで記述する位置を変える

### ミドルウェアの追加
| 書き方 | 意味 |
| ---- | ---- |
|config.middleware.use(new_middleware, args)|ミドルウェアスタックの末尾に新しいミドルウェアを追加します。|
|config.middleware.insert_before(existing_middleware, new_middleware, args)|新しいミドルウェアを、（第1引数で）指定された既存のミドルウェアの直前に追加します。|
|config.middleware.insert_after(existing_middleware, new_middleware, args)|新しいミドルウェアを、（第1引数で）指定された既存のミドルウェアの直後に追加します。|

```
# config/application.rb

# Rack::BounceFaviconを末尾に追加する
config.middleware.use Rack::BounceFavicon

# ActionDispatch::Executorの直後にLifo::Cacheを追加する
# Lifo::Cacheに引数{ page_cache: false }を渡す
config.middleware.insert_after ActionDispatch::Executor, Lifo::Cache, page_cache: false
```

### ミドルウェアを置き換える
| 書き方 | 意味 |
| ---- | ---- |
|config.middleware.swap|ミドルウェアスタック内にあるミドルウェアを置き換える|

```
# config/application.rb

# ActionDispatch::ShowExceptionsをLifo::ShowExceptionsで置き換える
config.middleware.swap ActionDispatch::ShowExceptions, Lifo::ShowExceptions
```

### ミドルウェアを移動する
| 書き方 | 意味 |
| ---- | ---- |
|config.middleware.move_before|指定したミドルウェアの前に移動する|
|config.middleware.move_after|指定したミドルウェアの後に移動する|

```
# config/application.rb

# ActionDispatch::ShowExceptionsをLifo::ShowExceptionsの前に移動
config.middleware.move_before Lifo::ShowExceptions, ActionDispatch::ShowExceptions

# ActionDispatch::ShowExceptionsをLifo::ShowExceptionsの後に移動
config.middleware.move_after Lifo::ShowExceptions, ActionDispatch::ShowExceptions
```

### ミドルウェアを削除する
| 書き方 | 意味 |
| ---- | ---- |
|config.middleware.delete|指定したミドルウェアを削除する|

```
# config/application.rb
config.middleware.delete Rack::Runtime
```

## ミドルウェアを自作する
- Rackインターフェイスのルールはシンプルで以下の規則にしたがっていれば、ミドルウェアとして追加できる
  - callというメソッドを定義する
  - callメソッドは慣習的にenvあるいはenvironmentと命名する引数を1つ受け取る
  - callメソッドは次の値を配列型で戻り値として返す必要がある
    - HTTPのステータスコードを表す値オブジェクト
    - HTTPヘッダーを表すハッシュオブジェクト
    - レスポンスボディとなる文字列を含んだ配列風オブジェクト
```疑似コード
class App
  defcall(env)
    status=200
    headers={"ContentType"=>"text/plain"}
    body=["sample"]
    [status,headers,body]
    end
  end
```

### Rubyの文字を大文字に変換するRackミドルウェア
```
# lib/middlewares/upcase_middleware.rb
class UpcaseMiddleware
  def initialize(app)
    @app = app
  end

  def call(env)
    status, headers, body = @app.call(env)
    body.each { |s| s.gsub!(/ruby/i, "RUBY")}
    [status,headers,body]
  end
end
```
```
# config/environments/development.rb
require "middleware/upcase_middleware"

Rails.application.configure do
  略
  config.middleware.use UpcaseMiddleware # ミドルウェアの末尾に挿入される
end
```

## 参照
- [railsガイド RailsとRack](https://railsguides.jp/rails_on_rack.html)
- パーフェクトRuby on Rails [増補改訂版]3-2 RackとRailsの関係