## Seeder 関連



#### Seeder とは

「Seeder（シーダー）」は主にwebアプリケーション開発で用いられる用語で、開発やテストの為のDBに初期データを投入する仕組みやファイルのことを示す。laravelやrailsなどのフレームワークでよく登場する。

＊ seed（種）なので、seeder は種を撒く人という意味ですね。

#### Seeder 作成

seeder 作成時に必要なコマンド

```
php artisan make:seeder xxxxxSeeder
```

ファイル名は「テーブル名+Seeder」という命名規則に従うのが一般的

このコマンドにより、database / seeders / UserSeeder.phpというファイルが自動生成される。



生成されるファイルに関しては基本構造が用意されていて、run()メソッド内にデータ挿入処理を記述する

```
public function run()
{
    // 単一ユーザーの作成例
    \App\Models\User::create([
        'name' => '山田太郎',
        'email' => 'yamada@example.com',
        'password' => bcrypt('password123')
    ]);
    
    // 複数ユーザーの一括作成例
    for ($i = 1; $i <= 10; $i++) {
        \App\Models\User::create([
            'name' => "テストユーザー{$i}",
            'email' => "test{$i}@example.com",
            'password' => bcrypt('password')
        ]);
    }
}
```



作成したSeeder を実行するために databases / seeders / DatabaseSeeder.phpのrun()メソッド内に以下を追記します。

``` 
$this->call([xxxxSeeder::class]);
```



以下のコマンドでSeeder を実行し、DBにテストデータを挿入することができる
```
php artisan db:seed
```



#### +α

>Seeder と ファクトリ（Factory）
>
>　Seeder は「DBに初期データを投入する仕組み」であるが、ファクトリ（Factory）は「ダミーデータを自動生成」する仕組みである。したがって、ファクトリを用いてSeeder を作成する際、挙動としては 「Seeder がファクトリを呼び出して、DBにダミーデータを投入する」という流れになる。



### DB移行作業

1. DB構造が同じ場合 
   　DB構造（アソシエーションやカラム名、データ属性）が一緒なので、SQL文でそのままインサートしてしまっても問題ない

2. DB構造が異なる場合

   　DB構造が異なっているので、そのままデータをインサートするとDBの整合性が取れなくなり、アプリが誤作動を起こす / 正常に動作しなくなる。

      →旧DBからデータを新DBへ移行する為のデータを抜き取った上で、新DBに適合する形になるようデータを整形する必要がある

      → データ整形も数個 ~ 十数個の場合は手作業で問題ないが、数百 ~ 数千個のデータが存在する場合は手作業では不可能なので

      → 「旧DBデータを新DBに適合するように整形する為のファイル・ロジックを組む必要がある」





CSVファイルをPHP配列へ変換する為のロジック

```php
<?php
/**
 * CSV を PHP 配列ファイルに変換（小規模向け）
 * 使い方:
 *   php scripts/csv_to_php_array.php storage/import/old_data.csv storage/import/data_tbl_array.php
 *
 * 入力CSVヘッダ想定:
 *   data_id,data_naiyo,make_date,make_time,sled_id,upd_id[,created_at]
 */

// コマンドライン引数の個数チェック。引数が足りなければエラー表示して終了
if ($argc < 3) {
    fwrite(STDERR, "Usage: php {$argv[0]} input.csv output.php\n");
    exit(1);
}
// 入力CSVファイルのパスを変数に代入
$in  = $argv[1];
// 出力PHPファイルのパスを変数に代入
$out = $argv[2];
// 入力ファイルが読めるかチェック。読めなければエラー表示して終了
if (!is_readable($in)) {
    fwrite(STDERR, "Cannot read: {$in}\n");
    exit(2);
}

// CSVファイルを読み込みモードで開く
$fh = fopen($in, 'r');
if ($fh === false) {
    fwrite(STDERR, "Failed to open: {$in}\n");
    exit(3);
}
// 最初の1行（ヘッダ）を取得
$header = fgetcsv($fh);
if ($header === false) {
    fwrite(STDERR, "Empty CSV: {$in}\n");
    exit(4);
}

$rows = []; // 全データを格納する配列
$count = 0; // 行数カウンタ

// 2行目以降、CSVデータを1行ずつ処理
while (($row = fgetcsv($fh)) !== false) {
    // 空行の場合はスキップ
    if ($row === [null] || $row === null) continue;

    $assoc = []; // 1行分のデータを連想配列として格納
    // ヘッダー名と値を対応付けて連想配列にする
    foreach ($header as $i => $col) {
        $key = trim((string)$col);
        $assoc[$key] = array_key_exists($i, $row) ? $row[$i] : null;
    }

    // 旧ID（data_id）を取得し、なければnull
    $legacyId = isset($assoc['data_id']) && $assoc['data_id'] !== '' ? (int)$assoc['data_id'] : null;
    // 親の旧ID（sled_id）を取得。0の場合はnull（親要素と判定）
    $parentLegacy = isset($assoc['sled_id']) && $assoc['sled_id'] !== '' ? (int)$assoc['sled_id'] : null;
    if ($parentLegacy === 0) $parentLegacy = null;

    // 投稿内容（description）を取得し、改行コードを統一
    $desc = (string)($assoc['data_naiyo'] ?? '');
    $desc = str_replace(["\r\n", "\r"], "\n", $desc);

    // 日付と時刻を合成してtimestampにする。両方なければcreated_atやdateのみ
    $date = $assoc['make_date'] ?? $assoc['make_data'] ?? null;
    $time = $assoc['make_time'] ?? null;
    $created = $assoc['created_at'] ?? null;
    if (!empty($date) && !empty($time)) {
        $ts = trim("$date $time");
    } elseif (!empty($created)) {
        $ts = $created;
    } elseif (!empty($date)) {
        $ts = $date;
    } else {
        $ts = null; // 日時情報がない場合はnull
    }

    // 必要な情報を配列にまとめて格納
    $rows[] = [
        'legacy_id'         => $legacyId,
        'parent_legacy_id'  => $parentLegacy,
        'description'       => $desc,
        'timestamp'         => $ts,
        'upd_id'            => isset($assoc['upd_id']) && $assoc['upd_id'] !== '' ? (int)$assoc['upd_id'] : null,
    ];
    $count++; // 行数をカウントアップ
}
// ファイルを閉じる
fclose($fh);

// PHP配列として出力ファイルに保存
file_put_contents($out, "<?php\nreturn " . var_export($rows, true) . ";\n");
// 標準出力に処理件数を表示
fwrite(STDOUT, "Wrote {$count} rows to {$out}\n");
```



seeder 
```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Carbon;

class OldDataFromArraySeeder extends Seeder
{
    /**
     * 実行前に以下を調整してください:
     * - $arrayPath: 生成した PHP 配列ファイルの場所
     * - $parentTable, $childTable: 新DBの実テーブル名（例: boards, comments）
     * - $childForeignKey: 子→親の外部キー列名（例: board_id, post_id, parent_id）
     * - $contentColumnParent / $contentColumnChild: 本文を入れる列名（例: content, body, description）
     * - $userIdForLegacy: 旧データを紐付けるユーザーID
     * - 必須なら他の固定カラム（category_id 等）を $parentFixed/$childFixed に追加
     *
     * 実行:
     *   php artisan db:seed --class=Database\\Seeders\\OldDataFromArraySeeder
     */
    public function run()
    {
        // 1. 配列ファイルの読み込み
        $arrayPath = base_path('storage/import/data_tbl_array.php');
        if (!file_exists($arrayPath)) {
            $this->command->error("配列ファイルが見つかりません: {$arrayPath}");
            return;
        }
        $rows = require $arrayPath;
        if (!is_array($rows)) {
            $this->command->error("配列ファイルの内容が不正です: {$arrayPath}");
            return;
        }

        // 2. 新DBのテーブル名やカラム名の設定
        $parentTable        = 'parents';     // 親テーブル名（'boards'）
        $childTable         = 'children';    // 子テーブル名（'boardcomments'）
        $childForeignKey    = 'parent_id';   // 子→親の外部キー名（'board_id'）
      	$contentColumnParent= 'description'; // 親の本文カラム名
        $contentColumnChild = 'description'; // 子の本文カラム名
        $userIdForLegacy    = 1;             // 旧データ用ユーザーID

        // 3. 親データのinsert（親IDをマッピング）
        $legacyToNew = []; // 旧ID→新IDの対応表
        foreach ($rows as $r) {
            // 親データかどうか判定（parent_legacy_idがnullまたは0）
            $parentLegacy = $r['parent_legacy_id'] ?? null;
            $isParent = ($parentLegacy === null || $parentLegacy === 0);
            if (!$isParent) continue;

            // 日時をCarbonで正規化
            $createdAt = $this->normalizeTimestamp($r['timestamp'] ?? null);

            // データ配列作成
            $data = [
                $contentColumnParent => $r['description'] ?? null,
                'user_id'            => $userIdForLegacy,
                'created_at'         => $createdAt,
                'updated_at'         => $createdAt,
            ];
            // insertして新IDを取得
            $newId = DB::table($parentTable)->insertGetId($data);

            // 旧ID→新IDの対応表に記録
            if (!empty($r['legacy_id'])) {
                $legacyToNew[(int)$r['legacy_id']] = $newId;
            }
        }

        // 4. 子データのinsert（親IDをマッピングして挿入）
        foreach ($rows as $r) {
            $parentLegacy = $r['parent_legacy_id'] ?? null;
            $isParent = ($parentLegacy === null || $parentLegacy === 0);
            if ($isParent) continue;

            // 旧親IDから新親IDを取得
            $newParentId = $legacyToNew[(int)$parentLegacy] ?? null;
            if ($newParentId === null) {
                $this->command->warn("親ID未解決のためスキップ: child legacy={$r['legacy_id']} parent_legacy={$parentLegacy}");
                continue;
            }

            $createdAt = $this->normalizeTimestamp($r['timestamp'] ?? null);

            $data = [
                $childForeignKey     => $newParentId,
                $contentColumnChild  => $r['description'] ?? null,
                'user_id'            => $userIdForLegacy,
                'created_at'         => $createdAt,
                'updated_at'         => $createdAt,
            ];
            DB::table($childTable)->insert($data);
        }
    }

    // タイムスタンプをCarbonで正規化する関数
    protected function normalizeTimestamp($ts)
    {
        if (empty($ts)) {
            return Carbon::now()->toDateTimeString();
        }
        try {
            return Carbon::parse($ts)->toDateTimeString();
        } catch (\Throwable $e) {
            return Carbon::now()->toDateTimeString();
        }
    }
}
```









