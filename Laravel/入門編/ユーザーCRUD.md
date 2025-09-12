## user CRUD機能の実装

#### User CRUD機能を実装するためのコマンド

```
docker compose exec app php artisan make:model User -a 
```

 Php artisan make User -a コマンドは Laravel アプリケーションでの基本的なCRUD機能を持つリソースを迅速に生成するためのツールとして使用される。このコマンドを実行すると、以下のものが自動的に生成される。

- モデル
- マイグレーションファイル
- コントローラー
- ルーティング
- ファクトリ
- シーディングクラス

　＊生成されるコードがプロジェクト要件と完全に一意しないことが多いので利用されないことが多い
　　容易にバグを含んでしまう可能性を孕んでいる

#### ユーザーテーブルの準備

　先期ほどのコマンドでdatabase/migrations配下に新しいファイルが作成されている。railsのマイグレーションファイルと同様に、カラム名とデータ型を示していく必要がある。

```php
    public function up(): void
    {
        Schema::create('users', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->integer('age');
            $table->timestamps();
        });
```

#### マイグレーションコマンド

マイグレーションの実行

```
docker compose exec php artisan migrate
```

（おまけ）

DBの初期化

```
docker compose exec app php artisan migrate:reset
docker compose exec app php artisan migrate
```



#### モデルの設定

　Laravelではrailsとは異なり、モデルファイルにもテーブルと同様のカラムの設定追加が必要になる。
src/app/Models/User.phpを開き、編集する。

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    use HasFactory;

    protected $fillable = ['name', 'age',];
}
```

#### ルーティング

　src/routes/web.phpファイルを編集し、ルーティング処理を記述する。
Laravelのルーティングでは、URLに対して処理を行うためのコントローラーへの関連付けを行う。
``` Route::resource```を使うことで、「一覧画面」「詳細画面」「登録処理」「登録画面」「更新処理」「削除処理」の７つのルーティングをまとめて設定することができる。

```php
<?php

use Illuminate\Support\Facades\Route;

use App\Http\Controllers\TopController;
use App\Http\Controllers\UserController;

... 省略 ...

Route::get('/', function () {
    return view('welcome');
});

Route::get('/top', [TopController::class, 'index'])->name('top.index');
Route::resource('users', UserController::class);
```

