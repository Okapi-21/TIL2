### PHPの型宣言（Type Declaration）

#### 型宣言とは

　PHPでは、メソッドや関数の「引数」と「戻り値」に対して、どのような型のデータを受け取り・返すかを宣言することができる。これを **型宣言（Type Declaration）** と呼ぶ。PHP 7.0以降で利用可能な機能であり、Laravelのコードでは広く使われている。

#### 戻り値の型宣言

　メソッド名の右に `: 〇〇` と書かれている部分が **戻り値の型宣言（Return Type Declaration）** である。

```php
public function create(): View          // View（画面）を返す
public function store(...): RedirectResponse  // リダイレクトを返す
public function handle(): int           // 整数を返す
```

#### 型の種類

##### PHP基本型（PHPの言語仕様として用意されている型）

| 型 | 意味 | 使用例 |
|----|------|-------|
| `void` | 何も返さない | 削除処理、ログ出力など内部処理のみのメソッド |
| `int` | 整数 | コマンドの終了コード、カウント値 |
| `string` | 文字列 | ファイルパス、名前 |
| `bool` | true / false | 認可チェック（`authorize(): bool`）、成功/失敗判定 |
| `float` | 小数 | 金額、座標 |
| `array` | 配列 | バリデーションルール（`rules(): array`）、一覧データ |
| `?string` | 文字列 or null | 値が存在しない可能性がある場合 |

##### クラス型（Laravelや自分が定義したクラスを指定する型）

| 型 | 意味 | 使用例 |
|----|------|-------|
| `View` | Bladeテンプレートの画面 | ページ表示（`return view('...')`） |
| `RedirectResponse` | リダイレクト | フォーム送信後の画面遷移 |
| `StreamedResponse` | ストリーム | ファイルダウンロード |
| `Transfer` | 自分で定義したモデル | DB登録後に作成されたレコードを返す |
| `Collection` | Eloquentコレクション | 複数レコードの取得結果 |
| `BelongsTo` | リレーション | モデル間の関連定義 |

#### 具体例（OneSendプロジェクトより）

```php
// Viewを返す → 画面表示
public function create(): View
{
    return view('transfers.create');
}

// RedirectResponseを返す → フォーム送信後にリダイレクト
public function store(StoreTransferRequest $request): RedirectResponse
{
    // ... ファイル保存処理 ...
    return redirect()->route('dashboard');
}

// intを返す → コマンドの成功/失敗コード
public function handle(): int
{
    // ... 期限切れデータ削除 ...
    return self::SUCCESS;  // 0
}

// boolを返す → このリクエストを許可するか
public function authorize(): bool
{
    return true;
}

// arrayを返す → バリデーションルールの配列
public function rules(): array
{
    return [
        'files' => ['required_without:file', 'array', 'min:1'],
    ];
}

// voidを返す → 内部処理のみ、何も返さない
function ($transfers) use ($disk, &$deletedCount): void {
    foreach ($transfers as $transfer) {
        $transfer->delete();
        $deletedCount++;
    }
}
```

#### 引数の型宣言

戻り値だけでなく、引数にも型を指定できる。

```php
public function download(Request $request, string $token, int $fileId): StreamedResponse
//                       ↑ Requestオブジェクト ↑ 文字列     ↑ 整数
```

これにより、「$token には文字列が来る」「$fileId には整数が来る」ことが保証される。

#### なぜ型宣言を書くのか

##### 1. ミスの早期発見

```php
public function handle(): int
{
    // return を書き忘れると...
    // → PHPが「intを返すはずなのに何も返していない」とエラーを出す
}
```

型宣言がなければ、このミスは気づかれず、後で別の場所で追いにくいバグになる。

##### 2. コードの可読性

```php
public function store(...): RedirectResponse
```

メソッドの中身を読まなくても「処理後にリダイレクトする」ということが分かる。チーム開発で特に重要。

##### 3. IDEの補完が効く

型宣言があることで、IDEが戻り値のメソッドやプロパティを正確に補完してくれる。

#### 補足

- 型宣言は書かなくてもPHPは動作する。しかし、Laravelのコードでは書くのが標準的な作法になっている。
- `?` を型の前につけると null も許容される（例: `?string` は「文字列 or null」）
- PHP 8.0以降では `string|int` のように複数の型を許容する **Union型** も使える
