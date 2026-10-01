### Controller と View

#### コントローラーの役割

- リクエストの処理
  　コントローラーはHTTPリクエストで送信されたデータを処理する。
  これにはGETクエリパラメータ、POSTデータ、URLセグメントなどが含まれる。
- ミドルウェアの適用
  　コントローラーではミドルウェアを指定することで、認証、権限チェック、ログ記録などの共通処理を一元的に実施することができる。
  　これにより、特定のリクエストに対する処理のフィルタリングが可能になる。
- バリデーション
   　リクエストデータの検証をコントローラー内で実施することで、不正な入力を排除し、ユーザーに適切なエラーメッセージを返すことができる。Laravelでは、$request->validate() メソッドを使って、バリデーションルールに基づいた入力検証を簡単に行える。
- Fillを用いたデータ操作
  　コントローラーでは、モデルの属性にリクエストデータを一括で代入するために、Eloquentモデルの fill メソッドが利用される。

　以上の操作に加え、セッション操作、レスポンスの形式制御、ビューとデータの橋渡し、非同期処理の実装が可能になる

#### コントローラーの命名規則

　コントローラーの命名規則は基本的に英語の「**単数系**」を使用する。
Laravelでの標準的な機能をそのまま利用することができるため、単数系で統一することが望ましい。

#### メソッドとアクション

　**メソッド**：クラス内に記述される関数全般を示す。 function xxxx() {...} の代わりに使うことで形式で記述され、
		　  public, protected, private といったアクセス修飾子をつけることができる。
		　  通常、HTTPリクエストに対応する処理は public メソッドとして実装される。

　**アクション**：ルーティングで指定され、実際にHTTPリクエストに応じた処理を実行するために公開されたメソッドのこと
　　　　　　　つまり、ルーティング設定でエンドポイントと紐付けられているメソッドが「アクション」と呼ばれ、
　　　　　　　これに対応するビューを返すなどして、ユーザーに適切なレスポンスを提供する

#### コントローラー生成時コマンド

```
docker compose exec app php artisan make:controller xxxxxController 
```



#### ビューの役割（重要なとこ）

##### コンポーネントの作成

　コンポーネントは、ビューを共通化するための仕組みである。
再利用可能なUI要素（例：カード、ボタン、ヘッダー）を作成し、複数のテンプレートで使いまわすことができる。

以下のコマンドでコンポーネントを作成する
```php
$ php artisan make:component Card
```

作成したコンポーネントはBladeテンプレート内で次のように呼び出す
```php
<x-card :title="$title" :content="$component" />
```

コンポーネントに渡されたデータは、テンプレート内でローカル変数として利用することができる。

##### レイアウト

　レイアウトは、Webアプリケーション全体で共通する構造（例：ヘッダー、フッター、ナビゲーションバー）を定義するテンプレートです。

　Laravelでは resources/views/layouts ディレクトリにレイアウトファイルを作成し、@yield や @section を使用してコンテンツを挿入して使用することで、各ページの固有のコンテンツがレイアウト内に挿入されます。レイアウトを使用することで、一貫性のあるデザインを維持しつつ、コードの重複を避け、効率的に開発を進めることができます。

##### ヘルパー

　ヘルパーとは、ビューテンプレート内でつかわれるコードをまとめて、再利用可能にする仕組みのことです。ビューに関連する処理や、表示のための関数、メソッドをまとめることができるので、同じようなコードを複数のビューファイルで何度も書く必要がなくなります。

```
・日付・文字列・数値のフォーマット
・画像・動画・スタイルシートなどへのHTMLリンク作成
・コンテンツのサニタイズ
・フォームの作成
・コンテンツのローカライズ
・配列やデータ操作
・etc
```



#### ddヘルパー関数

　Laravelに組み込まれているデバッグ用の関数で、コードの実行を停止し、指定したデータの内容を見やすい形式で出力するためのもの。以下の機能を持つ。

- データの内容を出力
  変数の値やオブジェクトの構造を確認するために使用できます。 
  var_dump() や print_r() の代わりに使うことで、視認性の高い出力を得られます。
- プログラムの実行を停止
  dd() を実行すると、指定したデータを出力した後にスクリプトの実行が停止します。
  これにより、特定の箇所でのデータ状態を簡単に確認できます。**実行後は必ず削除かコメントアウトすること**
- 配列やオブジェクトの詳細な構造を表示
  LaravelのコレクションやEloquentモデルのデータ構造を確認するのに便利です。
  ネストされたデータも整形して表示されるため、デバッグが容易になります。

　プログラム中の変数に間違った値が入っていないか、プログラムを通して、表面上は正しく動いているように見えても内部では間違った処理が行われていないか、配列やオブジェクトが正しく機能しているか、データが正しい形で入っているのか、を確認するために実行される関数といえます

例として以下のようなものがある

```php
public function handleProviderCallback(Request $request)
  {
      dd([
          'has_code' => $request->filled('code'),
          'has_state' => $request->filled('state'),
          'error' => $request->input('error'),
          'error_description' => $request->input('error_description'),
      ]);

      // 後続の認証処理
  }
```



#### ルート一覧表示

```
docker compose exec app php artisan route:list
```



#### Routes

Laravelのルート定義は基本的に「HTTPメソッド＋URL」に対して、実行する処理を割り当てることができるものである。

例えば、以下のような処理については

```php
Route::get('/users/{user}', [UserController::class, 'show']);
```

  - HTTPメソッド：GET
  - URL：/users/{user}
  - 実行先：UserController の show メソッド

といったことを定義している。



また、「ミドルウェア」を指定することで、一定要件を満たすものしか到達することができないページというふうなものも設定することが可能

例えば、特定の画面表示にログインを必須としたい場合は、以下のような記述になる。

```php 
Route::middleware('auth')->group(function () {
    Route::get('/profile', [ProfileController::class, 'edit'])->name('profile.edit');
    Route::patch('/profile', [ProfileController::class, 'update'])->name('profile.update');
    Route::delete('/profile', [ProfileController::class, 'destroy'])->name('profile.destroy');
});
```

また、xxx.blade.php において、routes のURLを表現する場合、基本的には name('aaa.bbb')という形で表現する形になる

```php+HTML
# ex)
<a href="{{ route('login.microsoft') }}" >{{ __('Microsoft') }}</a>
```



#### Request（リクエスト）

　Controllerのメソッドで引数として受け取る `$request` は、HTTPリクエスト全体を表すオブジェクトである。フォームから送信された値だけでなく、URL、HTTPメソッド、Cookie、IPアドレス、ログインユーザー情報など、あらゆる情報が含まれる。

##### 2つのRequestの使い分け

　Requestには **汎用クラス** と **カスタムクラス（FormRequest）** の2パターンがある。

**パターン1: `Illuminate\Http\Request`（汎用）**

特にバリデーション（入力チェック）が不要な場合に使う。

```php
use Illuminate\Http\Request;

public function download(Request $request, string $token, int $fileId): StreamedResponse
{
    // $request->ip() でアクセス元のIPアドレスを取得
    // $request->user() でログイン中のユーザーを取得
}
```

汎用Requestで使える代表的なメソッド：

| メソッド | 用途 |
|---------|------|
| `$request->input('name')` | 特定のフィールドを取得 |
| `$request->all()` | 全入力値を配列で取得 |
| `$request->only(['name', 'email'])` | 指定したものだけ取得 |
| `$request->hasFile('file')` | ファイルがアップロードされたか確認 |
| `$request->file('file')` | アップロードされたファイルを取得 |
| `$request->user()` | ログイン中のユーザーを取得 |
| `$request->ip()` | アクセス元IPアドレスを取得 |
| `$request->url()` | アクセスされたURLを取得 |
| `$request->method()` | HTTPメソッド（GET, POSTなど）を取得 |

**パターン2: カスタム FormRequest**

フォーム送信など、バリデーションが必要な場合に使う。`app/Http/Requests/` ディレクトリに自分でクラスを作成する。

```php
// app/Http/Requests/StoreTransferRequest.php
class StoreTransferRequest extends FormRequest
{
    // このリクエストを許可するかどうか
    public function authorize(): bool
    {
        return true;
    }

    // バリデーションルール
    public function rules(): array
    {
        return [
            'files' => ['required_without:file', 'array', 'min:1'],
            'files.*' => ['required', 'file'],
        ];
    }

    // エラーメッセージのカスタマイズ
    public function messages(): array
    {
        return [
            'files.required_without' => 'ファイルを選択してください',
        ];
    }
}
```

Controllerでは型宣言でカスタムRequestを指定する。Controllerに到達した時点で、バリデーションは完了済みになる。

```php
use App\Http\Requests\StoreTransferRequest;

public function store(StoreTransferRequest $request): RedirectResponse
{
    // ここに到達 = バリデーション成功済み
    $request->file('files');  // 安心して値を取り出せる
}
```

FormRequest生成コマンド：

```
php artisan make:request StoreTransferRequest
```

##### 使い分けの基準

| 状況 | 使うクラス |
|------|-----------|
| フォーム送信を受け取る（バリデーションが必要） | カスタム FormRequest |
| ダウンロード、API、情報取得のみ | `Illuminate\Http\Request`（汎用） |

##### FormRequestを使う理由

- **責任の分離**: Controller は「何をするか」、FormRequest は「何を受け取るか」
- **Controllerの簡潔化**: バリデーションロジックを外に出すことでControllerがスッキリする
- **再利用性**: 同じバリデーションルールを複数のControllerで使いまわせる

##### 継承の構造

```
StoreTransferRequest（自分で作成）
    ↓ extends
FormRequest（Laravel提供 / authorize, rules, messagesなどを持つ）
    ↓ extends
Request（Laravel提供 / input, file, user, ipなどを持つ）
    ↓ extends
SymfonyRequest（Symfony提供 / HTTPリクエストの基盤）
```

自分で作るFormRequestは、これらの親クラスが持つメソッドを全て使うことができる。

---

### ControllerからBladeへモデルが渡る仕組み

#### 最初に結論

Bladeが`Transfer`モデルを自動的に探しているわけではない。

Controllerが`view()`の第2引数で`Transfer`オブジェクトを渡しているため、Blade内で`$transfer`として利用できる。

```php
public function show(Transfer $transfer): View
{
    return view('transfers.show', [
        'transfer' => $transfer->load('files'),
    ]);
}
```

```blade
{{ $transfer->status }}
```

対応関係は次のとおり。

```text
Controllerの配列キー                 Bladeの変数名
'transfer' => $transfer        →     $transfer
'message'  => $message         →     $message
```

`view('transfers.show', ...)`の`transfers.show`は、次のBladeファイルを示す。

```text
resources/views/transfers/show.blade.php
```

一方、配列の値に渡せるものは`app/Models`配下のモデルだけではない。文字列、数値、配列、Collection、自作クラスのオブジェクトなど、PHPで扱える値を渡せる。

つまり、次の2つは別の話である。

- `resources/views`：Bladeファイルを置く標準ディレクトリ
- Bladeへ渡せるデータ：Controllerなどが`view()`へ渡したPHPの値

モデルをどのディレクトリへ置いたかによって、Bladeで利用できるかどうかが決まるわけではない。

#### リクエストから画面表示まで

OneSendのTransfer詳細画面は、概ね次の順番で処理される。

```text
ブラウザ
  ↓ GET /transfers/1
Route
  ↓ TransferController@showを選択
ルートモデルバインディング
  ↓ ID=1のTransferを取得
Controller
  ↓ filesを読み込み、view()へ渡す
Blade
  ↓ HTMLへ変換
ブラウザ
```

ルートの`{transfer}`とController引数の`Transfer $transfer`が対応しているため、Laravelが対象モデルを検索する。

```php
Route::get('/transfers/{transfer}', [TransferController::class, 'show']);

public function show(Transfer $transfer): View
```

この検索でモデルが見つからない場合は、Controllerの処理へ入る前に404になる。

### Bladeは「HTMLに埋め込めるPHP」

Bladeは独立したプログラミング言語ではなく、最終的にPHPへコンパイルされるテンプレートエンジンである。

例えば、次のBlade構文は、

```blade
{{ $transfer->status }}
```

概念的には次のようなPHPへ変換される。

```php
<?php echo e($transfer->status); ?>
```

`e()`はHTMLエスケープを行う。ユーザー入力に`<script>`などが含まれていても、そのままHTMLとして実行されるのを防ぐ。

主なBlade構文とPHPの対応は次のとおり。

| Blade | 意味 |
|---|---|
| `{{ $value }}` | 値をエスケープして表示する |
| `{!! $html !!}` | エスケープせず表示する。信頼できない値には使わない |
| `@if ... @elseif ... @else @endif` | PHPの条件分岐 |
| `@foreach ... @endforeach` | PHPの繰り返し |
| `@csrf` | CSRF対策用のhidden inputを生成する |
| `@method('PATCH')` | HTMLフォームのPOSTをLaravel上でPATCHとして扱う |
| `<x-primary-button>` | Bladeコンポーネントを呼び出す |

### OneSendの公開状態表示を読み解く

```blade
@if ($transfer->expires_at && $transfer->expires_at->lte(now()))
    <p>公開期限が終了しています。</p>
@elseif ($transfer->status === \App\Models\Transfer::STATUS_ACTIVE)
    {{-- 公開停止フォーム --}}
@else
    {{-- 再公開フォーム --}}
@endif
```

上から順番に判定する。

1. `expires_at`が存在し、現在時刻以前なら「期限切れ」を表示する。
2. 期限内で`status`が`active`なら「公開を停止」ボタンを表示する。
3. それ以外、つまり停止中なら「再公開」ボタンを表示する。

`Transfer::STATUS_ACTIVE`はモデルに定義された定数である。

```php
public const STATUS_ACTIVE = 'active';
```

文字列`'active'`を複数箇所へ直接書くより、タイプミスを減らし、意味を明確にできる。

### 属性とリレーションをBladeから参照する

```blade
{{ $transfer->status }}

@foreach ($transfer->files as $file)
    {{ $file->file_name }}
@endforeach
```

- `$transfer->status`：`transfers.status`カラムに対応する属性
- `$transfer->files`：`Transfer::files()`で定義したリレーションの取得結果
- `$file->file_name`：`transfer_files.file_name`カラムに対応する属性

EloquentモデルはPHPのマジックメソッドを利用して、DBカラムやリレーションをプロパティのように参照できるようにしている。

ただし、ViewでDB問い合わせを組み立てるのは避ける。Controllerで`load('files')`などを使って必要なデータを準備し、Bladeは表示の分岐に集中させる。

### Controllerで登場したメソッドと関数

#### `abort_unless()`

```php
abort_unless($transfer->user_id === auth()->id(), 403);
```

Laravelのグローバルヘルパー関数で、「条件がtrueでなければ処理を中断する」という意味。

```text
条件がtrue  → 次の処理へ進む
条件がfalse → 指定されたHTTPエラーを返す
```

この例では、Transferの所有者でない場合に403 Forbiddenを返す。

似た関数に`abort_if()`がある。

```php
abort_if($user->isBlocked(), 403);       // 条件がtrueなら中断
abort_unless($user->isOwner(), 403);     // 条件がfalseなら中断
```

#### `unavailableResponse()`

```php
private function unavailableResponse(string $message, int $status): Response
```

これはLaravel標準メソッドではなく、OneSendの`TransferController`内で定義した独自メソッドである。

```php
return $this->unavailableResponse('公開期限が終了しています。', 410);
```

`$this->`が付いているため、「現在のControllerオブジェクトが持つメソッドを呼ぶ」と読める。定義元が分からない場合は、まず同じクラス内をメソッド名で検索する。

#### `response()->view()`

```php
return response()->view(
    'transfers.unavailable',
    ['message' => $message],
    $status,
);
```

BladeをHTMLへ変換しつつ、HTTPステータスコードも指定してResponseを作る。

通常の`view()`は画面テンプレートを返す。`response()->view()`は「画面の内容に加えて403や410などのHTTP情報も明示したい」ときに使う。

#### `Storage::disk('local')`

```php
Storage::disk('local')->exists($file->file_path);
Storage::disk('local')->download($file->file_path, $file->file_name);
```

`Storage`はLaravelのFacadeで、ファイル保存先を同じAPIで扱うための窓口である。

`disk('local')`は`config/filesystems.php`の`local`設定を選択する。OneSendでは次の場所がルートになっている。

```text
storage/app/private
```

ファイルパスを直接組み立てず、Storageを介することで、将来S3などへ保存先を変えてもアプリ側の呼び出し方を揃えやすくなる。

### DBトランザクションとファイル保存の違い

```php
$transfer = DB::transaction(function () {
    // DBへの登録処理
});
```

トランザクションは、複数のDB操作を一つのまとまりとして扱う。

- クロージャが正常終了：commitして変更を確定する
- 例外が発生：rollbackしてDB変更を取り消す
- クロージャの`return`：`DB::transaction()`全体の戻り値になる

ただし、DBトランザクションが取り消せるのはDB操作だけである。Storageへ保存した実ファイルは自動では削除されない。

そのためOneSendのアップロード処理では、DB登録に失敗したとき、`catch`内で保存済みファイルを明示的に削除している。

### 未知のLaravelコードを読む手順

未知のメソッドを見つけたら、次の順番で調べる。

1. 左側の値を確認する。`$transfer->`、`Transfer::`、`Storage::`では探す場所が異なる。
2. 同じクラス内に独自メソッドがないか検索する。
3. 型宣言やIDE補完から、オブジェクトのクラスを確認する。
4. Laravel公式ドキュメントで引数、戻り値、例外、副作用を確認する。
5. 必要なら`vendor/laravel/framework/src`から実装を確認する。

#### 公式ドキュメント

- [Laravel 13.x Views](https://laravel.com/framework/docs/13.x/views)
- [Laravel 13.x Blade Templates](https://laravel.com/framework/docs/13.x/blade)
- [Laravel 13.x Controllers](https://laravel.com/framework/docs/13.x/controllers)
- [Laravel 13.x File Storage](https://laravel.com/framework/docs/13.x/filesystem)

