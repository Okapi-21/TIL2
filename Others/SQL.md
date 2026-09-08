## SQL

------

- 各テーブルが持つ「属性（カラム名）」だけを知りたい場合
  ```
  DESCRIBE xxxxxxx;
  ```
  
- 新しいDBを作りたい時
  ```
  CREATE DATABASE 新しいDB名;
  ```
  
- 既存のDBを削除したい時
  
  ```
  DROP DARABASE 既存のDB名;
  ```
  
- 新しいテーブルを作成したい場合

  ```bash
  create table 新しいテーブル名(
  	カラム名		データ型　　制約（PRIMARY KEY, NOT NULL, UNIQUE, etc..；,
  );
  ```
  
  
  
- DBのバックアップを取りたい時（postgresの場合）

  ```
  pg_dump -U ユーザー名　データベース名　＞ バックアップ名.sql
  ```
  
  （スキーマのみ）
  
  ```
  pg_dump -U ユーザー名　-s データベース名　＞ バックアップ名.sql
  ```
  
  （データのみ）
  
  ```
  pg_dump -U ユーザー名　-a データベース名　＞ バックアップ名.sql
  ```
  
  （MySQLの場合）
  
  ```bash
  mysqldump -u ユーザー名 -p データベース名 > バックアップ名.sql
  ```
  
  （スキーマのみ）
  
  ```bash
  mysqldump -u ユーザー名 -p --no-data データベース名 テーブル名 > schema_テーブル名.sql
  ```
  
  （データのみ）
  ```bash
  mysqldump -u ユーザー名 -p --no-create-info データベース名 テーブル名 > data_テーブル名.sql
  ```
  
- DBテーブルの文字数更新のためのコマンド
  ```bash 
  #mysql 
  ALTER TABLE テーブル名 MODIFY カラム名 VARCHAR(新しい文字数);
  ```
  
  

- ダンプの適応

  コマンドライン

  ```
  psql -U ユーザー名 -d データベース名 -f ダンプファイル名.sql
  ```

  対話式
  ```
  \i /パス/ダンプファイル名.sql
  ```


- 一つのテーブルを消去したい時

  ```bash
  #use databases コマンドを打ってから
  DROP TABLE IF EXISTS xxxxxx;
  ```

ダンプ（バックアップ）をとる時のコマンド

```
mysqldump -u ユーザー名 -p --databases DB名1 DB名2 --single-transation > ファイルパス or ファイル名
```

ダンプを流し込む時のコマンド

```
psql -U ユーザー名 -d データベース名 -f /パス/ダンプファイル.sql
```

