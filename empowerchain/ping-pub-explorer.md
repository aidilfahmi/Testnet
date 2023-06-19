# How to Create Explorer
## Installing
```
sudo apt autoremove nodejs -y
curl -fsSL https://deb.nodesource.com/setup_19.x | sudo -E bash -
curl -sL https://dl.yarnpkg.com/debian/pubkey.gpg | gpg --dearmor | sudo tee /usr/share/keyrings/yarnkey.gpg >/dev/null
echo "deb [signed-by=/usr/share/keyrings/yarnkey.gpg] https://dl.yarnpkg.com/debian stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
sudo apt update
sudo apt install nginx certbot python3-certbot-nginx nodejs git yarn -y 
```
## NGINX Configuration

### File Configuration
Create explorer file configuration in Nginx configuration folder
```
nano /etc/nginx/sites-enabled/your_explorer_server.conf
```
Replace `your_explorer_server` with your own server, ex.
```
nano /etc/nginx/sites-enabled/explorer.dnsarz.xyz.conf
```
Create this sample configuration
```
server {
    listen       80;
    listen  [::]:80;
    server_name explorer.dnsarz.xyz;

    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root /usr/share/nginx/html;
        index  index.html index.htm;
        try_files $uri $uri/ /index.html;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    gzip on;
    gzip_proxied any;
    gzip_static on;
    gzip_min_length 1024;
    gzip_buffers 4 16k;
    gzip_comp_level 2;
    gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php application/vnd.ms-fontobject font/ttf font/opentype font/x-woff image/svg+xml;
    gzip_vary off;
    gzip_disable "MSIE [1-6]\.";
}
```
replace `server_name` with your own server.<br>
### SSL Configuration
```diff
- I will Update Later
```
Restart your nginx.
```
systemctl restart nginx
```


## Explorer Configuration
### Clone Repository
```
cd $HOME
git clone https://github.com/ping-pub/explorer
```

### Create or edit your config file
There are so many configurations on ``$HOME/explorer/chains/mainnet``. You can delete files that you don't need or you can also create new files with your own configuration. Here's for example.
```
nano $HOME/explorer/chains/mainnet/empower.json
```

Here's my configuration for example

```
{
    "chain_name": "empower",
    "api": ["https://empower-testnet-api.polkachu.com"],
    "rpc": ["https://empower-testnet-rpc.polkachu.com"],
    "coingecko": "",
    "snapshot_provider": "",
    "sdk_version": "0.47.1",
    "coin_type": "118",
    "min_tx_fee": "500",
    "addr_prefix": "empower",
    "logo": "https://explorer.nodexcapital.com/logos/empower.png",
    "assets": [{
        "base": "umpwr",
        "symbol": "MPWR",
        "exponent": "6",
        "coingecko_id": "",
        "logo": "https://explorer.nodexcapital.com/logos/empower.png"
    }]
  }
```
### Build the Explorer
```
cd $HOME/explorer
yarn && yarn build
```
if you have an error when build like this ``yarn The engine "node" is incompatible with this module``, use this command
```
yarn install --ignore-engines
cd $HOME/explorer
yarn && yarn build
```
### Copy web file to Nginx html folder
```
cp -r $HOME/explorer/dist/* /usr/share/nginx/html
nginx -s reload
```

