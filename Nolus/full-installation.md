
<h2>MANUAL INSTALLATION</h2>

### Custom User (OPTIONAL)

```
sudo adduser nolus
sudo adduser nolus sudo
su - nolus
```
### ** Using Custom Port (OPTIONAL)
```diff
- Skip this step if running default port configurations
```
```
PORT=55
echo "export PORT=${PORT}" >> $HOME/.bash_profile
source $HOME/.bash_profile
```


## Preparing
```
sudo apt -q update
sudo apt -qy install curl git jq lz4 build-essential
sudo apt -qy upgrade
```

## Install GO
```
sudo rm -rf /usr/local/go && \
curl -Ls https://go.dev/dl/go1.19.5.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```

## Get Binary
```
cd $HOME
curl -LOf https://github.com/Nolus-Protocol/nolus-core/releases/download/v0.2.1-testnet/nolus.tar.gz
tar -zxvf nolus.tar.gz
```

### Install Cosmovisor
```
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.4.0
mkdir -p $HOME/.nolus/cosmovisor/genesis/bin
mv $HOME/nolusd $HOME/.nolus/cosmovisor/genesis/bin
ln -s $HOME/.nolus/cosmovisor/genesis $HOME/.nolus/cosmovisor/current
sudo ln -s $HOME/.nolus/cosmovisor/current/bin/nolusd /usr/bin/nolusd
```

## Init Generation

Replace `moniker_name` with your own moniker name
```
nolusd config chain-id nolus-rila
nolusd config keyring-backend test
nolusd init moniker_name --chain-id nolus-rila
```

## Set Peers and Seeds
```
PEERS=
SEEDS="3f472746f46493309650e5a033076689996c8881@nolus-testnet.rpc.kjnodes.com:43659"
sed -i -e "s|^persistent_peers *=.*|persistent_peers = \"$PEERS\"|" $HOME/$FOLDER/config/config.toml
sed -i -e "s|^seeds *=.*|seeds = \"$SEEDS\"|" $HOME/$FOLDER/config/config.toml
```

## Download Genesis and Addrbook
```
curl -Ls https://snapshots.kjnodes.com/nolus-testnet/genesis.json > $HOME/.nolus/config/genesis.json
curl -Ls https://snapshots.kjnodes.com/nolus-testnet/addrbook.json > $HOME/.nolus/config/addrbook.json
```

## Set Port app.toml and config.toml
```diff
- This Step If you running custom port, if you running default port, skip this step
```
```
nolusd config node tcp://localhost:${PORT}657
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${PORT}660\"%" $HOME/.nolus/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${PORT}317\"%; s%^address = \":8080\"%address = \":${PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${PORT}091\"%" $HOME/.nolus/config/app.toml
```

## Set Config Pruning
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.nolus/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.nolus/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.nolus/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.nolus/config/app.toml
```

## Set minimum gas price
```
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0unolus\"|" $HOME/.nolus/config/app.toml
```

## Create Service

```
sudo tee /etc/systemd/system/nolusd.service > /dev/null << EOF
[Unit]
Description=NOLUS Testnet
After=network-online.target
[Service]
User=$USER
ExecStart=$HOME/go/bin/cosmovisor run start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.nolus"
Environment="DAEMON_NAME=nolusd"
Environment="UNSAFE_SKIP_BACKUP=true"
[Install]
WantedBy=multi-user.target

EOF
```

## Register And Start Service
```
sudo systemctl enable nolusd
sudo systemctl daemon-reload
sudo systemctl start nolusd && sudo journalctl -fu nolusd -o cat
```


## Snapshots
### Stop Service and backup priv_validator
```
sudo systemctl stop nolusd
cp $HOME/.nolus/data/priv_validator_state.json $HOME/.nolus/priv_validator_state.json.backup
rm -rf $HOME/.nolus/data
```

### Download Snapshot
```
curl -L https://snapshots.kjnodes.com/nolus-testnet/snapshot_latest.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.nolus
mv $HOME/.nolus/priv_validator_state.json.backup $HOME/.nolus/data/priv_validator_state.json
```
### Restart Node and Check the logs
```
sudo systemctl restart nolusd && sudo journalctl -u nolusd -f --no-hostname -o cat
```

## Deleting Node
```
sudo systemctl stop nolusd && \
sudo systemctl disable nolusd && \
rm /etc/systemd/system/nolusd.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf .nolus && \
rm -rf $(which nolusd)
```
