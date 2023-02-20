<h2>MANUAL INSTALLATION</h2>

## Optional

#### Custom user

```
sudo adduser andromeda
sudo adduser andromeda sudo
su - andromeda
```
#### Using Custom Port
```
PORT=54
echo "export PORT=${PORT}" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
## Preparing
```
sudo apt -q update
sudo apt -qy install curl git jq lz4 build-essential wget make gcc tmux chrony -y
sudo apt -qy upgrade
```

## Install GO
```
ver="1.19" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```

## Get Binary
```
cd $HOME && rm -rf andromedad
git clone https://github.com/andromedaprotocol/andromedad.git
cd andromedad
git checkout galileo-3-v1.1.0-beta1
make install
```

<h3> Optional if you running as cosmovisor </h3>

```
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.4.0

mkdir -p $HOME/.andromedad/cosmovisor/genesis/bin
cp $HOME/go/bin/andromedad $HOME/.andromedad/cosmovisor/genesis/bin

ln -s $HOME/.andromedad/cosmovisor/genesis $HOME/.core/coreum-testnet-1/cosmovisor/current
sudo ln -s $HOME/.andromedad/cosmovisor/current/bin/andromedad /usr/bin/andromedad
```

## Init generation

Replace `moniker_name` with your own moniker name
```
andromedad config chain-id galileo-3
andromedad config keyring-backend test
andromedad init moniker_name --chain-id galileo-3
```

## Set peers and seeds
```
PEERS=06d4ab2369406136c00a839efc30ea5df9acaf11@10.128.0.44:26656,43d667323445c8f4d450d5d5352f499fa04839a8@192.168.0.237:26656,29a9c5bfb54343d25c89d7119fade8b18201c503@192.168.101.79:26656,6006190d5a3a9686bbcce26abc79c7f3f868f43a@37.252.184.230:26656
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.andromedad/config/config.toml
```

## Download genesis and addrbook
```

```

## Set Port app.toml and config.toml
```diff
- This Step If you running custom port, if you running default port, skip this step
```
```
andromedad config node tcp://localhost:${PORT}657
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${PORT}660\"%" $HOME/.core/coreum-testnet-1/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${PORT}317\"%; s%^address = \":8080\"%address = \":${PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${PORT}091\"%" $HOME/.core/coreum-testnet-1/config/app.toml

```

## Set Config Pruning
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.andromedad/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.andromedad/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.andromedad/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.andromedad/config/app.toml
```

## Set minimum gas price
```
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0025uandr\"/" $HOME/.andromedad/config/app.toml
```

## Create Service
### Without Cosmovisor
```
sudo tee /etc/systemd/system/andromedad.service > /dev/null <<EOF
[Unit]
Description=Andromeda Testnet
After=network-online.target

[Service]
User=$USER
ExecStart=$(which andromedad) start --home $HOME/.andromedad
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
### With Cosmovisor
```
sudo tee /etc/systemd/system/andromedad.service > /dev/null << EOF
[Unit]
Description=Andromeda Testnet
After=network-online.target
[Service]
User=$USER
ExecStart=$HOME/go/bin/cosmovisor run start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.andromedad"
Environment="DAEMON_NAME=andromedad"
Environment="UNSAFE_SKIP_BACKUP=true"
[Install]
WantedBy=multi-user.target

EOF
```

## Register And Start Service
```
sudo systemctl daemon-reload
sudo systemctl enable andromedad
sudo systemctl restart andromedad && sudo journalctl -fu andromedad -o cat
```

## Deleting Node
```
sudo systemctl stop andromedad && \
sudo systemctl disable andromedad && \
rm /etc/systemd/system/andromedad.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf .andromedad && \
rm -rf $(which andromedad)
```
