## DB

------

### DB生成

```
docker compose exec web rails db:create
```

### DB初期化

```
rails db:drop
rails db:create
rails db:migrate
```



### カラム追加の方法

```
rails g migration Addカラム名Toテーブル名 カラム名:データ
```



### マイグレーションの実行

```
rails db:migrate
rails db:migrate:status
```

