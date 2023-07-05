# $${\color{lightgreen}CLI \space Node \space Installation}$$

## ${\color{lightblue}Optional}$
### ${\color{orange}Custom-User}$
```javascript
sudo adduser pactus
sudo adduser pactus sudo
su - pactus
```
## ${\color{lightblue}Preparing}$
### ${\color{orange}Update}$
```javascript
sudo apt update && sudo apt upgrade 
sudo apt install curl
```
## ${\color{lightblue}Setup \space Node}$ 
### ${\color{orange}Install-Binary}$
```javascript
cd $HOME
rm -fr cli
curl --proto '=https' --tlsv1.2 -sSL  https://github.com/pactus-project/pactus/releases/download/v0.13.0/pactus_downloader.sh | sh
mv pactus-cli* cli
echo "export PATH=$PATH:$HOME/cli" >> $HOME/.bash_profile && source $HOME/.bash_profile
```
### ${\color{orange}Create \space or \space Restore \space Wallet}$
#### Create
```javascript
pactus-daemon init -w ~/pactus --testnet
```
#### Restore
```javascript
pactus-daemon init -w ~/pactus --testnet --restore "your wallet phase"
```
### If installation asking you for wallet password, `just press enter to skip it`
### ${\color{orange}Create \space Service}$
```javascript
sudo tee /etc/systemd/system/pactus.service > /dev/null <<EOF
[Unit]
Description=Pactus Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which pactus-daemon) start -w pactus
WorkingDirectory=$HOME
Restart=on-failure
RestartSec=3
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```
### ${\color{orange}Apply \space Service}$
```javascript
sudo systemctl enable pactus
sudo systemctl daemon-reload
sudo systemctl start pactus && sudo journalctl -fu pactus -o cat
```

## ${\color{lightblue}Custom \space Port}$ 
If you get `conflict port`, try this methods
```
nano $HOME/pactus/config.toml
```
Replace this port with your available port

![image](https://github.com/aidilfahmi/Testnet/assets/16186519/875ec148-e3d1-4c98-a63e-dec1240f2eab)
