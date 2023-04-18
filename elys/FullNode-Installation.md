# $${\color{lightgreen}Manual \space Node \space Installation}$$

## ${\color{lightblue}Optional \space Configuration}$
### Custom User

```javascript
sudo adduser elys
sudo adduser elys sudo
su - elys
```
### Custom Port
```diff
- Skip this step if running default port configurations
```
```
PORT=54
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
rm -rf elys
git clone https://github.com/elys-network/elys.git
cd elys
git checkout v0.2.3
```

#### Install Cosmovisor
```javascript
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.4.0
mkdir -p $HOME/.elys/cosmovisor/genesis/bin
mv $HOME/elys/build/elysd $HOME/.elys/cosmovisor/genesis/bin
ln -s $HOME/.elys/cosmovisor/genesis $HOME/.elys/cosmovisor/current
sudo ln -s $HOME/.elys/cosmovisor/current/bin/elysd /usr/bin/elysd
```

### Init Generation

Replace `moniker_name` with your own moniker name
```javascript
elysd config chain-id elystestnet-1
elysd config keyring-backend test
elysd init moniker_name --chain-id elystestnet-1
```

### Set Peers and Seeds
```javascript
PEERS="$(curl -sS https://rpc.elys-t.indonode.net/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}' | sed -z 's|\n|,|g;s|.$||')"
sed -i -e "s|^persistent_peers *=.*|persistent_peers = \"$PEERS\"|" $HOME/.elys/config/config.toml
sed -i 's/seeds = ""/seeds = "ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@testnet-seeds.polkachu.com:22056"/' ~/.elys/config/config.toml
```

### Download Genesis and Addrbook
```javascript
curl -Ls https://raw.githubusercontent.com/elys-network/elys/main/chain/genesis.json > $HOME/.elys/config/genesis.json
curl -Ls https://snapshots.indonode.net/elys-t/addrbook.json > $HOME/.elys/config/addrbook.json
```

### Set Port app.toml and config.toml
```diff
- This Step If you running custom port, if you running default port, skip this step
```
```javascript
elysd config node tcp://localhost:${PORT}657
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${PORT}660\"%" $HOME/.elys/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${PORT}317\"%; s%^address = \":8080\"%address = \":${PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${PORT}091\"%" $HOME/.elys/config/app.toml
```

### Set Config Pruning
```javascript
pruning="nothing"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="19"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.elys/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.elys/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.elys/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.elys/config/app.toml
```

### Set minimum gas price
```javascript
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0uelys\"|" $HOME/.elys/config/app.toml
```

### Create Service

```javascript
sudo tee /etc/systemd/system/elysd.service > /dev/null << EOF
[Unit]
Description=ELYS Testnet
After=network-online.target
[Service]
User=$USER
ExecStart=$HOME/go/bin/cosmovisor run start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.elys"
Environment="DAEMON_NAME=elysd"
Environment="UNSAFE_SKIP_BACKUP=true"
[Install]
WantedBy=multi-user.target

EOF
```

### Register And Start Service
```javascript
sudo systemctl enable elysd
sudo systemctl daemon-reload
sudo systemctl start elysd && sudo journalctl -fu elysd -o cat
