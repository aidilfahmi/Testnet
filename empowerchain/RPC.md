# Guide Create RPC API Node Cosmos


## Prepare : 
Make sure you already have your domain and create some sub-domain, here is my domain list:

> Domain : dnsarz.xyz <p>
>Subdomain: <br>
API   : api-empower.dnsarz.xyz <br>
RPC   : rpc-empower.dnsarz.xyz <br>
gRPC  : grpc-empower.dnsarz.xyz <br>
Explorer: https://explorer.dnsarz.xyz/empower

## Port and Address configuration
### Setting API in app.toml
```bash
nano $HOME/.empowerchain/config/app.toml
```
![image](https://github.com/aidilfahmi/Testnet/assets/16186519/0686dce5-bbcf-4443-84be-e33091e4ec0c)


> 1. Enable API status. <br>
> 2. Save address and port number. ex. tcp://0.0.0.0:`44317`<br>
>

### Check RPC IP:PORT in config.toml

```
 nano $HOME/.empowerchain/config/config.toml
 ```
![image](https://github.com/aidilfahmi/Testnet/assets/16186519/3f8ed5c9-3127-49cd-9a85-cfe9ac6fa56d)

<p>
	
So now we have list Configuration RPC and API like this:
> API : 0.0.0.0:44317 <br>
	RPC : 127.0.0.1:44657 <br> <br>
  all port will be diffrent for each nodes, its depend on your settings
	
## Install Package

```
sudo apt update && sudo apt upgrade -y
```
```
sudo apt install nginx certbot python3-certbot-nginx -y
```
```
curl -fsSL https://deb.nodesource.com/setup_19.x | sudo -E bash -
```
```
sudo apt-get update && apt install -y nodejs git
```
```
curl -sL https://dl.yarnpkg.com/debian/pubkey.gpg | gpg --dearmor | sudo tee /usr/share/keyrings/yarnkey.gpg >/dev/null
```
```
echo "deb [signed-by=/usr/share/keyrings/yarnkey.gpg] https://dl.yarnpkg.com/debian stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
```
```
sudo apt-get update && sudo apt-get install yarn -y
```

## Setting Proxy nginx and Install SSL


## NGINX Configuration
> After create some subdomain, now you have to create nginx configuration for each sub-domain. <br>
All NGINX configuration can be found and create at `/etc/nginx/sites-enabled/` folder

### Create API Config
create new config file for api-empower.dnsarz.xyz sub-domain
```
nano /etc/nginx/sites-enabled/api-empower.dnsarz.xyz.conf
```
copy paste this line
```
server {
    server_name api-empower.dnsarz.xyz;
    listen 80;
    location / {
        add_header Access-Control-Allow-Origin *;
        add_header Access-Control-Max-Age 3600;
        add_header Access-Control-Expose-Headers Content-Length;

	proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        proxy_set_header   Host             $host;

        proxy_pass http://0.0.0.0:44317;

    }
}
```
`ctrl + X + Y `to save it
### Create RPC Config
create new config file for rpc-empower.dnsarz.xyz sub-domain
```
nano /etc/nginx/sites-enabled/rpc-empower.dnsarz.xyz.conf
```
copy paste this line
```
server {
    server_name rpc-empower.dnsarz.xyz;
    listen 80;
    location / {
        add_header Access-Control-Allow-Origin *;
        add_header Access-Control-Max-Age 3600;
        add_header Access-Control-Expose-Headers Content-Length;

	proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        proxy_set_header   Host             $host;

        proxy_pass  http://127.0.0.1:44657;

    }
}
```

### Config Test

```bash
nginx -t 
```
should be like this

![image](https://github.com/aidilfahmi/Testnet/assets/16186519/85e42210-4dec-42a2-b599-dec88e9a95fc)


### Install Certificate SSL

```bash
sudo certbot --nginx --register-unsafely-without-email
```
![image](https://github.com/aidilfahmi/Testnet/assets/16186519/c0695a95-b6e7-4b33-8e4d-d24d81613ed2)


> Select 1 and press enter <br>
  if the BOT asking for 'redirect', select YES.<br>
  Do the option for all your sub-domains.

<p>

If BOT doesn't asking for redirect your sub-domain, you can use this command:
```
  sudo certbot --nginx --redirect
```

After all done, you can restart NGINX 
```
systemctl restart nginx
```
  
## Checking your API and RPC
Open your browser and check your API sub-domain
> https://api-empower.dnsarz.xyz
  
![image](https://user-images.githubusercontent.com/16186519/215946801-17929a44-0f03-40bc-b80f-a996bf66e235.png)

Open your browser and check your RPC sub-domain
> https://rpc-empower.dnsarz.xyz

![image](https://user-images.githubusercontent.com/16186519/215947098-0c58247b-d879-421c-90ce-172acdaa687d.png)

  
IF everything showing on your browser like that images, then well done, you already create your own API and RPC.
  
Thank you.
