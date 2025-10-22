# Ubuntu Server で、 自分の静的ファイルなどの公開方法

## 作成理由 → あまり良い記事がなかったから、議事録みたいな

### このリポジトリ通りで作ったサイト

https://www.moto0314.com

# 動作環境

Ubuntu 24.04.3 LTS (GNU/Linux 6.8.0-71-generic x86_64)

# git install

```sh
sudo apt install git
cd .ssh
ssh-keygen -t ed25519 -f id_ed25519_github -C "example@example.com"
chmod 700 ~/.ssh
nano ~/.ssh/config
```

```txt
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_github
  IdentitiesOnly yes
```

```sh
ssh -T git@github.com
```

# cloudflared install

```sh
cd ~
# cloudflare
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
sudo dpkg -i cloudflared-linux-amd64.deb
# cloudflare のinstall確認
cloudflared --version
# cloudflare にログインして、domainを登録する
cloudflared login
# トンネルを作る
cloudflared tunnel create example-tunnel
# cloudflare の設定ファイルを作る
sudo mkdir -p /etc/cloudflared
sudo cp ~/.cloudflared/tunnel-id.json /etc/cloudflared/
sudo chmod 600 /etc/cloudflared/tunnel-id.json
sudo chown root:root /etc/cloudflared/tunnel-id.json
```

```sh
# cloudflare にレコードを生成
cloudflared tunnel route dns example-tunnel example.com
```

```yml
tunnel: tunnel-id
credentials-file: /etc/cloudflared/tunnel-id.json

ingress:
  - hostname: ssh.example.com
    service: ssh://localhost:22
  - hostname: example.com
    service: http://localhost:${PORT}
  - service: http_status:404
```

```sh
# cloudflare を常駐化
sudo cloudflared service install
sudo systemctl daemon-reload
sudo systemctl restart cloudflared
sudo systemctl status cloudflared
```

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
Host server
  HostName ssh.example.com
  User username # ← SSH ログインするユーザー名
  ProxyCommand cloudflared access ssh --hostname %h
```

その後、ssh server で接続。<br>
