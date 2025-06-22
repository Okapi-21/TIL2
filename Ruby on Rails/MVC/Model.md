## Model

------

### マイグレーション

------

#### マイグレーションファイル適用

```
docker compose exec web rails db:migrate
```

#### 現在のマイグレーション確認

```
docker compose exec web rails db:migrate:status
```

#### 対象のマイグレーション down

```
docker compose exec web rails db:rollback STEP=1
```



### モデル生成

------

```
rails g model　[モデル名]　[属性名：データ型　属性名:データ型]　[オプション]
```



#### 中間テーブル作成

```
rails g model PostTag post:references tag:references
```

後ろ二つが外部キー

### Validation（データ制約）

------

```
lass User < ApplicationRecord
  authenticates_with_sorcery!

  validates :password, length: { minimum: 3 }, if: -> { new_record? || changes[:crypted_password] }
  validates :password, confirmation: true, if: -> { new_record? || changes[:crypted_password] }
  validates :password_confirmation, presence: true, if: -> { new_record? || changes[:crypted_password] }
  validates :first_name, presence: true, length: { maximum: 255 }
  validates :last_name, presence: true, length: { maximum: 255 }
  validates :email, presence: true, uniqueness: true
end
```



| 検証内容                                                     |          ヘルパーの使い方の例          |
| :----------------------------------------------------------- | :------------------------------------: |
| 必須のデータが入っているか                                   |     validates :foo, presence: true     |
| 数値を期待しているところに数値以外の値が入っていないか       |   validates :foo, numerically: true    |
| （数値の）範囲が適切か                                       |  validates :foo, inclusion: {in 0..9}  |
| 文字列の長さは適切か                                         | validates :foo, length: { maximum: 30} |
| データは一意か                                               |    validates :foo, uniqueness: true    |
| パスワードやメールアドレスがその確認用と内容が一致しているか |   validates :foo, confirmation: true   |



### 中間テーブル関連付け

------





### タグ生成

------

```ruby
# タグ保存・更新用メソッド
  def save_tags(tags_string)
    # 1. カンマ区切りで分割し、空白除去＆重複排除
    tag_names = tags_string.to_s.split(',').map(&:strip).reject(&:empty?).uniq

    # 2. タグを取得または作成
    tags = tag_names.map { |name| Tag.find_or_create_by(name: name) }

    # 3. 関連付けを更新（既存の関連を上書き）
    self.tags = tags
  end
```

