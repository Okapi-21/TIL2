## Caddyサーバー構築

------

### 前提条件

- **パソコン（サーバ）の名前（ドメイン）を決める**
  　例：yourdomain.com 
  
- **ドメインがサーバの住所（IP）を指しているか確認**  
  　（DNS A/AAAA レコードがサーバのグローバルIPを向く必要あり）

- **サーバの「80番」と「443番」の門を開けておく**  
  　→ Caddyがインターネットからのアクセスを受け取るため

    　80番（HTTP）はLet's Encrypt で証明書を発行するために必要です

    　**コマンド例（ufw使用時）**  
    ```
    sudo ufw allow 80,443/tcp
    sudo ufw reload
    ```

### Caddy公式リポジトリをサーバーにインストール

```bash
sudo apt-get update
sudo apt-get install -y debian-keyring debian-archive-keyring apt-transport-https
curl -fsSL https://dl.cloudsmith.io/public/caddy/stable/gpg.key \
  | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -fsSL https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt \
  | sudo tee /etc/apt/sources.list.d/caddy-stable.list
sudo apt-get update
```

### Caddy本体のインストール

```bash
sudo apt install caddy

# バージョン確認
caddy version
```

### Caddyfileの設定

```bash
sudo nano /etc/caddy/Caddyfile
```
または
```bash
sudo vi /etc/caddy/Caddyfile
```

```
# yourdomain.comに自分のドメインを記述してください
yourdomain.com {
    reverse_proxy 127.0.0.1:3000  # Giteaを使う場合
}
```

**Caddyはこの記述だけでLet’s Encryptによる自動HTTPS化をしてくれます！**

### 設定反映

```bash
# 構文チェック
sudo caddy validate --config /etc/caddy/Caddyfile

# 設定反映（リロード。ダウンタイムなし）
sudo systemctl reload caddy
```

### 動作確認

- ブラウザで https://yourdomain.com を開き、Gitea画面が表示されればOK

### エラー時コマンド

```bash
sudo journalctl -u caddy -f
```
- 証明書が取れないときは、ここに "error" や "challenge" の失敗理由が出ます