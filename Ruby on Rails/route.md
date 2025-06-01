### route

------

#### ルート状況の確認

------

URLからのweb上で確認できる http://localhost:3000/rails/info/routes



#### ログイン・ログアウト

------

```ruby
get 'login', to: 'user_sessions#new' #ログインフォームを表示するためのもの
post 'login', to: 'user_sessions#create'　#ログインフォームに入力されたデータを送信するためのもの
　　　　　　　　　　　　　　　　　　　　　　　　　#ログイン情報はweb常に履歴が残ると危ないのでPOSTリクエストを使っている
delete 'logout', to: 'user_sessions#destroy' #ログアウト処理を呼び出すためのもの
```



#### 複数形リソース（resources）

------

resources メソッドは、Ruby on Railsのルーティングに広く使用され、特定のリソースに対して標準的なRESTfulルートを一括で生成する。resources :users と記述することで、ユーザーに関連する一連のルート（index, new, create, show, edit, update, destroy）が自動的に設定される。

```
resources :hoge, only: %i[new create] 
```

- 「only指定」について
  - onlyオプションを使用　→  resources メソッドで生成されるルートの中から特定のアクションだけを選択して生成
  - resources :user, only: [:index] と指定　→  ユーザー一覧のルートのみを生成し、その他のアクションのルートは除外
- 「ログイン・ログアウト」などのパスは resources で生成することができないので、手動で設定する必要があるよ

#### 単数形リソース（resource）

------

rails のルーティングでは、リソースは通常、複数形で定義されるが、単数形リソースもサポートされている。単数形リソースは特定のモデルに一意の関連付けがある場合に使用する。

単数形リソースを使用する場合、ルーティング設定ではresourceを使う。

```
resource :hoge
```

これにより、以下のようなルーティングが生成される。

| Helper         | HTTP verb | Path       | コントローラー#アクション |
| -------------- | --------- | ---------- | ------------------------- |
| new_hoge_path  | GET       | /hoge/new  | hoges#new                 |
| edit_hoge_path | GET       | /hoge/edit | hoges#edit                |
| hoge_path      | GET       | /hoge      | hoges#show                |
| hoge_path      | POST      | /hoge      | hoges#create              |
| hoge_path      | PATCH     | /hoge      | hoges#update              |
| hoge_path      | DELETE    | /hoge      | hoges#destroy             |


