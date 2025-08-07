# Ubuntu Server で、 自分の制的ファイルなどの公開方法

## 作成理由 → あまり良い記事がなかったから、議事録みたいな

### このリポジトリ通りで作ったサイト

https://www.moto0314.com

# 動作環境

Ubuntu 24.04.2 LTS (GNU/Linux 6.8.0-71-generic x86_64)

# git install

sudo apt install git<br>
cd ~<br>
ssh-keygen -t ed25519 -f id_ed25519_github -C "example@example.com"<br>
mkdir -p ~/.ssh<br>
chmod 700 ~/.ssh<br>
sudo chmod -R 775 /var/www<br>
nano ~/.ssh/config<br>
の中に、<br>
Host github.com<br>
HostName github.com<br>
User git<br>
IdentityFile ~/.ssh/id_ed25519_github<br>
IdentitiesOnly yes<br>

ssh -T git@github.com<br>

cd /var/www/<br>
で、
git clone example.repository.git<br>

# Apache install

sudo apt install apache2 -y<br>

sudo systemctl status apache2<br>

sudo ufw allow 'Apache'<br>
sudo ufw enable<br>
sudo ufw status<br>

cd /<br>
cd /var/www/<br>
sudo rm -rf html<br>

sudo chown -R user-name:user-name /var/www<br>

git clone example.repository.git<br>

sudo nano /etc/apache2/sites-available/000-default.conf<br>
の中の、 DocumentRoot /var/www/html<br>
を、 DocumentRoot /var/www/example.repository.git<br>
sudo systemctl restart apache2<br>

sudo chown -R www-data:www-data /var/www/example.repository<br>
sudo chmod -R 755 /var/www/example.repository<br>

# cloudflared install

cd ~<br>

wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb<br>

sudo dpkg -i cloudflared-linux-amd64.deb<br>

## インストール確認

cloudflared --version<br>

## cloudflare にログインする → ssh で接続して、リンクからログイン推奨

cloudflared login<br>

## トンネルを作る

cloudflared tunnel create example-tunnel<br>

## 設定ファイルのやつ作る

sudo mkdir -p /etc/cloudflared<br>

# cloudflare にレコードを生成

cloudflared tunnel route dns example-tunnel example.com<br>

# cloudflare を常駐化

sudo cloudflared service install<br>

# ファイルの設定

sudo cp /root/.cloudflared/tunnel-id.json /etc/cloudflared/<br>

sudo nano /etc/cloudflared/config.yml<br>

```yml
tunnel: tunnel-id
credentials-file: /etc/cloudflared/tunnel-id.json

ingress:
  - hostname: ssh.example.com
    service: ssh://localhost:22
  - hostname: example.com
    service: http://localhost:80
  - hostname: www.example.com
    service: http://localhost:80
  - service: http_status:404
```

ls -l /etc/cloudflared/tunnel-id.json<br>

なければ、、、

sudo cp ~/.cloudflared/tunnel-id.json /etc/cloudflared/<br>
sudo chmod 600 /etc/cloudflared/_.json<br>
sudo chown root:root /etc/cloudflared/_.json<br>

sudo systemctl daemon-reload<br>
sudo systemctl restart cloudflared<br>

sudo systemctl status cloudflared<br>

https://example.com<br>

# SSH は、ローカル PC で、

cloudflare の dns 設定<br>

Type → CNAME<br>
Name → ssh.example.com<br>
Target → ほかの dns と同じ<br>
Proxy → On<br>

## その後、、、

sudo nano ~/.ssh/config<br>
その中に、<br>

```txt
Host ssh.moto0314.com
  User username # ← SSH ログインするユーザー名
  ProxyCommand cloudflared access ssh --hostname %h
```

その後、ssh サブドメイン.example.com で接続。<br>
