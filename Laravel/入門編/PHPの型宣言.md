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
public function download(
    Request $request,
    string $token,
    int $fileId,
): StreamedResponse|Response
// ↑ 引数の型                              ↑ 戻り値は2種類のどちらか
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

---

### OneSendで使われているUnion型

Union型は、「複数の型のうち、いずれかを返す」と宣言する記法である。

```php
public function share(string $token): View|Response
```

このメソッドは状況によって戻り値が変わる。

```text
公開可能   → View
期限切れ   → Response（410）
公開停止中 → Response（403）
```

ダウンロードも同様である。

```php
public function download(
    Request $request,
    string $token,
    int $fileId,
): StreamedResponse|Response
```

```text
ダウンロード可能 → StreamedResponse
利用不可         → Response
```

型宣言によって、メソッドを読む人は「常にファイルが返るわけではない」と中身を読む前に分かる。

### nullable型とnull安全演算子は別物

#### nullable型`?Response`

```php
private function unavailableTransferResponse(Transfer $transfer): ?Response
```

戻り値の`?Response`は次のUnion型と同じ意味である。

```php
Response|null
```

```text
利用不可 → Responseを返す
利用可能 → nullを返す
```

これは「型宣言」の話である。

#### null安全演算子`?->`

```php
$transfer->expires_at?->lte(now())
```

こちらの`?->`は、PHPのnull安全演算子である。

概念的には次の条件分岐を短く書いている。

```php
if ($transfer->expires_at === null) {
    return null;
}

return $transfer->expires_at->lte(now());
```

`expires_at`が`null`なら、`lte()`を呼ばずに式全体が`null`になる。値が存在すれば`lte()`を呼ぶ。

```text
?Response  → nullを返してよいという型宣言
?->        → nullなら後続メソッドを呼ばない演算子
```

### `expires_at`で`lte()`を呼べる理由

Transferモデルにはcastが定義されている。

```php
protected function casts(): array
{
    return [
        'expires_at' => 'datetime',
    ];
}
```

このcastにより、DB上の日付文字列がPHPではCarbon日時オブジェクトとして扱われる。

そのため、次のメソッドを呼べる。

```php
$transfer->expires_at->lte(now())
```

- `now()`：現在日時のCarbonオブジェクトを作るLaravelヘルパー
- `lte()`：less than or equal、つまり「引数の日時以下か」を判定する

公開期限の例では次の意味になる。

```text
expires_at <= 現在日時
```

つまり「公開期限が現在時刻以前なら期限切れ」である。

### 型を手掛かりにコードを読む

不明なメソッドを見つけたら、最初に左側の値の型を確認する。

```php
$transfer->expires_at?->lte(now());
```

1. `$transfer`は`Transfer`モデル。
2. `expires_at`はcastによってCarbonまたは`null`。
3. `lte()`はLaravel ControllerのメソッドではなくCarbonのメソッド。
4. 結果は`bool`または`null`。

メソッド名だけを検索するより、「どの型が持つメソッドか」を確認すると定義元を見つけやすい。

#### 公式マニュアル

- [PHP 型宣言](https://www.php.net/manual/ja/language.types.declarations.php)
- [PHP Union型](https://www.php.net/manual/ja/language.types.type-system.php#language.types.type-system.composite.union)
- [PHP null安全演算子](https://www.php.net/manual/ja/language.oop5.basic.php#language.oop5.basic.nullsafe)
- [Laravel 13.x Eloquent Casts](https://laravel.com/framework/docs/13.x/eloquent-mutators#date-casting)
