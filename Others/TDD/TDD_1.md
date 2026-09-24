## Laravel TDD ガイド

## 基本方針

TDD（テスト駆動開発）では、機能を完成させてからテストを書くのではなく、実現したい振る舞いをテストで先に表現する。

開発は小さな要件ごとに、次のサイクルを繰り返す。

1. **RED**：必要最小限のテストを書き、未実装の振る舞いを理由に失敗することを確認する。
2. **GREEN**：テストを通すための最小限の実装を行う。
3. **REFACTOR**：テストが通る状態を保ちながら、重複や命名、責務を整理する。

## テスト種別の選び方

### Featureテスト

HTTPリクエストを入口として、Route、Middleware、Controller、Validation、Model、DBなどが連携した結果を確認する。

CRUD、認証、認可、入力チェックなど、Laravelアプリケーションの多くの機能ではFeatureテストを優先する。

```bash
docker compose  exec app_web php artisan make:test Post/CreatePostTest
```

### Unitテスト

HTTPやDBから切り離せる、独立したクラスや計算処理などを確認する。

```bash
docker compose exec app_web php artisan make:test PriceCalculatorTest --unit
```

## 基本的な進め方

### 1. 小さな要件を一つ決める

例：「ログイン済みユーザーは記事を投稿できる」

正常系だけでなく、必要に応じて未認証、認可、バリデーション境界も別のテストとして扱う。

### 2. 必要ならひな型を生成する

Model、Migration、Factoryのひな型はまとめて生成できる。

```bash
docker compose -f Docker/docker-compose.yml exec -T app_web php artisan make:model Post -mf
```

ひな型の生成は構わないが、すべての実装を完成させてからテストを書く進め方にはしない。

Seederは開発環境や初期データの投入に使うもので、通常のテストデータ作成には必要ない。

### 3. Featureテストを書く

テストは「準備・実行・検証」の順に整理する。

```php
public function test_authenticated_user_can_create_a_post(): void
{
    // 準備（Arrange）
    $user = User::factory()->create();

    // 実行（Act）
    $response = $this->actingAs($user)->post('/posts', [
        'title' => 'テスト記事',
        'body' => '本文です。',
    ]);

    // 検証（Assert）
    $response->assertRedirect();

    $this->assertDatabaseHas('posts', [
        'title' => 'テスト記事',
        'user_id' => $user->id,
    ]);
}
```

DBを利用するFeatureテストでは、テスト間でDB状態を共有しないようにする。

```php
use Illuminate\Foundation\Testing\RefreshDatabase;

class CreatePostTest extends TestCase
{
    use RefreshDatabase;
}
```

### 4. REDを確認する

対象テストだけを実行する。

```bash
docker compose -f Docker/docker-compose.yml exec -T app_web php artisan test --filter=authenticated_user_can_create_a_post
```

失敗理由が「機能が未実装だから」であることを確認する。DB接続失敗、構文エラー、テストデータ不足などは、正しいREDとは扱わない。

### 5. GREENにする

Migration、Model、Factory、Route、Controller、Form Requestなど、テストを通すために必要なものだけを実装する。

実装後、同じ対象テストを再実行して成功を確認する。

### 6. REFACTORする

テストが通る状態を維持しながら、重複、責務、命名、Laravelの規約との整合を整理する。整理後も対象テストを再実行する。

## FactoryとSeederの使い分け

- Factory：各テストに必要なデータを作るために使用する。
- Seeder：開発環境のダミーデータや、システムに必要な初期データを投入するために使用する。

テストはSeederの実行済みデータに依存させず、必要なデータをテスト内でFactoryから作成する。

```php
$user = User::factory()->create();
```

## 実装完了時の確認

対象テストから順に、関連テスト、全体テストへ確認範囲を広げる。

```bash
docker compose -f Docker/docker-compose.yml exec -T app_web php artisan test --filter=<テスト名>
docker compose -f Docker/docker-compose.yml exec -T app_web composer test
docker compose -f Docker/docker-compose.yml exec -T app_web vendor/bin/pint --dirty
docker compose -f Docker/docker-compose.yml exec -T app_web composer test
```

フロントエンドを変更した場合は、あわせてビルドを確認する。

```bash
docker compose -f Docker/docker-compose.yml exec -T app_web npm run build
```

最後に`git diff`を確認し、デバッグコードや意図しない変更が含まれていないことを確認する。