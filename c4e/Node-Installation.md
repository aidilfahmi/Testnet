# $${\color{lightgreen}Manual \space Node \space Installation}$$

## ${\color{lightblue}Optional \space Configuration}$
### Custom User

```javascript
sudo adduser c4e
sudo adduser c4e sudo
su - c4e
```
### Custom Port
```diff
- Skip this step if running default port configurations
```
```
PORT=51
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
curl -LO https://github.com/chain4energy/c4e-chain/releases/download/v1.2.0/c4ed_v1.2.0_linux_amd64.tar.gz
tar -xvf c4ed_v1.2.0_linux_amd64.tar.gz

```

#### Install Cosmovisor
```javascript
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.4.0
mkdir -p $HOME/.c4e-chain/cosmovisor/genesis/bin
mv $HOME/c4ed $HOME/.c4e-chain/cosmovisor/genesis/bin
ln -s $HOME/.c4e-chain/cosmovisor/genesis $HOME/.c4e-chain/cosmovisor/current
sudo ln -s $HOME/.c4e-chain/cosmovisor/current/bin/c4ed /usr/bin/c4ed
c4ed version
```
ver = 1.2.0


### Init Generation

Replace `moniker_name` with your own moniker name
```javascript
c4ed config chain-id babajaga-1
c4ed config keyring-backend test
c4ed init moniker_name --chain-id babajaga-1
```

### Set Port app.toml and config.toml
```diff
- This Step If you running custom port, if you running default port, skip this step
```
```javascript
c4ed config node tcp://localhost:${PORT}657
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${PORT}660\"%" $HOME/.c4e-chain/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${PORT}317\"%; s%^address = \":8080\"%address = \":${PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${PORT}091\"%" $HOME/.c4e-chain/config/app.toml
```

### Set Peers and Seeds
```javascript
peers="de18fc6b4a5a76bd30f65ebb28f880095b5dd58b@66.70.177.76:36656,33f90a0ac7e8f48305ea7e64610b789bbbb33224@151.80.19.186:36656"
sed -i -e "s|^persistent_peers *=.*|persistent_peers = \"$peers\"|" $HOME/.ojo/config/config.toml
```

### Download Genesis and Addrbook
```javascript
curl -Ls https://raw.githubusercontent.com/chain4energy/c4e-chains/main/babajaga-1/genesis.json > $HOME/.c4e-chain/config/genesis.json
```

### Set Config Pruning
```javascript

```

### Set minimum gas price
```javascript

```

### Disable indexer
```javascript
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.c4e-chain/config/config.toml
```

### Create Service

```javascript
sudo tee /etc/systemd/system/c4ed.service > /dev/null << EOF
[Unit]
Description=C4E Testnet
After=network-online.target
[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
WorkingDirectory=$HOME
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.c4e-chain"
Environment="DAEMON_NAME=c4ed"
Environment="UNSAFE_SKIP_BACKUP=true"
[Install]
WantedBy=multi-user.target
EOF
```

### Register Service
```javascript
sudo systemctl enable c4ed
sudo systemctl daemon-reload
```

### Sync with state-sync
```javascript
cp $HOME/.c4e-chain/data/priv_validator_state.json $HOME/.c4e-chain/priv_validator_state.json.backup
blockxd tendermint unsafe-reset-all --home $HOME/.c4e-chain --keep-addr-book
```
```javascript
SNAP_RPC1="https://rpc-testnet.c4e.io:443"
SNAP_RPC2="https://rpc-testnet.c4e.io:443"
```
```javascript
LATEST_HEIGHT=$(curl -s https://rpc-testnet.c4e.io:443/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); \
TRUST_HASH=$(curl -s "https://rpc-testnet.c4e.io:443/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)
```
```javascript
sudo systemctl stop c4ed
```
```javascript
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC1,$SNAP_RPC2\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.c4e-chain/config/config.toml

sudo systemctl start c4ed && sudo journalctl -fu c4ed -o cat
```

### Disable State Sync
After successful synchronization using state sync above, we advise you to disable synchronization with state sync and restart the node
```javascript
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1false|" $HOME/.c4e-chain/config/config.toml
sudo systemctl start c4ed && sudo journalctl -fu c4ed -o cat
```
