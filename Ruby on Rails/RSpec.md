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

設定ファイルの作成（例）

```docker compose exec web bundle exec rails g rspec:install
docker compose exec web bundle exec rails g rspec:install
```

テストファイルの実行（例）

```
docker compose exec web bundle exec rspec spec/models/task_spec.rb
```



### テスト順序

------

1. テストケースの洗い出し	
   1. どんなことをテストする必要があるのかを洗い出す
      1. どんな処理を「失敗」とするのか
      2. どんな処理を「成功」とするのか
2. テストケースの書き出し
   1. describe ~ it までを一通り書き出す
3. テストコードの記載
   1. it 以下において、検証メソッドを使って、「成功」「失敗」の詳細を記す
4. factory botで作成したデータをテストコードに反映する



## model spec

------

アプリケーションのモデル（DBのテーブルやオブジェクトのクラスなど）の振る舞いを確認するためのテストのこと。具体的なテストでは、モデルのインスタンスを生成し、属性を設定して保存したり、バリデーションエラーが正しく発生するのか条件をテストしたりする。期待していた結果とテスト結果を照合して、テストが成功したかどうかを判断する。model の動作を確認するために非常に重要。



### テストのグループ化

```
describe "クラス名やメソッドの処理内容などを記載する" do
  context "状況や条件などを記載する" do
    it "テストケースの説明" do
      ...
    end
  end
end
```

- describe 
  テスト対象のクラスやメソッドなどのグループを定義する。テストコードでは複数のdescribeを作ることができ、階層構造を持たせることもできる。

- context
  特定の状況や条件に基づいてテストをグループ化するために使用される。describeの内部に記述される事が多い。

- it

  itブロックは、特定のテストケースを定義し、1つのitブロックは1つのテストケースを表す。



### 検証メソッド

------

値を検証するために**expect**メソッドを利用する。

- eq（２つの値が等しいかどうか）
  ```
  expect(テスト対象の値).to eq(期待する値)
  ```

- be（真偽値が等しいかどうか）
  ```
  expect(テスト対象の値).to be true # be true : 値がtrueであるかどうかをチェックします
  expect(テスト対象の値).to be false # be false : 値がfalseであるかどうかをチェックします
  expect(テスト対象の値).to be_nil # be_nil : 値がnilであるかどうかをチェックします
  ```

- be_empty（配列や文字列、ハッシュなどが空であるかどうか）
  ```
  expect(テスト対象の配列).to be_empty
  expect(テスト対象の文字列).to be_empty
  expect(テスト対象のハッシュ).to be_empty
  ```

- be_valid（バリデーションを通るのか）
  ```
  expect(テスト対象のオブジェクト).to be_valid
  expect(テスト対象のオブジェクトのエラーの配列).to be_empty
  ```

  be_emptyと組み合わせて、エラーメッセージの格納する配列が空であることも追加でテストとして記載するとより正確なテストを書くことができる。

- be_invalid （バリデーションを通らないのか）
  ```
  expect(object).to be_invalid
  ```



### factory bot のデータ作成

------

オブジェクトのファクトリ（工場）を定義するもの。一度ファクトリを定義すると、テストケース内でそのファクトリを使用してオブジェクトを簡単に繰り返し作成する事ができる。

```
FactoryBot.define do
  factory :user do
    name { "okapi" }
    sequence(:email) { |n| "okapi_#{n}@example.com" }
    age { 5 }
  end
end
```

**sequence**という連番を生成する機能がある

### factory botのデータの作成

------

- createメソッド
  ```
  user = FactoryBot.create(:user)
  or
  user = create(:user)
  ```

  データベースに実際にオブジェクトを保存するメソッド。データベースにレコードを作成してからそれを利用するようなテストケースで使用される。以下のような形で、データの書き換えも可能。
  ```
  user = FactoryBot.build(:user)
  or
  user = build(:user)
  ```

- buildメソッド
  ```
  user = FactoryBot.build(:user)
  or
  user = build(:user)
  ```

  データベースに「保存せずにオブジェクトを作成する」ためのメソッド。buildメソッドを使用すると、作成したオブジェクトはデータベースに保存されず、一時的なオブジェクトとしてのみ存在する。DBにデータを保存する必要がない場合や関連オブジェクトの作成など、一時的なオブジェクト作成のために作られる。createと同様にデータの上書きが可能。
  
- Factorybotの導入のために[rails_helper.rb ]へ以下を記入
  ```
  config.include FactoryBot::Syntax::Methods
  ```




## capybara

------

RSpecで統合テストを実行するためのDSL（ドメイン固有言語）のこと。主にWebアプリケーションのテストをシミュレートするために使用される。Capybaraはブラウザとのやり取りをシミュレートすることで、ユーザーが実際に行うであろう操作（クリック、フォーム入力、ページ遷移など）をテストコード内で再現できる。

#### capybaraのメソッド

------

##### visit （指定のURLやパスにアクセスする）

```
# /usersに対してアクセスする
visit '/users'

# Railsで定義されているroot_pathに対してアクセスする
visit root_path
```

##### click_on, click_link, click_button（対象のボタンをクリックする）

```
# 「ログイン」をクリックする
click_on 'ログイン'
```

##### fill_in（フォームの入力フィールドに値を入力する）

```
# 「ユーザー名」に対して、'らんてくん'を入力する
fill_in 'ユーザー名', with: 'らんてくん'
```

##### find（CSSセレクタで指定した要素を探す）

```
# classが「title」の要素を探す
find('.title')
```

##### select(セレクトボックスからオプションを選択する)

```
# 「言語」のセレクトボックスから'Ruby'を選択する
select 'Ruby', from: '言語'
```

##### have_content（指定した要素内にテキストが存在するか確認する）

```
# page全体のなかに'ようこそ'の文字列があるか確認する
expect(page).to have_content('ようこそ')
```



#### 処理の共通化

------

##### 変数宣言

##### let（テストケース内で使用する変数を定義する）

```
let(:user) { User.create(name: 'runteq', email: 'runteq@example.com') }
it 'userインスタンスが有効であること' do
  # ↓ここではじめて参照されるのでuserの変数が作られる
  expect(user).to be_valid
end
```

＊ let に似たものに let ! があるが、let ! は即座に変数を評価し、宣言が行われたのと同時に初期化する。 

##### 事前処理

##### before（各テストケースの実行前に共通のセットアップコードを実行する）

```
let(:user) { User.create(name: 'runteq', email: 'runteq@example.com') }
# ↓テスト前に処理を行う
before { login(user) }
```

##### 処理のmodule化（テストで行う処理のモジュール化）

```
module LoginMacros
  def login(user)
    visit login_path
    fill_in 'メールアドレス', with: user.email
    fill_in 'パスワード', with: user.password
    click_on 'ログイン'
  end
end
```

＊以下のように include することも可能

```
RSpec.describe 'hogehoge', type: :system do
  include LoginMacros

  it 'ログインした後に...' do
    user = User.create(email: 'runteq@example.com', password: 'password')
    login(user)
    # ログイン後のテストを書く
  end
end
```



### 個人的なポイント

------

1. テストコードはFactorybotの定義から始める
   　Factorybotを使ったテストを行う場合は、必ずFactorybotの定義が必要になるので、テストコードを書く前にFactorybotの定義から書き始める。でないと、テストコードで要らぬ設定が増えて、冗長化してしまう。テストをしていて、新しいモデル定義が必要になった時は、その都度、定義を更新していく。また、buildや createを使いたい場合は、FactoryBotのファクトリー定義が必須らしい...
2. テストで用いるオブジェクト名はそのテスト内容に沿ったものにする
   　例えば、titleのないtaskは「task_without_status」というふうに定義する。一目でどんなテストが弾かれたのかがわかるように。
3. 





