## CORSとは
- Cross Origin Resource Sharingの略
- 同一オリジンとは、スキーム(https)、ホスト(www.example.com)、ポート(443)がすべて同じである事

## step1 : CORSがなかった時代は同一オリジンのみ通信できた
- あるWebサーバー(https://wwww.example.com)からwebコンテンツを**ブラウザ**が読み込む → jsが動く → cookieの情報と一緒にajaxがWebサーバー(https://wwww.example.com)にリクエストを送る → 結果のjsonを受け取る(通信できた)
- あるWebサーバー(https://wwww.example.com)からwebコンテンツを**ブラウザ**が読み込む → jsが動く → cookieの情報と一緒にajaxが異なるオリジンのサーバー(https://api.example.com)にリクエストを送る → リクエストが拒否される(通信できなかったので不便)
- ポイント : **CORSを考える時はブラウザがどのWebサーバーのコンテンツを動かしていて、それに対してオリジンが同一か異なるかを考える**

## step2 : CORSとしてはこうしたい
- あるWebサーバー(https://wwww.example.com)からwebコンテンツを**ブラウザ**が読み込む → jsが動く → cookieの情報と一緒にajaxが異なるオリジンのサーバー(https://api.example.com)にリクエストを送る → 結果のjsonを受け取る(通信できた)
- また、リクエストにクッキーをつけて送受信したい

## step3 : 但し、上記のように単純に制限を緩めるとセキュリティ上問題が起こる（脆弱性）
- うっかり悪意のあるサイト(https://evil.example.org)にアクセスしてしまい、悪意のあるサイトのコンテンツを**ブラウザ**が読み込む。つまり、この時ブラウザのオリジンは悪意のあるオリジンになる。 → 悪意のあるjsが動く → 悪意のあるリクエストが他のオリジンであるAPIサーバー(https://api.example.com)にリクエストを送る → これが通ると、APIサーバー内で意図しない変更(csrf攻撃)が行われる　→ APIサーバーから返ってきた重要な情報を元の悪意あるサイト(https://evil.example.org)に転送することもできる(情報漏洩)

## CORSの保護戦略
- 悪意あるサイトを踏んで飛んでくるjsに対して防御するためには、APIサーバー側で保護戦略が必要
- 呼び出されるAPIサーバー側が、リクエストを送りつけてきたJavaScriptを信頼して良いかどうかを判断してあげることで、保護を行う。その判断基準がオリジンを基準とした判断となる。
- オリジンを信頼するかどうかは、サーバーがブラウザに対して、Access -Control-Allow-XXXヘッダを用いて伝える
- 方法
  - 1. リクエストを送る前にお伺いを立てる = プリフライトリクエスト (OPTIONSメソッド)
  - 2. 返ってきたレスポンスをJavaScriptに渡して良いことを許可する
- 許可と拒否の例
  - APIサーバー(https://api.example.com)を基準として、
    - 許可 : https://www.example.com（同一オリジン）
    - 拒否 : https://evil.example.org（異なるオリジン）

## preflightリクエストの内容とそのレスポンス
- ブラウザがjsリクエストの内容から、(ブラウザが)自動的にAPIサーバーに許可を求める
```
OPTIONS /api HTTP/1.1　 # OPTIONSメソッドを使う
Origin : htttps://www.example.com # ブラウザが開いているコンテンツ(jsが動いている)のオリジン情報が伝えられる
Access-Control-Request-Method : POST # Methodについて確認する
Access-Contorl-Request-Headers : content-type # リクエストのヘッダーに設定して良いか尋ねる
```

- APIサーバーからの許可状況をみて、ブラウザが後続のリクエストを送信する
```
HTTP /1.1 200 OK # 許可を与える場合2xxx
Access-Control-Allow-Origin : https://www.example.com # このオリジンに許可を与えます
Access-Control-Allow-Methods : POST, GET, OPTIONS　＃これらのメソッドは使っていいですよ
Access-Control-Allow-Headers : Content-Type # これらのリクエストヘッダは設定していいですよ
```

- 実際にリクエストを送った時のレスポンス
```
# レスポンスの内容をjsに渡して良いかを定めている
Access-Control-Allow-Origin : https://www.example.com
Access-Control-Allow-Credentials : true 
```

- Access-Control-Allow-Originヘッダ
  - オリジンを明示的に表示
    - Access-Control-Allow-Origin : https://www.example.com
  - ワイルドカード
    - Access-Control-Allow-Origin : *
    - 基本的に公開情報用APIに用いる

## クッキー付きでリクエストを送信する場合
- デフォルトでは、異なるオリジンに対してのリクエストにはCookieがつかない
- そのため、クッキー付きでリクエストを送るためには
```
var req = new XMLHttpRequest()
req.open('POST', 'https://api.example.com/api')
req.withCredentials = true # これが必要！！
```
- さらに、レスポンスを受け取るためには以下の条件を満たしていないといけない(2点を満たさないとJSでエラーになる)
  - Access-Control-Allow-Originヘッダは、オリジンを明示する必要があり、ワイルドカード指定は使えない
  - Access-Control-Allow-Credentials : true

## プリフライトに関する疑問
- 本当に必要か？
- というのも、プリフライトリクエストの主な役割はCRSF対策
- CRSF対策は古来から**アプリケーション側**の責任ではないのか？(トークンを使う)
- だとすると、二重に対策をしていることになるのでは？ → 従来対策されていなかった可能性がある攻撃も存在
  - DELETEメソッドは、POSTやUPDATEメソッドと違い、ウェブフォームでの指定がないメソッドなのでCORSがない時代(=同一ドメインしかリクエストが通らなかった時代)はCSRFの攻撃ルートがなかった → CRSF対策不要なケースだけプリフライトリクエストで保護しよう！つまり、以下のような単純リクエスト(=従来CSRF対策が必要だった条件)に関しては、プリフライトは不要で、従来通りアプリケーション側でカバーする。
  - 但し、プリフライトが飛ぶか飛ばないかは難しい時があるので、基本的には常にCRSF対策をWebアプリケーションで行うべき。

## 単純リクエスト
- メソッドは以下のいずれか
  - GET
  - POST
  - HEAD
- 設定できるリクエストヘッダは以下のいずれか(主要なもの)
  - Accept
  - Accept-Language
  - Content-Language
  - Content-Type
    - application/x-wwww-form-urlencoded
    - multipart/form-data
    - text/plain



## 参照
- [徳丸浩のウェブセキュリティ講座 今日こそ理解するCORS](https://www.youtube.com/watch?v=yBcnonX8Eak&t=8s)