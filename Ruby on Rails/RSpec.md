## RSpec

------

　RSpecはRubyプログラミング用のテストフレームワークのこと。rails と組み合わせて利用することが多く、Railsアプリケーションの自動テストを作成する際にとても便利。

Gem

```ruby
group :development, :test do
  gem 'rspec-rails'
  gem 'factory_bot_rails'
end
```

設定ファイルの作成

```docker compose exec web bundle exec rails g rspec:install
docker compose exec web bundle exec rails g rspec:install
```

テストファイルの実行

```
docker compose exec web bundle exec rspec spec/models/task_spec.rb
```



