# letter_opener_web

![Build Status](https://github.com/fgrehm/letter_opener_web/actions/workflows/main.yml/badge.svg)
[![Gem Version](https://badge.fury.io/rb/letter_opener_web.svg)](http://badge.fury.io/rb/letter_opener_web)
[![Code Climate](https://codeclimate.com/github/fgrehm/letter_opener_web.svg)](https://codeclimate.com/github/fgrehm/letter_opener_web)

[letter_opener](https://github.com/ryanb/letter_opener)で送信されたメールを
ウェブ上で簡単に閲覧できるインターフェースを提供するGemです。

## インストール方法

まず、開発環境用のGemとして追加し、`bundle`コマンドでインストールします。

```ruby
group :development do
  gem 'letter_opener_web', '~> 3.0'
end
```

## 使い方

`routes.rb`に以下を追加してください：

```ruby
Your::Application.routes.draw do
  mount LetterOpenerWeb::Engine, at: "/letter_opener" if Rails.env.development?
end
```

そして、アプリで[`letter_opener`配信方法](https://github.com/ryanb/letter_opener#rails-setup)
が設定されていることを確認してください。  
その後、メールを送信した後に`http://localhost:3000/letter_opener`にアクセスすると、
送信したメールをウェブで見ることができます。

もし[Vagrant](http://vagrantup.com)やDockerなどの仮想環境でアプリを動かしていて、
`letter_opener`の`launchy`による自動ブラウザ起動を無効にしたい場合は、
以下のようなエラーが出ることがあります：

```terminal
12:33:42 web.1  | Failure in opening /vagrant/tmp/letter_opener/1358825621_ba83a22/rich.html
with options {}: Unable to find a browser command. If this is unexpected, Please rerun with
environment variable LAUNCHY_DEBUG=true or the '-d' commandline option and file a bug at
https://github.com/copiousfreetime/launchy/issues/new
```

このような場合（もしくは単にウェブ上でメールを見たい場合）は、
`config/environments/development.rb`で配信方法を`letter_opener_web`に変更してください：

```ruby
config.action_mailer.delivery_method = :letter_opener_web
```

`letter_opener_web`を配信方法として使う場合は、
以下のようにしてメール保存先を変更することもできます（initializerやdevelopment.rbに追加）：

```ruby
LetterOpenerWeb.configure do |config|
  config.letters_location = Rails.root.join('your', 'new', 'path')
end
```

## ステージング環境などで使いたい場合

本番メール送信を防ぐ目的で、ステージングなどの事前検証環境でも使えます。
セットアップ手順は以下です：

1. `Gemfile`でGemをdevelopmentグループから出す
2. 対象の`config/environments/<env>.rb`で`config.action_mailer.delivery_method`を設定
3. `routes.rb`でその環境用にルートを有効化

例：`Gemfile`は

```ruby
gem 'letter_opener_web'
```

`routes.rb`は

```ruby
Your::Application.routes.draw do
  # 独自のconfig/environments/staging.rbがある場合
  mount LetterOpenerWeb::Engine, at: "/letter_opener" if Rails.env.staging?

  # ステージング環境でRAILS_ENV=productionを使う場合は本番で無効化する工夫が必要
  mount LetterOpenerWeb::Engine, at: "/letter_opener" unless ENV["PRODUCTION_FOR_REAL"]
end
```

**注意：HerokuでこのGemを使う場合、Dynoが1つだけで、バックグラウンドジョブでメールを送信していない場合のみ動作します。詳細や最新情報は[GH-35](https://github.com/fgrehm/letter_opener_web/issues/35)を参照してください。**

## 謝辞

[@alexrothenberg](https://github.com/alexrothenberg)さんの
[このプルリクエスト](https://github.com/ryanb/letter_opener/pull/12)のアイデアや、
[@pseudomuto](https://github.com/pseudomuto)さんの長期間のメンテナンスに感謝します。

## コントリビュート（貢献方法）

1. フォークして `bin/setup` を実行
2. 新しいブランチを作成（`git switch -c my-new-feature`）
3. 変更をコミット（`git commit -am 'Add some feature'`）
4. ブランチをプッシュ（`git push origin my-new-feature`）
5. プルリクエストを作成