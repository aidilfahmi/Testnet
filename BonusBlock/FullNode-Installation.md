# $${\color{lightgreen}Manual \space Node \space Installation}$$

## ${\color{lightblue}Optional \space Configuration}$
### Custom User

```
sudo adduser bonus
sudo adduser bonus sudo
su - bonus
```
### Custom Port
```diff
- Skip this step if running default port configurations
```
```
PORT=58
echo "export PORT=${PORT}" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
 
## ${\color{orange}Preparing}$	
### Server Update
```
sudo apt -q update
sudo apt -qy install curl git jq lz4 build-essential
sudo apt -qy upgrade
```

### Install GO
```
sudo rm -rf /usr/local/go && \
curl -Ls https://go.dev/dl/go1.19.5.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version


```
## ${\color{orange}Node Installation}$	
### Get Binary
```
cd $HOME
git clone https://github.com/BBlockLabs/BonusBlock-chain
cd BonusBlock-chain
make build

```

#### Install Cosmovisor
```
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.4.0
mkdir -p $HOME/.bonusblock/cosmovisor/genesis/bin
cp $HOME/go/bin/bonus-blockd $HOME/.bonusblock/cosmovisor/genesis/bin
ln -s $HOME/.bonusblock/cosmovisor/genesis $HOME/.bonusblock/cosmovisor/current
sudo ln -s $HOME/.bonusblock/cosmovisor/current/bin/bonus-blockd /usr/bin/bonus-blockd
```

### Init Generation

Replace `moniker_name` with your own moniker name
```
bonus-blockd config chain-id blocktopia-01
bonus-blockd config keyring-backend test
bonus-blockd init moniker_name --chain-id blocktopia-01
```

### Set Peers and Seeds
```
PEERS=e5e04918240cfe63e20059a8abcbe62f7eb05036@bonusblock-testnet-p2p.alter.network:26656
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.bonusblock/config/config.toml
```

### Download Genesis and Addrbook
```
curl https://bonusblock-testnet.alter.network/genesis? | jq '.result.genesis' > ~/.bonusblock/config/genesis.json
```

### Set Port app.toml and config.toml
```diff
- This Step If you running custom port, if you running default port, skip this step
```
```
bonus-blockd config node tcp://localhost:${PORT}657
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${PORT}660\"%" $HOME/.bonusblock/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${PORT}317\"%; s%^address = \":8080\"%address = \":${PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${PORT}091\"%" $HOME/.bonusblock/config/app.toml
```

### Set Config Pruning
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.bonusblock/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.bonusblock/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.bonusblock/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.bonusblock/config/app.toml
```

### Set minimum gas price
```
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0025ubonus\"/" $HOME/.bonusblock/config/app.toml
```
### Disable Indexer
```
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.bonusblock/config/config.toml
```
### Create Service

```
sudo tee /etc/systemd/system/bonus-blockd.service > /dev/null << EOF
[Unit]
Description=BonusBlock Testnet
After=network-online.target
[Service]
User=$USER
ExecStart=$HOME/go/bin/cosmovisor run start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.bonusblock"
Environment="DAEMON_NAME=bonus-blockd"
Environment="UNSAFE_SKIP_BACKUP=true"
[Install]
WantedBy=multi-user.target

EOF
```

### Register And Start Service
```
sudo systemctl enable bonus-blockd
sudo systemctl daemon-reload
sudo systemctl start bonus-blockd && sudo journalctl -fu bonus-blockd -o cat
```

## ${\color{orange}State sync}$	

### Stop the service and reset the data
```
sudo systemctl stop bonus-blockd
cp $HOME/.bonusblock/data/priv_validator_state.json $HOME/.bonusblock/priv_validator_state.json.backup
bonus-blockd tendermint unsafe-reset-all --home $HOME/.bonusblock --keep-addr-book
```

### Configure state sync information
```
SNAP_RPC="https://rpc-bonusblock.sxlzptprjkt.xyz:443"
STATESYNC_PEERS="d7f5483ba2d290a3853b4db5df21579e9c8da69a@peers-bonusblock.sxlzptprjkt.xyz:30656"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.bonusblock/config/config.toml
sed -i -e "s|^persistent_peers *=.*|persistent_peers = \"$STATESYNC_PEERS\"|" $HOME/.bonusblock/config/config.toml

mv $HOME/.bonusblock/priv_validator_state.json.backup $HOME/.bonusblock/data/priv_validator_state.json
```
### Restart Node and Check the logs
```
sudo systemctl start bonus-blockd && sudo journalctl -fu bonus-blockd -o cat
```

## Deleting Node
```
sudo systemctl stop bonus-blockd && \
sudo systemctl disable bonus-blockd && \
rm /etc/systemd/system/bonus-blockd.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf .bonusblock && \
rm -rf $(which bonus-blockd)
```
