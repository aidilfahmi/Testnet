# $${\color{lightgreen}Manual \space Node \space Installation}$$

## ${\color{lightblue}Optional \space Configuration}$
### Custom User

```javascript
sudo adduser empower
sudo adduser empower sudo
su - empower
```
### Custom Port
```diff
- Skip this step if running default port configurations
```
```
PORT=44
echo "export PORT=${PORT}" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
 
## ${\color{orange}Preparing}$	
### Server Update
```javascript
sudo apt -q update
sudo apt -qy install curl git jq lz4 build-essential
sudo apt -qy upgrade
```

### Install GO
```javascript
sudo rm -rf /usr/local/go && \
curl -Ls https://go.dev/dl/go1.20.5.src.tar.gz | sudo tar -xzf - -C /usr/local && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version


```
## ${\color{orange}Node Installation}$	
### Get Binary
```javascript
cd $HOME
wget https://github.com/EmpowerPlastic/empowerchain/releases/download/v1.0.0-rc3/empowerd-v1.0.0-rc3-linux-amd64.zip
unzip empowerd-v1.0.0-rc3-linux-amd64.zip
./empowerd  version
```

#### Install Cosmovisor
```javascript
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.4.0
mkdir -p $HOME/.empowerchain/cosmovisor/genesis/bin
mv empowerd $HOME/.empowerchain/cosmovisor/genesis/bin
ln -s $HOME/.empowerchain/cosmovisor/genesis $HOME/.empowerchain/cosmovisor/current
sudo ln -s $HOME/.empowerchain/cosmovisor/current/bin/empowerd /usr/bin/empowerd
```

### Init Generation

Replace `moniker_name` with your own moniker name
```javascript
empowerd config chain-id circulus-1
empowerd config keyring-backend test
empowerd init moniker_name --chain-id circulus-1
```

### Set Port app.toml and config.toml
```diff
- This Step If you running custom port, if you running default port, skip this step
```
```javascript
empowerd config node tcp://localhost:${PORT}657
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${PORT}660\"%" $HOME/.empowerchain/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${PORT}317\"%; s%^address = \":8080\"%address = \":${PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${PORT}091\"%" $HOME/.empowerchain/config/app.toml
```

### Set Peers and Seeds
```javascript
SEEDS="258f523c96efde50d5fe0a9faeea8a3e83be22ca@seed.circulus-1.empower.aviaone.com:20272,d6a7cd9fa2bafc0087cb606de1d6d71216695c25@51.159.161.174:26656,babc3f3f7804933265ec9c40ad94f4da8e9e0017@testnet-seed.rhinostake.com:17456"
PEERS=""
sed -i 's|^seeds *=.*|seeds = "'$SEEDS'"|; s|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.empowerchain/config/config.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\" /" $HOME/.empowerchain/config/config.toml
```

### Download Genesis and Addrbook
```javascript
curl -Ls https://raw.githubusercontent.com/aidilfahmi/Testnet/main/empowerchain/genesis.json > $HOME/.empowerchain/config/genesis.json
curl -Ls https://raw.githubusercontent.com/aidilfahmi/Testnet/main/empowerchain/addrbook.json > $HOME/.empowerchain/config/addrbook.json
```

### Set Config Pruning
```javascript
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="19"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.empowerchain/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.empowerchain/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.empowerchain/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.empowerchain/config/app.toml
```

### Set minimum gas price
```javascript
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.025umpwr\"/" $HOME/.empowerchain/config/app.toml
```

### Disable indexer
```javascript
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.empowerchain/config/config.toml
```

### Create Service

```javascript
sudo tee /etc/systemd/system/empowerd.service > /dev/null << EOF
[Unit]
Description=Empowerchain Testnet
After=network-online.target
[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
WorkingDirectory=$HOME
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.empowerchain"
Environment="DAEMON_NAME=empowerd"
Environment="UNSAFE_SKIP_BACKUP=true"
[Install]
WantedBy=multi-user.target
EOF
```

### Register And Start Service
```javascript
sudo systemctl enable empowerd
sudo systemctl daemon-reload
sudo systemctl start empowerd && sudo journalctl -fu empowerd -o cat
