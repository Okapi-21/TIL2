## SQL

------

- 各テーブルが持つ「属性（カラム名）」だけを知りたい場合
  ```
  DESCRIBE xxxxxxx;
  ```

  

- DBのバックアップを取りたい時




- 一つのテーブルを消去したい時

  ```bash
  #use databases コマンドを打ってから
  DROP TABLE IF EXISTS xxxxxx;
  ```

  

ダンプ（バックアップ）をとる時のコマンド

```
mysqldump -u ユーザー名 -p --databases DB名1 DB名2 --single-transation > ファイルパス or ファイル名
```

