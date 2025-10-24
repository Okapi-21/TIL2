### Blackhole適用手順（Round cube ）

＊ example.comは実際に利用しているドメイン名に変更してください。

#### 1. 現在の設定確認

```bash
cat /etc/postfix/main.cf
```

main.cf に以下のような行があるか確認してください。

```
virtual_mailbox_domains = example.com
virtual_mailbox_maps = hash:/etc/postfix/vmailbox
#無い場合は後で追加します。
virtual_alias_maps = xxxxxxxxxxxxx
```

------

#### 2. /etc/postfix/virtual の作成・編集

存在しないユーザー宛を blackhole に流す処理を/etc/postfix/virtual に設定します。  

```text
@example.com      blackhole      # catch-all to blackhole
```

> これらの行は /etc/postfix/vmailbox ではなく /etc/postfix/virtual に記載します。
> 右辺は "blackhole" のみとしてください。

---

#### 3. 例外マップの作成

/etc/postfix/vmailbox の1列目（実在メールアドレス）を用いて、実在ユーザーは「自分自身へ」エイリアスするマップを作成します。

```bash
awk '!/^($|#)/{print $1 " " $1}' /etc/postfix/vmailbox > /etc/postfix/virtual_exceptions
```

（参考）vmailbox の1列目がローカル部のみで、2列目の先頭がドメイン名の場合は次を利用してください。

```bash
awk '!/^($|#)/{
  local=$1; dom=$2; sub(/\/.*/,"",dom);
  print local "@" dom " " local "@" dom
}' /etc/postfix/vmailbox > /etc/postfix/virtual_exceptions
```

> vmailboxの内容を変更（ユーザー追加・削除）した場合は、virtual_exceptionsも必ず再生成し、postmap・reloadを実施してください。

---

#### 4. マップをDB化

```bash
postmap /etc/postfix/virtual
postmap /etc/postfix/virtual_exceptions
```

確認

```bash
postmap -q @example.com      /etc/postfix/virtual         # => blackhole

#user1@example.comにはDBに実在するユーザーメールアドレスを入力して下さい
postmap -q user1@example.com /etc/postfix/virtual_exceptions  # => user1@example.com

postmap -q @example.com /etc/postfix/virtual | cat -A   # 期待: blackhole$
```

> 編集時にWindows改行(CRLF)が混入している場合は正常動作しないことがあります。  
> 必要に応じて `dos2unix /etc/postfix/virtual` を実行してください。

---

#### 5. /etc/aliases に blackhole を追加 → DB更新

```text
blackhole: /dev/null
```

```bash
newaliases
```

> aliasesを編集した場合は必ずnewaliasesを再実行してください。

---

#### 6. main.cf の設定

`/etc/postfix/main.cf` を編集し、以下のように設定します。  
**virtual_exceptions → virtual の順序を必ず守るようにお願いします。**

```
virtual_mailbox_domains = example.com
virtual_mailbox_maps = hash:/etc/postfix/vmailbox
virtual_alias_maps = hash:/etc/postfix/virtual_exceptions, hash:/etc/postfix/virtual
```

反映:

```bash
postfix reload
# または
systemctl restart postfix
```

---

#### 7. 動作確認

- 実在するユーザー宛：正常通りメール受信可能
- 実在しないユーザー宛：サーバー上で破棄され、送信者にはエラーが返らない

------

#### 補足

- 即時拒否（REJECT/550）をしたい場合は `smtpd_recipient_restrictions` での `check_recipient_access` + `REJECT/DISCARD` もありますが、本手順は「受信したふりで破棄（silent discard）」を目的としています。したがって、`smtpd_recipient_restrictions = reject_unverified_recipient`としている場合は**正常にblackholeが反応しなくなってしまう**ので、削除かコメントアウトをお願いします。