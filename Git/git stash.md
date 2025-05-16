## 概要

- git stashの基本的な使い方

## git stashとは

- gitのコマンドの1つ。
- 変更差分をコミットせずに一時的に退避させることで保存できる。
- 作業中に他のブランチでの作業が必要になったときなどに便利。

## 使い方

### `git stash`

- 変更差分を退避させる。
- untracked fileは退避されない。

### `git stash -u`

- untracked fileも含めて変更差分を退避させる。

### `git stash save コメント`

- 退避にコメントをつけられる。
- `git stash list`で退避のリストを見るときに便利。

### `git stash list`

- 退避させた変更の一覧表示。
- `git stash save`で保存すると何の差分がわかって便利。

### `git stash list -p`

- `git stash list`と`git diff`の合わせ技。
- あんまり見やすくないし、`git stash save`でコメント残しておけば良いので自分はあまり使わない。

### `git stash apply`

- 退避させた変更を戻す。
- 複数の退避がある場合、最新のものを戻す。
- 最新のものより前の変更を戻す場合は`git stash apply 2`のような形で退避リストの何番目かを指定。
- 何番目かは`git stash list`で確認できる。
- 戻した内容は退避リストに残ったまま。

### `git stash pop`

- `git stash apply`の、戻すと同時に退避リストから削除するバージョン。

### `git stash drop`

- 退避リストから変更を削除する。
- `git stash pop`と同じく、最新の変更が削除される。
- 最新以前のものを削除する場合は`git stash drop 2`みたいな感じで。

### `git stash clear`

- 退避リストを全部削除する。



参考資料

https://qiita.com/keisuke0508/items/4ad7caf544b1ad631fd7#git-stash



