## Active Record

### ・Rails ActiveRecordにおける `where` と `find_by` の違い

### 1. `where`

- **複数件のレコードを取得**できる
- 返り値は「ActiveRecord::Relation」（配列のようなオブジェクト）
- 条件に合うレコードが**0件でも空のRelationが返る**
- 例：
  ```ruby
  users = User.where(name: "Taro")
  # => 複数件のUserオブジェクト（該当しなければ空配列）
  ```

### 2. `find_by`

- **最初の1件だけ**を取得する（複数該当しても最初の1件だけ）
- 返り値は「モデルのインスタンス」か「nil」
- 条件に合うレコードがなければ**nilが返る**
- 例：
  ```ruby
  user = User.find_by(name: "Taro")
  # => 1件のUserオブジェクト（該当しなければnil）
  ```

### まとめ表

|         | 返り値                       | 件数    | 該当なし   |
| ------- | ---------------------------- | ------- | ---------- |
| where   | Relation（配列のようなもの） | 0件以上 | 空Relation |
| find_by | モデル or nil                | 最大1件 | nil        |



### joinの使い方

### 1. 複数のテーブルを複合して調べることが可能になる

　`joins`メソッドは、モデル同士の関連（アソシエーション）を使ったSQLのJOIN句を生成します。

単純な結合
```ruby
User.joins(:posts)
```

複数の関連を結合

```ruby
User.joins(:posts, :comments)
```

入れ子の関連を結合

```ruby
Author.joins(books: :publisher)
```

検索条件を組み合わせるために用いることも可能

```ruby
User.joins(:posts).where(posts: { published: true })
```



### selectの引数がシンボルと文字列で違う理由

### シンボルの場合
- 例：`select(:film_id, :title)`
- カラム名だけを指定する場合に使用
- Railsが自動的にSQLに変換し、安全に扱えます

### 文字列の場合
- 例：`select("category.category_id, category.name, sum(payment.amount) as revenue")`
- SQLの関数や集計（sum）、エイリアス（as）など、  
  生SQLの表現が必要な場合に使用
- 文字列で指定することで、SQLにそのまま渡されます



###  `group` の使い方

### 1. 基本の使い方

`group`はSQLのGROUP BYと同じで、指定したカラムごとにデータをグループ化し、集計できます。
ここでのグループ化というのは、複数のレコードの組み合わせのうち完全に内容が一致しているものを一つにまとめることを示す
＊だからsumやcount, avgと使用するんですね

#### 例1：ユーザーごとの投稿数を集計

```ruby
User.joins(:posts).group("users.id").select("users.id, count(posts.id) as post_count")
```
→ ユーザーIDごとに投稿数（post_count）を集計

#### 例2：カテゴリごとの売上を合計

```ruby
Category.joins(:products).group("categories.id").select("categories.id, sum(products.price) as total_price")
```
→ カテゴリIDごとに商品の価格合計（total_price）を集計

### 2. 複数カラムでグループ化

```ruby
Order.group(:customer_id, :status).count
```
→ 顧客ID・注文ステータスごとの件数を取得

### 3. 注意点

- `group`は集計関数（sum, count, avgなど）と組み合わせて使う
- `select`で集計結果のカラム名やエイリアス（as）を指定すると分かりやすい
- SQLのGROUP BYと同じ働き



### メソッドチェーンについて

　Active Recordでは、多くの場合、メソッドの順序を変えても同じ結果になるので、基本的には順番は考慮する必要はない。

実務でのベストプラクティスとしては、以下の順序で記述していくと良いとされている。

```
Model.joins(:associations)      # 1. テーブル結合
     .where(conditions)         # 2. 条件絞り込み
     .select(columns)           # 3. カラム選択
     .group(columns)            # 4. グループ化
     .having(conditions)        # 5. グループ条件
     .order(columns)            # 6. ソート
     .limit(number)             # 7. 件数制限
```



### HAVING句とWHERE句の違い（SQL）

### WHERE句
- **データを「グループ化（集計）」する前に使う**
- 1行ごとのデータが対象
- 指定した条件に合う行だけを選び出す

#### 例
```sql
SELECT * FROM users WHERE gender = 'female';
```
→ 性別が女性の行だけ選択

---

### HAVING句
- **データを「グループ化（集計）」した後に使う**
- グループごと（集計結果）が対象
- 集計後のグループに条件をつけて絞り込む

#### 例
```sql
SELECT user_id, SUM(payment) FROM orders GROUP BY user_id HAVING SUM(payment) >= 10000;
```
→ 合計金額が1万円以上のユーザーだけ表示

---

### まとめ表

| 項目       | WHERE句      | HAVING句                   |
| ---------- | ------------ | -------------------------- |
| タイミング | 集計前       | 集計後                     |
| 対象       | 1行ごと      | グループごと（集計結果）   |
| 目的       | 行を選び出す | 集計したグループを絞り込む |

1. WHERE句  
   データ（明細）→【WHEREで絞る】→グループ化（集計）→結果

2. HAVING句  
   データ（明細）→グループ化（集計）→【HAVINGで絞る】→結果

---