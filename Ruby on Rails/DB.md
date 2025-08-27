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



### マイグレーション

------

```ruby
class SorceryCore < ActiveRecord::Migration[7.0]
  def change
    create_table :users do |t|
    # t.データ型 シンボル　　　　　　 データに対する制約条件
      t.string :email,            null: false, index: { unique: true }
      t.string :crypted_password
      t.string :salt
      t.string :first_name,       null: false
      t.string :last_name,        null: false

      t.timestamps                null: false
    end
  end
end
```

- null: false　：　データがnull であることを許容しない
- index:  { unique: true}　：　重複した値を許さない













