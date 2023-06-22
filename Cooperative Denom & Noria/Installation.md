# $${\color{lightgreen}Manual \space Node \space Installation}$$

## ${\color{lightblue}Optional \space Configuration}$
### Custom User

```javascript
sudo adduser blockx
sudo adduser blockx sudo
su - blockx
```
### Custom Port
```diff
- Skip this step if running default port configurations
```
```
PORT=46
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
rm -rf noria
git clone https://github.com/noria-net/noria.git
cd noria
git checkout v1.3.0

# Build binaries
make build
```

#### Install Cosmovisor
```javascript
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.4.0
mkdir -p $HOME/.noria/cosmovisor/genesis/bin
mv build/noriad $HOME/.noria/cosmovisor/genesis/bin
ln -s $HOME/.noria/cosmovisor/genesis $HOME/.noria/cosmovisor/current
sudo ln -s $HOME/.noria/cosmovisor/current/bin/noriad /usr/bin/noriad
rm -fr build
```

### Init Generation

Replace `moniker_name` with your own moniker name
```javascript
noriad config chain-id oasis-3
noriad config keyring-backend test
noriad init moniker_name --chain-id oasis-3
```

### Set Port app.toml and config.toml
```diff
- This Step If you running custom port, if you running default port, skip this step
```
```javascript
noriad config node tcp://localhost:${PORT}657
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${PORT}660\"%" $HOME/.noria/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${PORT}317\"%; s%^address = \":8080\"%address = \":${PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${PORT}091\"%" $HOME/.noria/config/app.toml
```

### Set Peers and Seeds
```javascript
sed -i -e "s|^seeds *=.*|seeds = \"3f472746f46493309650e5a033076689996c8881@noria-testnet.rpc.kjnodes.com:16159\"|" $HOME/.noria/config/config.toml
```

### Download Genesis and Addrbook
```javascript
curl -Ls https://snapshots.kjnodes.com/noria-testnet/genesis.json > $HOME/.noria/config/genesis.json
curl -Ls https://snapshots.kjnodes.com/noria-testnet/addrbook.json > $HOME/.noria/config/addrbook.json
```

### Set Config Pruning
```javascript
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="19"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.noria/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.noria/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.noria/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.noria/config/app.toml
```

### Set minimum gas price
```javascript
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.0025ucrd\"|" $HOME/.noria/config/app.toml
```

### Disable indexer
```javascript
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.noria/config/config.toml
```

### Create Service

```javascript
sudo tee /etc/systemd/system/noriad.service > /dev/null << EOF
[Unit]
Description=Noria Testnet
After=network-online.target
[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
WorkingDirectory=$HOME
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.noria"
Environment="DAEMON_NAME=noriad"
Environment="UNSAFE_SKIP_BACKUP=true"
[Install]
WantedBy=multi-user.target
EOF
```

### Register And Start Service
```javascript
sudo systemctl enable noriad
sudo systemctl daemon-reload
sudo systemctl restart noriad && sudo journalctl -fu noriad -o cat
