# Ubuntu Server で、 自分の制的ファイルなどの公開方法

## 作成理由 → あまり良い記事がなかったから、議事録みたいな

### このリポジトリ通りで作ったサイト

https://www.moto0314.com

# 動作環境

Ubuntu 24.04.2 LTS (GNU/Linux 6.8.0-71-generic x86_64)

# git install

sudo apt install git
cd ~
ssh-keygen -t ed25519 -f id_ed25519_github -C "example@ths.hal.ac.jp"
mkdir -p ~/.ssh
chmod 700 ~/.ssh
sudo chmod -R 775 /var/www
nano ~/.ssh/config
の中に、
Host github.com
HostName github.com
User git
IdentityFile ~/.ssh/id_ed25519_github
IdentitiesOnly yes

ssh -T git@github.com

cd /var/www/
で、
git clone example.repository.git

# Apache install

sudo apt install apache2 -y

sudo systemctl status apache2

sudo ufw allow 'Apache'
sudo ufw enable
sudo ufw status

cd /
cd /var/www/
rm -rf html
git clone example.repository.git
sudo nano /etc/apache2/sites-available/000-default.conf
の中の、 DocumentRoot /var/www/html
を、 DocumentRoot /var/www/example.repository.git
sudo systemctl restart apache2

sudo chown -R www-data:www-data /var/www/example.repository
sudo chmod -R 755 /var/www/example.repository

# cloudflared install

wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb

sudo dpkg -i cloudflared-linux-amd64.deb

## インストール確認

cloudflared --version

## cloudflare にログインする → ssh で接続して、リンクからログイン推奨

cloudflared login

## トンネルを作る

cloudflared tunnel create example-tunnel

## 設定ファイルのやつ作る

sudo mkdir -p /etc/cloudflared

# cloudflare にレコードを生成

cloudflared tunnel route dns moto-tunnel example.com

# cloudflare を常駐化

sudo cloudflared service install

# ファイルの設定

sudo cp /root/.cloudflared/tunnel-id.json /etc/cloudflared/

sudo nano /etc/cloudflared/config.yml

```yml
tunnel: tunnel-id
credentials-file: /etc/cloudflared/tunnel-id.json

ingress:
  - hostname: ssh.moto0314.com
    service: ssh://localhost:22
  - hostname: example.com
    service: http://localhost:80
  - hostname: www.example.com
    service: http://localhost:80
  - service: http_status:404
```

ls -l /etc/cloudflared/tunnel-id.json

なければ、、、

sudo cp ~/.cloudflared/tunnel-id.json /etc/cloudflared/
sudo chmod 600 /etc/cloudflared/_.json
sudo chown root:root /etc/cloudflared/_.json

sudo systemctl daemon-reload
sudo systemctl restart cloudflared

sudo systemctl status cloudflared

https://example.com

# SSH は、ローカル PC で、

cloudflare の dns 設定

Type → CNAME
Name → ssh.example.com
Target → ほかの dns と同じ
Proxy → On

## その後、、、

sudo nano ~/.ssh/config
その中に、

```txt
Host ssh.moto0314.com
  User username # ← SSH ログインするユーザー名
  ProxyCommand cloudflared access ssh --hostname %h
```

その後、ssh サブドメイン.example.com で接続。
