## Manual Installation

<!-- tabs:start -->
#### **Custom User & Port (optional)**
<span style="color: rgb(241, 196, 15);">Custom user</span>

```bash
sudo adduser mantra
sudo adduser mantra sudo
su - mantra
```

<span style="color: rgb(241, 196, 15);">Custom Port </span>

```bash
PORT=45
echo "export PORT=${PORT}" >> $HOME/.bash_profile
mantra $HOME/.bash_profile
```

#### **Installing Binary**
<span style="color: rgb(241, 196, 15);">Install Update and Dependencies
```bash
sudo apt -q update
sudo apt -qy upgrade
sudo apt -qy install curl git jq wget unzip lz4 build-essential
```

<span style="color: rgb(241, 196, 15);">Installing GO
```bash
sudo rm -rf /usr/local/go && \
curl -Ls https://go.dev/dl/go1.20.5.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
mantra $HOME/.bash_profile && \
go version
```

<span style="color: rgb(241, 196, 15);">Installing Binary
```bash
cd $HOME
rm mantrachaind
wget https://github.com/MANTRA-Finance/public/raw/main/mantrachain-testnet/mantrachaind-linux-amd64.zip
unzip mantrachaind-linux-amd64.zip
sudo mv mantrachaind /usr/local/bin
```

#### **Init Generation**
<span style="color: rgb(241, 196, 15);">Create Init
```bash
mantrachaind config chain-id mantrachain-testnet-1
mantrachaind config keyring-backend test
mantrachaind init moniker_name --chain-id mantrachain-testnet-1
```

<span style="color: rgb(241, 196, 15);">Download Genesis & addrbook
```bash
curl -Ls https://snapshots.indonode.net/mantra/genesis.json > $HOME/.mantrachain/config/genesis.json
curl -Ls https://snapshots.indonode.net/mantra/addrbook.json > $HOME/.mantrachain/config/addrbook.json
```

#### **Custom Config**
<span style="color: rgb(241, 196, 15);">Custom Peers and Seeds
```bash
PEERS="96d63849a529a15f037a28c276ea6e3ac2449695@34.222.1.252:26656,0107ac60e43f3b3d395fea706cb54877a3241d21@35.87.85.162:26656"
sed -i -e "s|^persistent_peers *=.*|persistent_peers = \"$PEERS\"|" $HOME/.mantrachain/config/config.toml
```
<span style="color: rgb(241, 196, 15);">Custom Port
```bash
mantrachaind config node tcp://localhost:${PORT}657
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${PORT}660\"%" $HOME/.mantrachain/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${PORT}317\"%; s%^address = \":8080\"%address = \":${PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${PORT}091\"%" $HOME/.mantrachain/config/app.toml
sed -i.bak -e "s%^address = \"tcp://localhost:1317\"%address = \"tcp://localhost:${PORT}317\"%; s%^address = \":8080\"%address = \":${PORT}080\"%; s%^address = \"localhost:9090\"%address = \"localhost:${PORT}090\"%; s%^address = \"localhost:9091\"%address = \"localhost:${PORT}091\"%" $HOME/.mantrachain/config/app.toml
```
<span style="color: rgb(241, 196, 15);">Custom Pruning
```bash
pruning="custom"
pruning_keep_every="0"
pruning_keep_recent="100"
pruning_interval="19"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.mantrachain/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.mantrachain/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.mantrachain/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.mantrachain/config/app.toml
```

<span style="color: rgb(241, 196, 15);">Indexer & Gas Price
```bash
# Setting Indexer Off
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.mantrachain/config/config.toml
```
```bash
# Minimum Gas Price
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.25umantra\"/" $HOME/.mantrachain/config/app.toml
```
#### **Service**
```bash
sudo tee /etc/systemd/system/mantrachaind.service > /dev/null <<EOF
[Unit]
Description=Mantrachain testnet node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which mantrachaind) start
Restart=always
RestartSec=3
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```
```bash
sudo systemctl daemon-reload
sudo systemctl enable mantrachaind
sudo systemctl restart mantrachaind && sudo journalctl -fu mantrachaind -o cat
```

<!-- tabs:end -->

## Create Validator

<!-- tabs:start -->
#### **Wallet**
- Create wallet
```bash
mantrachaind keys add your_wallet_name
```
- Recover Wallet
```bash
mantrachaind keys add your_wallet_name --recover
```

#### **Create Validator**
```bash
mantrachaind tx staking create-validator \
  --amount 9900000uaum \
  --from wallet_name \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.10" \
  --min-self-delegation "1000000" \
  --pubkey $(mantrachaind tendermint show-validator) \
  --moniker YOUR_MONIKER \
  --chain-id mantrachain-1 \
  --identity=  \
  --website="" \
  --details=" " \
  --gas="auto" \
  --gas-prices="0uaum" \
  --gas-adjustment="1.15" \
  -y
```
<!-- tabs:end -->


## CLI Cheatsheet

<!-- tabs:start -->
#### **Key management**
<span style="color: rgb(241, 196, 15);">Add New Key</span>

```bash
mantrachaind keys add wallet
```

<span style="color: rgb(241, 196, 15);">Recover Existing Key</span>

```bash
mantrachaind keys add wallet --recover
```

<span style="color: rgb(241, 196, 15);">List All Keys</span>

```bash
mantrachaind keys list
```

<span style="color: rgb(241, 196, 15);">Delete Key</span>

```bash
mantrachaind keys delete wallet
```

<span style="color: rgb(241, 196, 15);">Export Key to File</span>

```bash
mantrachaind keys export wallet
```

<span style="color: rgb(241, 196, 15);">Import Key From File</span>

```bash
mantrachaind keys import wallet wallet.backup
```

<span style="color: rgb(241, 196, 15);">Query Wallet Balance</span>

```bash
mantrachaind q bank balances $(mantrachaind keys show wallet -a)
```

#### **Validator management**
!> *Please make sure you have adjusted `moniker`, `identity`, `details` and `website` to match your values.*

<span style="color: rgb(241, 196, 15);">Edit Existing Validator</span>
```bash
mantrachaind tx staking edit-validator \
--new-moniker "YOUR_MONIKER_NAME" \
--identity "YOUR_KEYBASE_ID" \
--details "YOUR_DETAILS" \
--website "YOUR_WEBSITE_URL" \
--chain-id mantrachain-1 \
--commission-rate 0.10 \
--from wallet_name \
--gas="auto" \
--gas-prices="0uaum" \
--gas-adjustment="1.15" \
-y
```

<span style="color: rgb(241, 196, 15);">Unjail Validator</span>
```bash
mantrachaind tx slashing unjail --from wallet --chain-id source-1 --gas-prices "0.025usource" --gas-adjustment 1.5 --gas "auto" -y 
```

<span style="color: rgb(241, 196, 15);">View Validator Details</span>
```bash
mantrachaind q staking validator $(mantrachaind keys show wallet --bech val -a) 
```

#### **Token management**
<span style="color: rgb(241, 196, 15);">Withdraw Rewards From All Validator</span>
```bash
mantrachaind tx distribution withdraw-all-rewards --from wallet --chain-id source-1 --gas-prices "0.025usource" --gas-adjustment 1.5 --gas "auto"
```

<span style="color: rgb(241, 196, 15);">Withdraw Commission and Rewards From Your Validator</span>
```bash
mantrachaind tx distribution withdraw-rewards $(mantrachaind keys show wallet --bech val -a) --commission --from wallet --chain-id source-1 --gas-prices "0.025usource" --gas-adjustment 1.5 --gas "auto"
```

<span style="color: rgb(241, 196, 15);">Delegate Tokens to Yorself</span>
```bash
mantrachaind tx staking delegate $(mantrachaind keys show wallet --bech val -a) 1000000usource --from wallet --chain-id source-1 --gas-prices "0.025usource" --gas-adjustment 1.5 --gas "auto"
```

<span style="color: rgb(241, 196, 15);">Delegate Elegate Tokens to Validator</span>
```bash
mantrachaind tx staking delegate TO_VALOPER_ADDRESS 1000000usource --from wallet --chain-id source-1 --gas-prices "0.025usource" --gas-adjustment 1.5 --gas "auto"
```

<span style="color: rgb(241, 196, 15);">Redelegate Tokens to Another Validator</span>
```bash
mantrachaind tx staking redelegate $(mantrachaind keys show wallet --bech val -a) TO_VALOPER_ADDRESS 1000000usource --from wallet --chain-id source-1 --gas-prices "0.025usource" --gas-adjustment 1.5 --gas "auto"
```

<span style="color: rgb(241, 196, 15);">Unbound Tokens From Your Validator</span>
```bash
mantrachaind tx staking unbond $(mantrachaind keys show wallet --bech val -a) 1000000usource --from wallet --chain-id source-1 --gas-prices "0.025usource" --gas-adjustment 1.5 --gas "auto" -y 
```

#### **Governance**
<span style="color: rgb(241, 196, 15);">List all proposals</span>
```
mantrachaind query gov proposals
```

<span style="color: rgb(241, 196, 15);">Vote Proposal</span>
<p class="callout info">Vote **YES / NO / ABSTAIN / NO\_WITH\_VETO**</p>
```
mantrachaind tx gov vote 1 yes --from wallet --chain-id source-1 --gas-prices "0.025usource" --gas-adjustment 1.5 --gas "auto"
```

<span style="color: rgb(241, 196, 15);">Create new text proposal</span>

```
mantrachaind tx gov submit-proposal \
--title="Title" \
--description="Description" \
--deposit=100000000usource \
--type="Text" \
--from=wallet \
--gas-prices 0usource\ 
--gas-adjustment 1.5 \
--gas "auto"
```


#### **Utility**
<span style="color: rgb(241, 196, 15);">Update Ports</span>  
```bash
CUSTOM_PORT=53
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${CUSTOM_PORT}58\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${CUSTOM_PORT}57\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${CUSTOM_PORT}60\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${CUSTOM_PORT}56\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${CUSTOM_PORT}66\"%" $HOME/.mantrachaind/config/config.toml
sed -i -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${CUSTOM_PORT}17\"%; s%^address = \":8080\"%address = \":${CUSTOM_PORT}80\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${CUSTOM_PORT}90\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${CUSTOM_PORT}91\"%" $HOME/.mantrachaind/config/app.toml
```

<span style="color: rgb(241, 196, 15);">Disable indexer</span>
```
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.mantrachaind/config/config.toml
```

<span style="color: rgb(241, 196, 15);">Enable indexer</span>
```
sed -i -e 's|^indexer *=.*|indexer = "kv"|' $HOME/.mantrachaind/config/config.toml
```

<span style="color: rgb(241, 196, 15);">Update Pruning</span>
```bash
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "19"|' \
  $HOME/.mantrachaind/config/app.toml
```

#### **Maintenance**
<span style="color: rgb(241, 196, 15);">Get validator Info</span>

```bash
mantrachaind status 2>&1 | jq .ValidatorInfo
```

<span style="color: rgb(241, 196, 15);">Get Sync Info</span>

```bash
mantrachaind status 2>&1 | jq .SyncInfo
```

<span style="color: rgb(241, 196, 15);">Get Node Peer</span>

```bash
echo $(mantrachaind tendermint show-node-id)'@'$(curl -s ifconfig.me)':'$(cat $HOME/.mantrachaind/config/config.toml | sed -n '/Address to listen for incoming connection/{n;p;}' | sed 's/.*://; s/".*//')
```

<span style="color: rgb(241, 196, 15);">Get Live Peers</span>

```bash
curl -sS http://localhost:14657/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'
```

<span style="color: rgb(241, 196, 15);">Set Minimum Gas Price</span>

```bash
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.025aISLM\"/" $HOME/.mantrachaind/config/app.toml
```

<span style="color: rgb(241, 196, 15);">Enable Prometheus</span>

```bash
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.mantrachaind/config/config.toml
```

<span style="color: rgb(241, 196, 15);">Reset Chain Data</span>

```bash
mantrachaind tendermint unsafe-reset-all --keep-addr-book --home $HOME/.mantrachaind --keep-addr-book
```

<span style="color: rgb(241, 196, 15);">Check Service Logs</span>

```bash
sudo journalctl -u mantrachaind -f --no-hostname -o cat
```

##### **Remove Node**

<p class="callout danger">Please, before proceeding with the next step! All chain data will be lost! Make sure you have backed up your **priv\_validator\_key.json**!</p>

```bash
cd $HOME
sudo systemctl stop mantrachaind
sudo systemctl disable mantrachaind
sudo rm /etc/systemd/system/mantrachaind.service
sudo systemctl daemon-reload
rm -f $(which mantrachaind)
rm -rf $HOME/.mantrachaind
```
<!-- tabs:end -->


