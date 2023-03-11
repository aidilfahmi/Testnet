# $${\color{lightgreen}Manual \space Node \space Installation}$$

## ${\color{lightblue}Optional \space Configuration}$
### Custom User

```
sudo adduser ojo
sudo adduser ojo sudo
su - ojo
```
### Custom Port
```diff
- Skip this step if running default port configurations
```
```
PORT=59
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
curl -LOf https://github.com/ojo-network/ojo/releases/download/v0.1.2/ojod-v0.1.2-linux-amd64
chmod +x ojod-v0.1.2-linux-amd64
mv ojod-v0.1.2-linux-amd64 ojod

```

#### Install Cosmovisor
```
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.4.0
mkdir -p $HOME/.ojo/cosmovisor/genesis/bin
cp $HOME/ojod $HOME/.ojo/cosmovisor/genesis/bin
ln -s $HOME/.ojo/cosmovisor/genesis $HOME/.ojo/cosmovisor/current
sudo ln -s $HOME/.ojo/cosmovisor/current/bin/ojod /usr/bin/ojod
```

### Init Generation

Replace `moniker_name` with your own moniker name
```
ojod config chain-id OJO-DEVNET
ojod config keyring-backend test
ojod init moniker_name --chain-id OJO-DEVNET
```

### Set Peers and Seeds
```
peers="d5519e378247dfb61dfe90652d1fe3e2b3005a5b@65.109.68.190:50656,5af3d50dcc231884f3d3da3e3caecb0deef1dc5b@142.132.134.112:25356,62fa77951a7c8f323c0499fff716cd86932d8996@65.108.199.36:24214,9edc978fd53c8718ef0cafe62ed8ae23b4603102@136.243.103.32:36656,ac5089a8789736e2bc3eee0bf79ca04e22202bef@162.55.80.116:29656,bd35cfd5bfbea4c2a63e893860d4f9a7d880957c@213.239.217.52:45656,408ee86160af26ee7204d220498e80638f7874f4@161.97.109.47:38656,c37e444f67af17545393ad16930cd68dc7e3fd08@95.216.7.169:61156,fbeb2b37fe139399d7513219e25afd9eb8f81f4f@65.21.170.3:38656,239caa37cb0f131b01be8151631b649dc700cd97@95.217.200.36:46656,e54b02d103f1fcf5189a86abe542670979d2029d@65.109.85.170:58656,9bcec17faba1b8f6583d37103f20bd9b968ac857@38.146.3.230:21656,1145755896d6a3e9df2f130cc2cbd223cdb206f0@209.145.53.163:29656,b0968b57bcb5e527230ef3cfa3f65d5f1e4647dd@35.212.224.95:26656,8671c2dbbfd918374292e2c760704414d853f5b7@35.215.121.109:26656,2691bb6b296b951400d871c8d0bd94a3a1cdbd52@65.109.93.152:33656,cbe534c7d012e9eb4e71a5573aee8acc1adf4bc6@65.108.41.172:28056,a23cc4cbb09108bc9af380083108262454539aeb@35.215.116.65:26656,3d11a6c7a5d4b3c5752be0c252c557ed4acc2c30@167.235.57.142:36656,b6b4a4c720c4b4a191f0c5583cc298b545c330df@65.109.28.219:21656"
sed -i -e "s|^persistent_peers *=.*|persistent_peers = \"$peers\"|" $HOME/.ojo/config/config.toml
```

### Download Genesis and Addrbook
```
curl -Ls https://snapshots.kjnodes.com/ojo-testnet/genesis.json > $HOME/.ojo/config/genesis.json
curl -Ls https://snapshots.kjnodes.com/ojo-testnet/addrbook.json > $HOME/.ojo/config/addrbook.json
```

### Set Port app.toml and config.toml
```diff
- This Step If you running custom port, if you running default port, skip this step
```
```
ojod config node tcp://localhost:${PORT}657
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${PORT}660\"%" $HOME/.ojo/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${PORT}317\"%; s%^address = \":8080\"%address = \":${PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${PORT}091\"%" $HOME/.ojo/config/app.toml

```

### Set Config Pruning
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="19"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.ojo/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.ojo/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.ojo/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.ojo/config/app.toml
```

### Set minimum gas price
```
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0uojo\"|" $HOME/.ojo/config/app.toml
```

### Create Service

```
sudo tee /etc/systemd/system/ojod.service > /dev/null << EOF
[Unit]
Description=OJO Testnet
After=network-online.target
[Service]
User=$USER
ExecStart=$HOME/go/bin/cosmovisor run start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.ojo"
Environment="DAEMON_NAME=ojod"
Environment="UNSAFE_SKIP_BACKUP=true"
[Install]
WantedBy=multi-user.target

EOF
```

### Register And Start Service
```
sudo systemctl enable ojod
sudo systemctl daemon-reload
sudo systemctl start ojod && sudo journalctl -fu ojod -o cat
```

## ${\color{orange}SNAPSHOT}$	

### Stop Service and backup priv_validator
```
sudo systemctl stop ojod
cp $HOME/.ojo/data/priv_validator_state.json $HOME/.ojo/priv_validator_state.json.backup
rm -rf $HOME/.ojo/data
```

### Download Snapshot
```
curl -L https://ojo-t.service.indonode.net/ojo-snapshot.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.ojo
mv $HOME/.ojo/priv_validator_state.json.backup $HOME/.ojo/data/priv_validator_state.json
```
### Restart Node and Check the logs
```
sudo systemctl restart ojod && sudo journalctl -u ojod -f --no-hostname -o cat
```

## Deleting Node
```
sudo systemctl stop ojod && \
sudo systemctl disable ojod && \
rm /etc/systemd/system/ojod.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf .ojo && \
rm -rf $(which ojod)
```
