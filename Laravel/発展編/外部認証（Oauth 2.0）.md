## 外部認証（Oauth 2.0）



#### 外部認証とは

　ログイン機能の外部委託のこと。通常やDB上にユーザーの名前やパスワード、メールアドレスなどを登録して、アプリ自身で認証を行うが、外部（MicrosoftやGoogle）へ認証機能を委託することもできる。このことにより、アプリではユーザー認証フローの一部を省くことができるし、ユーザーも一つのアカウントで複数のサービスを利用することが可能になる。



#### **Laravel Socialite（ソーシャライト）**

　Laravel Socialiteとは、Laravel公式が提供している「外部サービスのログイン（Oauth認証）」を簡単に実装できる拡張パッケージ**のこと。ユーザー登録やログイン時に「メール」「パスワード」ではなく、他サイトのアカウントで認証したい際に難しいコードを書かずに実装してくれる。



1. #####  composer Socialiteのインストール

```bash
#breezeを使用した際はbreezeの初期設定後に
composer require laravel/socialite

#SocialiteProviders/microsoftのインストール
composer require socialiteproviders/microsoft
```



2. ##### 環境変数の設定

   API連携を行うための環境変数の設定を行います。
   ＊テナントIDはシングルテナント（組織内の人のみがアクセスできるようにしたい場合の）運用を行う際に設定する必要あり

   マルチテナント（組織内外に関わらず、アカウントを持っていればアクセス可能）運用を行いたい場合は不要

```.env
#クライアントID
MICROSOFT_CLIENT_ID=xxxxxxxxxxxx
#シークレットクライアント
MICROSOFT_CLIENT_SECRET=xxxxxxxxxxxx
#テナントID(socialiteでは必須で設定する必要)
MICROSOFT_TENANT_ID=xxxxxxxxxxxx
#リダイレクトURI
MICROSOFT_REDIRECT_URI=https://localhost:3000/login/認証サービス名/callback
```

同様に、config/service.phpにおいても以下のように記述する

```php
    'microsoft' => [
        'client_id' => env('MICROSOFT_CLIENT_ID'),
        'client_secret' => env('MICROSOFT_CLIENT_SECRET'),
        'redirect' => env('MICROSOFT_REDIRECT_URI'),
        'tenant' => env('MICROSOFT_TENANT_ID'),
    ],
```

#### point

------

service.phpとは外部サービスの「設定情報」をまとめて管理するためのファイルのこと。

具体的には「メール送受信サービス」「Slack通知」「Microsoftログイン」など、Laravelアプリが外部サービスと連携する時に必要なIDや秘密鍵などの情報をここに載せる。

.envに本当の値（ID、パスワード）を書いておき、ここで呼び出すことで、プログラム本体に機密情報を書かなくても良くなり、安全で管理もしやすくなる。

------



3. ##### マイグレーションファイル・モデルの変更

  　認証方式の変更に伴って、マイグレーションファイルにも変更を施した。

追加ファイルが増えていくのが面倒だったので、一度、migrate:resetを行ってから以下のようにファイルを修正

```php
Schema::create('users', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('email')->unique();
  					$table->string('auht_provider, 20');
            $table->string('provider_id'); //provider_idを追加, PWとPW確認は使用しなくなったので消去(breezeを使用している場合、テストが通らなくなるので、nullableとして残しておいても良い)
            $table->integer('user_auth')->default(1); 
            $table->rememberToken()->nullable();　// ログイン状態を記録しておくために必要
            $table->timestamps();
  
  					$table->unique(['auth_provider', 'provider_id']); // auth_providerとprovider_idの組み合わせがユニークになるように設定
```

入力フォームでモデルの値を「一括代入」するので、モデルファイルにも変更を施す必要がある

```php
    protected $fillable = [
        'name',
        'email',
        'provider_id', // 'password' を廃止して、'provider_id'を代わりに入れる
      　'auth_provider' // google 認証も取り入れる場合は区別するためにこのカラムも必要になる
    ];
```



##### 4. コントローラー生成

以下のコマンドでコントローラーを生成する

```bash
php artisan make:controller Auth/MicrosoftController
```

コントローラーを以下のように記述
```php
<?php

namespace App\Http\Controllers\Auth; //名前空間（ディレクトリ）にあることを示す

use App\Http\Controllers\Controller; //親クラスとして使用するコントローラーをインポート
use Laravel\Socialite\Facades\Socialite;　//外部認証で各種外部サービス認証を操作するためのファサードを読み込む
use App\Models\User; //ユーザーモデルをインポート
use Illuminate\Support\Facades\Auth; //Laravelの認証のファサードを読み込む


class MicrosoftController extends Controller // MicrosoftController クラス定義の開始, controllerを継承
{
    public function redirectToProvider()　//Microsoftへリダイレクトするためのメソッド定義
    {
        // シングルテナント: tenantを付与（env/configで設定している場合）
        $tenant = config('services.microsoft.tenant_id'); //シングルテナントのためconfig/servicesよりテナントID取得
        $driver = Socialite::driver('microsoft');//Socialiteの「microsoft」ドライバ（Microsoft認証の仕組み）を取得
      
      //テナントIDが設定されていれば、認証リクエストに「tenantパラメータ」を付与する
        if ($tenant) {
            $driver = $driver->with(['tenant' => $tenant]);
        }
      //Microsoft認証画面へのリダイレクト用レスポンスを返す
        return $driver->redirect();
    }

    public function handleProviderCallback()//Microsoft認証後、コールバック時に呼ばれる処理
    {
      
      if ($request->input('error') === 'access_denied') {
            return redirect()
                ->route('login')
                ->withErrors(['oauth' => 'Microsoft認証がキャンセルされました。']);
       }
      
      
      try {
            // Microsoftから返された情報（ユーザー名・メール・IDなど）を取得する
            $microsoftUser = Socialite::driver('microsoft')->user();
        } catch (InvalidStateException) {
            return redirect()
                ->route('login')
                ->withErrors(['oauth' => '認証情報を確認できませんでした。もう一度お試しください。']);
        } catch (Exception $exception) {
            report($exception);

            return redirect()
                ->route('login')
                ->withErrors(['oauth' => 'Microsoft認証に失敗しました。もう一度お試しください。']);
        }
      	

        $id = $microsoftUser->getId();
        if (empty($id)) {
            abort(500, 'MicrosoftからIDが取得できませんでした');
        }

        // 外部サービス（Microsoft等）から「ログインしてきたユーザーの情報（メールアドレス・名前・ID）」を受け取り、
        // usersテーブルに「このprovider_idの人が既にいるかを探す）
        // いなければ新規作成 / いれば「名前」と「provider_id」を最新情報で上書き更新
        $user = User::updateOrCreate(
            ['auth_provider' => 'microsoft', 'provider_id' => $id],
            [
                'name' => $microsoftUser->getName(),
                'email' => $microsoftUser->getEmail(),
            ]
        );

        Auth::login($user, true); // これだとremember_tokenを更新しようとして失敗
        return redirect('/dashboard');
    }
}
```

#### Point 【handleProviderCallbackのここが大切】

------

以下の処理に着目する

```php
$user = User::updateOrCreate(
            ['auth_provider' => 'microsoft', 'provider_id' => $id],
            [
                'name' => $microsoftUser->getName(),
                'email' => $microsoftUser->getEmail(),
            ]
        );
```

もし、一方だけだと...

- 新規作成のみだと、同じメールアドレスで2回目以降の外部ログイン時にエラーになる（unique制約違反など）

  - 既に登録してるメールアドレスと登録しようとしているメールアドレスが重複してしまってエラーがでる

- 上書き更新だけだと、最初にログインした人しかusersテーブルに残らない

  - 上書き更新だけであると、一人のユーザーを何度も上書きすることになってしまう

- そのため「**updateOrCreate**」が両方を自動でやってくれる

  - > [!IMPORTANT]
    >
    > `updateOrCreate` メソッドが **自動で「存在チェック」→「update or create」までやってくれる** から「if (あるか？) { 更新 } else { 新規作成 }」の分岐を書く必要がない
    >
  
------



5. ##### Web.route

 routeに以下を追記する

```php
use App\Http\Controllers\Auth\MicrosoftController; // 認証のためのcontroller追記
use SocialiteProviders\Manager\SocialiteWasCalled; // 認証をやってくれる仕組みを追記

Route::get('login/microsoft', [MicrosoftController::class, 'redirectToProvider'])->name('login.microsoft');
Route::get('login/microsoft/callback', [MicrosoftController::class, 'handleProviderCallback']);
```



6. ##### イベントリスナー登録

LaravelのSocialite（公式）は、標準で「Google」「GitHub」「Facebook」「Twitter / X」などしか対応していません。したがって、**MicrosoftやLINEなど“拡張プロバイダ”を使いたい場合**は「SocialiteProviders」という追加パッケージを使います
SocialiteProvidersを使うことで、Socialite::driver('microsoft') のような「ドライバ」を**Laravelに後付けで登録する**必要があります。


したがって、

```bash
touch app/Providers/EventServiceProvider.php
```

を行い、下記のように記述する

```php
<?php

namespace App\Providers;

// 追記時にコメントアウト（Laravel13では仕様が変化している?）
// use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;
// use SocialiteProviders\Manager\SocialiteWasCalled;

use Illuminate\Support\Facades\Event;
use Illuminate\Support\ServiceProvider;
use SocialiteProviders\Manager\SocialiteWasCalled;
use SocialiteProviders\Microsoft\Provider;


class EventServiceProvider extends ServiceProvider //Laravelのイベント機能を拡張・管理するための特別なクラス
{
    protected $listen = [ // $listenという「イベントとリスナー（処理）の対応表」を定義
      										// 配列の形で「このイベントが発生したら、このリスナーを実行してね」とLaravelに伝えます。
        // 他のイベント...
        SocialiteWasCalled::class => [
            \SocialiteProviders\Microsoft\MicrosoftExtendSocialite::class.'@handle',
          	
            // ここに他のプロバイダも並べてOK
        ],
    ];

    public function boot()
    {
        parent::boot();
        // ここに直接リスナー登録を書くこともできるが、上記の配列に書くのが推奨
    }
}
```



##### 7. login.blade.php

以下のようにログイン画面にマイクロソフトの認証窓口を追加

```php+HTML
 <a class="underline text-sm text-gray-600 hover:text-gray-900 rounded-md focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500" href="{{ route('login.microsoft') }}">
                {{ __('Microsoft') }}
            </a>
```





