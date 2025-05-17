### route

------



#### 単数形リソース

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

