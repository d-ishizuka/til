## リクエストがRailsのコントローラに届くまで

|メソッド| 処理内容|
|-|-|
|Rails::Engine#call||
|Rack::Proxy#call||
|Webpacker::DevServerProxy#perform_request||
|Rack::MiniProfiler#call||
|ActionDispatch::HostAuthorization#call|リクエストの送信先ホストを明示的に許可することで、DNSリバインディング攻撃から保護します|
|Rack::Sendfile#call|X-Sendfile headerを設定します。config.action_dispatch.x_sendfile_headerオプション経由で設定を変更できます。|
|ActionDispatch::Static#call|静的ファイルの配信に使います。config.public_file_server.enabledをfalse`にするとオフになります。|
|ActionDispatch::Executor#call|スレッドセーフのコードを開発中にリロードするときに使います。|
|ActiveSupport::Cache::Strategy::LocalCache::Middleware#call|メモリキャッシュで用います。このキャッシュはスレッドセーフではありません。|
|Rack::Runtime#call|X-Runtimeヘッダーを設定します。このヘッダーには、リクエストの処理にかかった時間が秒単位で表示されます。|
|Rack::MethodOverride#call|params[:_method]が設定されている場合に（HTTP）メソッドが上書きされるようになります。HTTPのPUTメソッド、DELETEメソッドを実現するためのミドルウェアです。PUTやDELETEの場合は、HTTP Methodは"POST"で、"_method"の中に実態が入っているので、実態にあったHTTP Methodをrequest.envに詰める|
|ActionDispatch::RequestId#call|レスポンスでX-Request-Idヘッダーを有効にしてActionDispatch::Request#request_idメソッドが使えるようにします。|
|ActionDispatch::RemoteIp#call|IPスプーフィング攻撃をチェックします。|
|Rails::Rack::Logger#call|リクエストの処理が開始されたことをログに出力します。リクエストが完了すると、すべてのログをフラッシュします。|
|ActionDispatch::ShowExceptions#call|アプリケーションが返すすべての例外をrescueし、例外処理用アプリケーション （エンドユーザー向けに例外を整形するアプリケーション） を起動します。|
|WebConsole::Middleware#call||
|ActionDispatch::DebugExceptions#call|例外をログに出力します。ローカルからのリクエストの場合は、デバッグ用ページも表示します。|
|ActionDispatch::ActionableExceptions#call||
|ActionDispatch::Executor#call||
|ActionDispatch::Callbacks#call|リクエストをディスパッチする直前および直後に実行されるコールバックを提供します。|
|ActiveRecord::Migration::CheckPending#call|未実行のマイグレーションがないか確認します。未実行のものがあった場合は、ActiveRecord::PendingMigrationErrorを発生させます。|
|ActionDispatch::Cookies#call|cookie機能を提供します。|
|Rack::Session::Abstract::Persisted#call||
|ActionDispatch::ContentSecurityPolicy::Middleware#call|Middleware Content-Security-Policyヘッダー設定用のDSLを提供します。|
|ActionDispatch::PermissionsPolicy::Middleware#call||
|Rack::Head#call|すべてのHEADリクエストに対して空のbodyを返します。その他のリクエストは変更しません。|
|Rack::ConditionalGet#call|条件付きGET（Conditional GET）」機能を提供します。"条件付き GET"が有効になっていると、リクエストされたページで変更が発生していない場合に空のbodyを返します。|
|Rack::ETag#call||
|Rack::TempfileReaper#call||
|ActionDispath:Routing::RouteSet#call|pathをNormalize|
|ActionDispatch::Journey::Router#serve|pathからroute(controller, action)を特定して処理を移譲する|
|ActionDispath:Routing::RouteSet::Dispatcher#serve|controllerへdispatch|
|ActionController::Metal.dispatch|bodyが文字列のみのレスポンスにETagヘッダを追加します。ETagはキャッシュのバリデーションに使われます。|
|ActionView::Rendering#process||
|AbstractController::Base#process|actionを呼び出し|
|ActiveRecord::Railties::ControllerRuntime#process_action|ActiveRecord::LogSubscriberをreset|
|ActionController::ParamsWrapper#process_action||
|ActionController::Instrumentation#process_action||
|ActionController::Rescue#process_action||
|ActionController::Rendering#process_action||
|AbstractController::Base#process_action||
|ActionController::BasicImplicitRender#send_action||

## 参照
- [RailsでリクエストがControllerにたどり着くまで](https://zenn.dev/kehra/articles/babf3ee58574eb)
- [Railsガイド Rails エンジン入門](https://railsguides.jp/engines.html)
