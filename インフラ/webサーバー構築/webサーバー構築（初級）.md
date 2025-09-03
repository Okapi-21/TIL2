## webサーバーの構築

## Apache（Webサーバー）のインストール

1. **ターミナルを開く**  
   Oracle Linuxにログインしたら、ターミナルを開いてください。

2. **Apacheのインストール**  
   以下のコマンドを入力して、Apacheをインストールします。
   ```bash
   sudo dnf install httpd
   ```
   - `sudo`: 管理者権限でコマンドを実行するための命令です。
   - `dnf`: Oracle Linuxでソフトウェアを管理するコマンドです。
   - `install httpd`: `httpd`（Apacheのこと）をインストールします。

3. **インストールの確認**  
   インストール中に「y/n」と聞かれた場合は、`y`（Yes）を押して進めてください。

---

### Apacheの起動と自動起動設定

1. **Apacheを起動する**  
   以下のコマンドを入力して、Apacheを起動します。
   ```bash
   sudo systemctl start httpd
   ```

2. **Apacheを自動起動に設定**  
   サーバーを再起動した時もApacheが自動で立ち上がるように設定します。
   ```bash
   sudo systemctl enable httpd
   ```

---

### ファイアウォールの設定

Webサーバーに外部からアクセスできるように、ファイアウォールの設定をします。

```bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
```

---

ここまでで、Webサーバー（Apache）が起動し、外部からアクセスできる状態になりました。

---

## Apacheが起動していてもブラウザに表示されない場合の確認手順

#### 1. サーバーのIPアドレス確認

```bash
ip addr
```
- `inet` のあとに出てくる数字（例: `192.168.0.10`）がサーバーのIPです。
- ブラウザで `http://<IPアドレス>/` にアクセスしてください。

---

#### 2. Apacheのデフォルトページの場所

Apacheのデフォルト公開ディレクトリは `/var/www/html` です。

```bash
ls /var/www/html
```

- 何もファイルが表示されない場合、テスト用ファイルを作成します。

```bash
echo "Apache動作テストページ" | sudo tee /var/www/html/index.html
```

- もう一度ブラウザでアクセスします。

---

#### 3. SELinuxの状態確認

```bash
getenforce
```
- `Enforcing` と表示された場合、アクセスがブロックされている可能性があります。

- 一時的に `Permissive` に切り替えるには:

```bash
sudo setenforce 0
```

- この状態で再度ブラウザでアクセスしてください。

---

#### 4. アクセス元PCからのネットワーク疎通確認

```bash
ping <サーバーのIPアドレス>
```
- 通らない場合は、ネットワーク設定やセグメントの問題の可能性があります。

---

#### 5. Apacheのログ確認

```bash
sudo tail -f /var/log/httpd/error_log
```
- ブラウザでアクセスしてエラーが出ていないか確認します。

---

## WordPressのダウンロードと設置

#### 1. 必要なパッケージのインストール

```bash
sudo dnf install php php-mysqlnd php-fpm php-gd php-xml php-mbstring php-json
```

#### 2. データベース（MariaDB）のインストール

```bash
sudo dnf install mariadb-server
sudo systemctl enable --now mariadb
```

#### 3. WordPressのダウンロード

```bash
cd /tmp
curl -O https://ja.wordpress.org/latest-ja.tar.gz
```

#### 4. WordPressの展開と設置

```bash
tar zxvf latest-ja.tar.gz
sudo mv wordpress/* /var/www/html/
sudo chown -R apache:apache /var/www/html/ 
```

#### 5. WordPress用データベースの作成

```bash
sudo mysql -u root -p
```
MariaDB内で：

```sql
CREATE DATABASE wordpress;
CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wpuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```
※`password`は任意の安全なパスワードに変更してください。

#### 6. WordPressセットアップ画面にアクセス

ブラウザで  
`http://192.168.11.111/`  
にアクセスすると、WordPressのインストール画面が表示されます。