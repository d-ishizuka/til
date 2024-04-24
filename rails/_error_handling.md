# エラーハンドリング
- Railsにおけるエラーハンドリングの手法は2つ
  - コントローラでrescue_fromメソッドを利用する
  - エラーハンドリング用のRack Middlewareを利用する

## rescue_fromを使ったエラーハンドリング
- ActiveRecord::RecordNotFoundまたはActionController::RoutingErrorが発生した場合はerror404をいうアクションを実行し、それ以外の時にはerror500というアクションを実行する。
- また、error500の時にはログにエラーとバックトレースを出力する
```
# app/controllers/application_controller.rb

class ApplicationController < ActionController::Base
  略

  rescue_from Execption, with: :error500
    rescue_from ActiveRecord::RecordNotFound, ActionController::RoutingError, with: :error404
  略

  def error404(e)
    render "error404", status: 404, formats: [:html] #リクエストがどんなフォーマットを要求してもHTMLのみ返す
  end

  def error500(e)
    logger.error [e, *e.backtrace].join("\n")
    render "error500", status: 500, formats: [:html]
  end
```
```
# app/views/application/error404.html.erb
ご指定になったページは存在しません
```
```
# app/views/application/error500.html.erb
エラーが発生しました
```

- このエラーハンドリングの場合、```config/routes.rb```に定義されていないURLがリクエストされた場合、エラーはRack Middlewareで発生するのでコントローラレベルで定義しているrescue_fromではキャッチできない。そのため、ルーティング設定の最後に全てのURLをキャッチする設定を追加して対応する。
```
# config.routes.rb
Rails.application.routes.draw do
  略

  # どのルーティングにも当てはまらない場合、error404を返すように設定する
  match "path" => "application#error404", via: :all
end
```

## Rack Middlewareを利用したエラーハンドリング
- rescue_fromを使ったエラーハンドリングは簡単だが、Rack Middlewareで起こったエラーは検知できない
- そのため、エラーハンドリングを行うRack Middlewareを利用することで解決する
- Rack Middlewareにおけるエラーキャッチの流れ
  - デフォルトで提供されている```ActionDispatch::ShowExceptions```がエラーを検知する
  - ```Rails.application.config.exceptions_app```として設定されたRackアプリケーションで処理が行われ、デフォルトでは、```ActionDispatch::PublicExceptions```のインスタンスが設定される。
  - ```ActionDispatch::PublicExceptions```は、例外クラスによって表示するHTMLを切り替える
    - e.g. ```ActionController::RoutingError```なら、public/404.htmlを、```ActionController::BadRequest```なら、public/400.htmlを表示しようとする
- アプリケーション内で定義した独自の例外が発生したときに、public/500.html以外を表示したい場合は、```config.action_dispatch.rescue_responses```を修正することで設定可能

```
# config/application.rb
略

module AwasomeEvents
  class Application < Rails::Application
    略
    config.action_dispatch.rescue_responses.merge!(
      "YourNewException" => :not_found
    )
  end
end
```

## 開発環境で本番環境と同じエラーページを表示させる方法
- Middlewareの中で、```ActionDispatch::ShowExceptions```の次に```ActionDispatch::DebugExceptions```が定義されていて、これが開発用のエラーページの表示を管理している。
- ```config.consider_all_requests_local=false```にすることで```ActionDispatch::DebugExceptions```は開発用のエラーページを表示せずに、再度エラーraiseを行って、結果として本番と同じエラーベー時が表示される

```
# config/environments/development.rb

Rails.application.configure do
略 
  config.consider_all_requests_local=false
略
end
```

## 参照
- パーフェクトRuby on Rails [増補改訂版]8-3 落穂ひろい