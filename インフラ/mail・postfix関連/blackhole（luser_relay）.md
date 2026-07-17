### Postfix で「存在しないユーザー宛メールをサイレント破棄」する手順

この文書は、Roundcubeを使っている/いないに関わらず、Postfixサーバーで「存在しないローカルユーザー宛てのメールだけをエラー通知なしで破棄（blackhole）」するための手順書です。



### 想定環境・この手順が向いている人
- Postfix はローカルユーザー配信（/etc/passwd + /etc/aliases + Maildir など）で運用している
  - 例: main.cf に `virtual_mailbox_maps` が無い、または仮想ユーザー運用をしていない
- 「存在するユーザー宛は今までどおり配信」「存在しないユーザー宛は捨てたい（バウンスさせたくない）」
- `$mydestination` 宛（例: user@yourdomain）にのみ適用できればよい

注記
- Mailman 等で `virtual_alias_domains` を使っている場合（例: ml.example.jp）は、この手順の影響を受けません（ローカル配信ではないため）。
- 重要: `luser_relay` は Postfix の local(8) 配送でのみ有効です。`mailbox_transport` を LMTP などに変更している場合はこの方式は効きません（別方式が必要）。

---

### 変更の全体像（なぜそれをするのか）
- `luser_relay` を設定して「不明なローカルユーザー宛メール」を `blackhole` というエイリアスにリレーします。
- `blackhole` エイリアスは /etc/aliases で `/dev/null` に向けます。これにより、メールはローカルで破棄されます。
- `local_recipient_maps` を空にして「SMTPの段階で不明ユーザーを拒否」する機能を止め、ローカル配送(local)まで進めます。
- もし `reject_unverified_recipient` が有効だと SMTP 段階で 550 拒否してしまい、サイレント破棄が成立しません。必ず外します。

---

## 作業前チェック（5分）
1) 設定のバックアップ
```bash
sudo cp -a /etc/postfix/main.cf /etc/postfix/main.cf.bak.$(date +%F-%H%M%S)
sudo cp -a /etc/aliases /etc/aliases.bak.$(date +%F-%H%M%S)
```

2) 現在の重要設定を確認
```bash
sudo postconf -n | grep -E '^(mydestination|alias_maps|virtual_(mailbox|alias)_(domains|maps)|smtpd_(recipient|relay)_restrictions|recipient_delimiter|home_mailbox|relayhost|mailbox_transport|local_transport)\b'
```
見るポイント
- `mydestination` に対象ドメインが含まれている（ローカル配信対象）
- `alias_maps` に `hash:/etc/aliases` が含まれている（aliases が使われる）
- `smtpd_recipient_restrictions` に `reject_unverified_recipient` が入っていたら、後で外す
- `smtpd_relay_restrictions` や `check_recipient_access` で REJECT していると luser_relay まで進みません（必要に応じて見直し）
- `mailbox_transport` や `local_transport` をカスタムしている場合、local(8) 経由でないと luser_relay は効きません



## 手順

### 1. blackhole エイリアスの作成（捨て先の用意）
「blackhole という宛先に来たメールは /dev/null へ捨てる」定義を aliases に追加します。
```text
# /etc/aliases に追記
blackhole: /dev/null
```
反映（aliases DB を更新）
```bash
sudo newaliases
```
なぜ必要？
- luser_relay で `blackhole` に流したメールを最終的に捨てるための受け皿です。
- newaliases を忘れると、Postfix が新しい設定を認識しません。

---

### 2. main.cf の編集
以下の3点を設定します。
- `local_recipient_maps` を「空」にして、SMTP段階の未知ユーザー拒否を無効化
- `luser_relay` で「不明ローカルユーザー宛は blackhole へ」
- `smtpd_recipient_restrictions` から `reject_unverified_recipient` を外す（サイレント破棄を阻害するため）

変更例（差分イメージ）
```diff
- smtpd_recipient_restrictions = permit_mynetworks,
-                                permit_sasl_authenticated,
-                                reject_unauth_destination,
-                                reject_unknown_recipient_domain,
-                                reject_unverified_recipient


+ smtpd_recipient_restrictions = permit_mynetworks,
+                                permit_sasl_authenticated,
+                                reject_unauth_destination,
+                                reject_unknown_recipient_domain

+# ↑ reject_unverified_recipient を外す（またはコメントアウト）

+# 追加（空にする）
+ local_recipient_maps =
+
+# 追加（未知ローカルユーザー宛は blackhole へ）
+ luser_relay = blackhole
```

実際の編集（どちらでもOK）
```bash
# エディタ派
sudo vi /etc/postfix/main.cf

# 直接反映派（タイプミスを避けたい場合に推奨）
sudo postconf -e 'local_recipient_maps ='
sudo postconf -e 'luser_relay = blackhole'
sudo postconf -e 'smtpd_recipient_restrictions = permit_mynetworks, permit_sasl_authenticated, reject_unauth_destination, reject_unknown_recipient_domain'
```

編集後の確認
```bash
sudo postconf -n | grep -E '^(local_recipient_maps|luser_relay|smtpd_recipient_restrictions)\b'
# 期待値:
# local_recipient_maps =
# luser_relay = blackhole
# smtpd_recipient_restrictions に reject_unverified_recipient が含まれていない
```

---

### 3. Postfix の反映と構文チェック
```bash
sudo postfix check     # 構文チェック（エラーが無いか）
sudo postfix reload    # 設定を再読み込み
# うまくいかない/不安なら
# sudo systemctl restart postfix
```

---

## 動作確認
ログの場所
- RHEL/CentOS系: /var/log/maillog
- Debian/Ubuntu系: /var/log/mail.log

1) 存在しないユーザー宛
```bash
DOM=$(postconf -h mydomain)
echo "bh test" | mail -s "bh test" notfound@"$DOM"
sleep 2
sudo tail -n 200 /var/log/maillog | grep -Ei 'notfound@|luser_relay|alias|blackhole|local|status'
```
期待
- 送信者にバウンスメールが届かない
- ログに「local(8) → luser_relay=blackhole → aliases → /dev/null」的な流れが見える

2) 存在するユーザー宛（通常通り届くこと）
```bash
echo "ok test" | mail -s "ok test" existinguser@"$DOM"
sleep 2
sudo tail -n 200 /var/log/maillog | grep -Ei 'existinguser@|delivered|status=sent|Maildir'
```

3) 旧キューが残っている場合（任意）
```bash
mailq
# 特定IDだけ削除（慎重に）
# sudo postsuper -d QUEUE_ID
```

---

## よくあるつまずきと対処
- まだバウンス（配信失敗通知）が来る
  - `smtpd_recipient_restrictions` に `reject_unverified_recipient` が残っていないか
  - `local_recipient_maps =` が空になっているか
  - 別のポリシーで REJECT（例: `check_recipient_access`、`smtpd_relay_restrictions`、postscreen 等）
  - 宛先ドメインが本当に `$mydestination` の対象か（ローカル配信でなければ luser_relay は効かない）

- 捨てられずに配送されてしまう／unknown user で止まる
  - `/etc/aliases` に `blackhole: /dev/null` があるか、`newaliases` 済みか
  - `alias_maps` に `hash:/etc/aliases` が含まれているか（`postconf -n | grep ^alias_maps`）
  - ログに `alias` ルックアップの痕跡が出ているか

- セキュリティ（オープンリレー）について
  - `reject_unauth_destination` は絶対に残してください（既定で入っていればOK）
  - `mynetworks` の範囲は組織ポリシーに合っているか確認

- Mailman（仮想ドメイン）への影響
  - `virtual_alias_domains` や `virtual_alias_maps` は別の配送系統です。今回の方式Aは基本影響しません。

---

## ロールバック（元に戻したいとき）
1) main.cf の変更を戻す（バックアップの値に戻すのが安全）
```diff
- local_recipient_maps =
- luser_relay = blackhole
+ # 例: 以前の既定値に戻す場合（環境差あり）
+ # local_recipient_maps = unix:passwd.byname $alias_maps
+ # luser_relay =    # 行を削除/コメントアウト
+ # （必要なら reject_unverified_recipient を recipient_restrictions に再追加）
```

2) 反映
```bash
sudo postfix reload
```

3) aliases の `blackhole` は残しても問題ありません（不要なら削除→`newaliases`）

---

## 参考
- 効く対象: `$mydestination` にマッチする「ローカル配信」の未知ユーザー
- 効かない対象:
  - `virtual_alias_domains` などの仮想ドメイン配信（Mailmanなど）
  - 外部中継（relayhost 経由）
- 特定ドメインだけ捨てたい、または仮想ユーザー運用の場合は「方式B（virtual_alias_maps＋例外リスト）」を検討