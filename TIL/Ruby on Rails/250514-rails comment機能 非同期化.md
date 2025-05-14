# rails 基礎 16

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
