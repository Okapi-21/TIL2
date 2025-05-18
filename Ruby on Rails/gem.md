

## <u>Gem</u>

------

### 画像

------

### gem 'carrierwave'



### 非同期処理

------

### gem 'turbo stream'

Railsアプリケーションを高速かつインタラクティブにするためのgem。

```
gem 'turbo-rails', '1.1.1' 
```

turbo stream の基本アクション

Append　（指定した要素の内部に新しいコンテンツを追加）

```ruby
<%= turbo_stream.append "comments" do %>
  <%= render @comment %>
<% end %>
```

comments という ID を持つ要素の内部に、@comment の部分テンプレートを追加します。

これにより、新しいコメントがリストの最後に追加される。

Appendアクションは、新しい要素を既存のコンテンツの後に追加する際に使用します。

**Prepend （指定した要素の内部の最初に新しいコンテンツの追加）**

```ruby
<%= turbo_stream.prepend "comments" do %>
  <%= render @comment %>
<% end %>
```

このコードは、comments という ID を持つ要素の内部の最初に、@comment の部分テンプレートを追加します。これにより、新しいコメントがリストの先頭に追加されます。

Prepend アクションは、新しい要素を既存のコンテンツの前に追加する際に使用します。

**Replace （指定した要素の置き換え）**

```ruby
<%= turbo_stream.replace "comment_#{@comment.id}" do %>
  <%= render @comment %>
<% end %>
```

comments という ID を持つ要素の内部に、@comment の部分テンプレートを追加します。これにより、新しいコメントがリストの最後に追加されます。

**Update （指定した要素の内容の更新）**

```ruby
<%= turbo_stream.update "comment_likes_#{@comment.id}" do %>
  <%= @comment.likes_count %>
<% end %>
```

comment_likes_#{@comment.id} という ID を持つ要素の内容を、@comment.likes_count で更新します。これにより、いいね数がリアルタイムで更新される

**Remove（指定した要素の削除）**

```ruby
<%= turbo_stream.remove "comment_#{@comment.id}" %>
```

comment_#{@comment.id} という ID を持つ要素を削除

Remove アクションは、不要になった要素を DOM から取り除く際に使用

**Before （指定した要素の直前に新しいコンテンツを追加)**

```ruby
<%= turbo_stream.before "comment_#{@comment.id}" do %>
  <%= render @new_comment %>
<% end %>
```

comment_#{@comment.id} という ID を持つ要素の直前に、@new_comment の部分テンプレートを追加する。これにより、新しいコメントが指定した位置の前に追加される。

**After （指定した要素の直後に新しいコンテンツを追加 )**

```ruby
<%= turbo_stream.after "comment_#{@comment.id}" do %>
  <%= render @new_comment %>
<% end %>
```

comment_#{@comment.id} というIDを持つ要素の直後に、@new_comment の部分テンプレートを追加します。これにより、新しいコメントが指定した位置の後に追加される。



### ページネーション

------

### gem 'kaminari'

Ruby on rails でページネーション機能を簡単に実装するためのライブラリ。

大量のデータを効率的に分割し、ユーザーにページングされたビューを提供することができる。

```
gem 'kaminari', '1.2.2'
```



### gem 'bootstrap5-kaminari-views'

gem 'kaminari' と Bootstrap 5 を組み合わせてページネーションのスタイルを簡単に適用するためのgem。

```
gem 'bootstrap5-kaminari-views'
```



## 検索

------

### gem 'ransack'

Ruby on rails に高度な検索機能を簡単に追加するためのgem

ransack は「検索フォームの作成」「検索クエリの生成」「検索結果の表示」までを包括的にサポートする。

```
gem 'ransack', '3.2.1'
```



#### メソッド

##### result(distinct: true)　：　検索結果を重複のない、ユニーク（一意）なレコードに限定する。

```
 def index
    @q = Board.ransack(params[:q])
    @boards = @q.result(distinct: true).includes(:user).order(created_at: :desc).page(params[:page])
  end
```



### パスワード再設定

------

### gem 'letter_opener_web'

送信されるメールをwebブラウザで確認できるgem（https://github.com/fgrehm/letter_opener_web）

```ruby
group :development do
  gem 'letter_opener_web', '~> 3.0'
end
```

メール送信処理が行われると、メール内容がブラウザで表示され、実際にメールを送信されることなく内容を確認することができる。これにより、メールの内容やフォーマットを簡単に確認・修正することが可能となる。

### gem 'config'

Railsアプリケーションで設定管理を簡単に行うためのgem。環境ごとに異なる設定を一括管理することができる。

（https://github.com/rubyconfig/config）

```ruby
gem 'config'
```

それぞれの環境で異なる設定を行いたい場合に、非常に有効。 config/setting/xxx.yml で環境ごとの設定を変更することができる。

### Action 「Mailer」

Railsでメールを送信するためのフレームワーク。

メールのテンプレートや送信設定を簡単に行うことができ、HTML形式やテキスト形式のメールを送信することが可能

ActionMailerを使用することで、ユーザー認証 や パスワードリセット、通知メールなど、様々なメール機能をアプリケーションに実装することができる。メールのビューは app/views/xxxx_mailer.rb に保存され、HTMLとテキストのテンプレートを作成する。



### 新しい gem を追加した際に必要なコード

```
docker compose run web bundle install
docker compose run web rails g xxxxx:install
docker compose restart
docker compose exec web bin/dev
```





