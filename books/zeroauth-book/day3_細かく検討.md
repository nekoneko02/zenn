# Day3

Day1, Day2を経て、基本形となる処理の流れが設計できてきました。今回は、セキュリティ観点で、問題を洗い出していきたいと思います。

## Day2までの図（再掲）

```plantuml
@startuml コンポーネント図
actor ユーザー as user
rectangle ブラウザ as browser
rectangle クライアント as client
rectangle "サービス" as service

user <--> browser: 画面表示/操作
browser <--> client: tokenのやり取り
browser <--> service: パスワードとtokenの交換
client <--> service: tokenでAPI連携する

@enduml
```

```plantuml
@startuml シーケンス図

actor ユーザー as user
participant ブラウザ as browser
participant クライアント as client
participant サービス as service

user -> browser: 操作
browser -> client: 1. フロー開始する
client --> browser: 2. token要求を送る（__リダイレクト__）
browser -> service: HTTPリクエストを送信する
service --> browser: ログイン画面を表示する
browser --> user: 画面を表示する
user -> browser: パスワードを入力する
browser -> service: 3. パスワードを送信する
service --> browser: 4. tokenを発行する（__リダイレクト__）
browser -> client: 5. tokenを送信する
client -> client: 6. tokenを保存する
loop API利用
    client -> service: token付きAPI呼び出し
    service --> client: レスポンス
end

@enduml
```

## セキュリティを考える

Day2までの設計を基に、セキュリティ観点の問題を考えていきます。

進め方としては、以下の順番に考えていきます：

- （攻撃者に）奪われたら困るものを洗い出す
- 「それらはどうすれば奪えるのか」を考える・・・①
- 「それらが奪われたらどうなるか」を考える・・・②
- ①を基に、奪われない方法を考える
- ②を基に、奪われた後に被害を減らす方法を考える

### 奪われたら困るもの

まず、zeroauthで登場するものは以下の通りです。

- サービスのパスワード
- token
- （サービス内に保存されているユーザーの）個人情報など

```plantuml
@startuml コンポーネント図
actor ユーザー as user
rectangle ブラウザ as browser
rectangle クライアント as client
rectangle "サービス" as service

user <--> browser: 画面表示/操作
note right of browser: token, パスワード, 個人情報など
browser <--> client: tokenのやり取り
note left of client: token, 個人情報（API連携されたもの）
browser <--> service: パスワードとtokenの交換
client <--> service: tokenでAPI連携する
note right of service: token, パスワード, 個人情報など

@enduml
```

token,パスワード,個人情報はどれも奪われると困ります。ただし、パスワードはスコープ外とします。詳細は次節の通りで、話を簡潔にするためです。

### サービスでの認証はスコープ外とする

Day2までの設計では、クライアントのtoken要求に対して、

- サービスはログイン画面を表示して、
- ユーザーはパスワードを使ってログインをする

と定義していました。しかし、実際にはログイン方法は何でもよく、パスワードといっても、ワンタイムパスワードでもいいですし、パスワード以外にも、パスキー、指紋認証などもあります。

zeroauthとしては、「サービスがユーザーを認証する」ことだけが必要で、その方法は何でもよいです。こうすることで、zeroauthとして、「パスワードをどう守るか？」を考える必要はなくなります（実際に使用する認証方法の脆弱性は考える必要はあるが、zeroauth側のフレームワークとしては考える必要がない）。

改めてシーケンス図を修正すると、以下の通りです。

```plantuml
@startuml シーケンス図

actor ユーザー as user
participant ブラウザ as browser
participant クライアント as client
participant サービス as service

user -> browser: 操作
browser -> client: 1. フロー開始する
client --> browser: 2. token要求を送る（リダイレクト）
browser -> service: HTTPリクエストを送信する
service --> browser: __認証画面__を表示する
browser --> user: 画面を表示する
user -> browser: __認証情報__を入力する
browser -> service: 3. __認証情報__を送信する
service --> browser: 4. tokenを発行する（リダイレクト）
browser -> client: 5. tokenを送信する
client -> client: 6. tokenを保存する
loop API利用
    client -> service: token付きAPI呼び出し
    service --> client: レスポンス
end

@enduml
```

## tokenと個人情報を守るには？

前節の議論により、考慮が必要なのはtokenと個人情報に絞られました。

次に、①「それらはどうすれば奪えるのか」と②「それらが奪われたらどうなるか」を考えていきます。