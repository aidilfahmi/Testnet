# $${\color{lightgreen}Manual \space Node \space Installation}$$

## ${\color{lightblue}Optional \space Configuration}$
### Custom User

```javascript
sudo adduser composable
sudo adduser composable sudo
su - composable
```
### Custom Port
```diff
- Skip this step if running default port configurations
```
```
PORT=50
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
curl -Ls https://go.dev/dl/go1.19.5.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version


```
## ${\color{orange}Node Installation}$	
### Get Binary
```javascript
cd $HOME
rm -rf composable-testnet
git clone https://github.com/notional-labs/composable-testnet.git
cd composable-testnet
git checkout v2.3.2
make build
```

#### Install Cosmovisor
```javascript
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.4.0
mkdir -p $HOME/.banksy/cosmovisor/genesis/bin
mv $HOME/composable-testnet/bin/banksyd $HOME/.banksy/cosmovisor/genesis/bin
ln -s $HOME/.banksy/cosmovisor/genesis $HOME/.banksy/cosmovisor/current
sudo ln -s $HOME/.banksy/cosmovisor/current/bin/banksyd /usr/bin/banksyd
```

### Init Generation

Replace `moniker_name` with your own moniker name
```javascript
banksyd config chain-id banksy-testnet-2
banksyd config keyring-backend test
banksyd init moniker_name --chain-id banksy-testnet-2
```

### Set Port app.toml and config.toml
```diff
- This Step If you running custom port, if you running default port, skip this step
```
```javascript
banksyd config node tcp://localhost:${PORT}657
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${PORT}660\"%" $HOME/.banksy/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${PORT}317\"%; s%^address = \":8080\"%address = \":${PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${PORT}091\"%" $HOME/.banksy/config/app.toml
```

### Set Peers and Seeds
```javascript
PEERS="$(curl -sS https://rpc.composable-t.nodexcapital.com/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}' | sed -z 's|\n|,|g;s|.$||')"
SEEDS="872c8a78a17a24d6f44e1126c46ef52069c7bb18@65.109.80.150:2630,5c2a752c9b1952dbed075c56c600c3a79b58c395@composable-testnet-seed.autostake.com:26976,20e1000e88125698264454a884812746c2eb4807@seeds.lavenderfive.com:22256,3f472746f46493309650e5a033076689996c8881@composable-testnet.rpc.kjnodes.com:15959,ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@testnet-seeds.polkachu.com:22256,945e8384ea51c5c6f7b9a90df8d8da120516d897@rpc.composable-t.indonode.net:47656"
sed -i -e "s|^persistent_peers *=.*|persistent_peers = \"$PEERS\"|" $HOME/.banksy/config/config.toml
sed -i.bak -e "s/^seeds =.*/seeds = \"$SEEDS\"/" $HOME/.banksy/config/config.toml
```

### Download Genesis and Addrbook
```javascript
curl -Ls https://raw.githubusercontent.com/notional-labs/composable-networks/main/testnet-2/genesis.json > $HOME/.banksy/config/genesis.json
curl -Ls https://snap.nodexcapital.com/composable/addrbook.json > $HOME/.banksy/config/addrbook.json

```

### Set Config Pruning
```javascript
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="19"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.banksy/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.banksy/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.banksy/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.banksy/config/app.toml
```

### Set minimum gas price
```javascript
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0upica\"|" $HOME/.banksy/config/app.toml
```

### Disable indexer
```javascript
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.banksy/config/config.toml
```

### Create Service

```javascript
sudo tee /etc/systemd/system/banksyd.service > /dev/null << EOF
[Unit]
Description=Composable-testnet Testnet
After=network-online.target
[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
WorkingDirectory=$HOME
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.banksy"
Environment="DAEMON_NAME=banksyd"
Environment="UNSAFE_SKIP_BACKUP=true"
[Install]
WantedBy=multi-user.target
EOF
```

### Register And Start Service
```javascript
sudo systemctl enable banksyd
sudo systemctl daemon-reload
sudo systemctl start banksyd && sudo journalctl -fu banksyd -o cat
