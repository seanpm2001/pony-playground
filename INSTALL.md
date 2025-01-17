# Setup playground server

## From Ubuntu 18.04

```
# let's update first
sudo apt-get update

# allow packages to be installed over https
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release \
    gnupg-agent \
    software-properties-common \
    libssl-dev \
    pkg-config \
    build-essential \
    python-pip

sudo pip install pygments
```

### Install Docker

See https://docs.docker.com/engine/install/ubuntu/

It boils down to:

```
# cleanup old stuff
sudo apt-get remove docker docker-engine docker.io containerd runc

# add Docker GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# add docker stable repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
# update to get latest package listings after adding Docker repository
sudo apt-get update

# install latest Docker
sudo apt-get install -y docker-ce docker-ce-cli containerd.io
```

### Webserver setup

```
add-apt-repository ppa:certbot/certbot
apt-get update
apt-get install -y nginx python-certbot-nginx
```

Create /etc/nginx/sites-enabled/playground.ponylang.io.conf

```
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    root /var/www/html;
    server_name playground.ponylang.io;

    location / {
      proxy_pass      http://127.0.0.1:8000;
    }
}
```

```
rm /etc/nginx/sites-enabled/default
ln -sf /etc/nginx/sites-available/playground.ponylang.io.conf /etc/nginx/sites-enabled/playground.ponylang.io.conf

nginx -t && nginx -s reload
```

### SSL setup

```
certbot --nginx -d playground.ponylang.io -m ponylang.main@gmail.com
```

crontab -e

```
0 12 * * * /usr/bin/certbot renew --quiet
05 12 * * * systemctl restart playground
```

### Start docker

```
systemctl enable docker
systemctl start docker
```

### Install rust

```
curl https://sh.rustup.rs | sh
```

select `1` from prompt


As this file might be outdated, make sure the version here corresponds to the version listed in the `rust-toolchain` file of this repo.

```
source /root/.profile
rustup install 1.60.0
rustup default 1.60.0
```

### Build playground image

```
git clone https://github.com/ponylang/pony-playground.git
cd pony-playground
docker build docker --pull -t ponylang-playpen
```

### Set up gist access

Create a personal access token with gist access.
install in GITHUB_TOKEN environment variable e.g. to `$HOME/.profile` for manual testing.

Should ONLY be the token, not "user:token"

### Build it

```
cargo build --release --bin playpen
```

### Create Systemd Unit

Put the following in the file `/etc/systemd/system/playground.service`, put in the generated GITHUB_TOKEN from above:

```
[Unit]
Description=Pony Playground Systemd Service Unit
Requires=docker.service
After=network.target

[Service]
Environment="GITHUB_TOKEN=..."
Environment="RUST_LOG=debug"
ExecStart=/root/pony-playground/target/release/playpen

[Install]
WantedBy=multi-user.target
```

### Enable and Run it

```
systemctl enable playground
systemctl start playground
```

STDOUT and STDERR both go the journal. If you want to investigate logs, use:

```
journalctl -u playground ...
```

