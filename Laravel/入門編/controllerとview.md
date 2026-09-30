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



















