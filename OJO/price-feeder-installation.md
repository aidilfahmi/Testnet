# Price Feeder Installation
## Disclaimer
To run pricefeeder you validator should be in active set. Otherwise price feeder will not vote on periods.

## Install the pricefeeder binary and create directory for pricefeeder configuration
```
cd $HOME && rm price-feeder -rf
git clone https://github.com/ojo-network/price-feeder
cd price-feeder
git checkout v0.1.1
make build
```
```
sudo mv ./build/price-feeder /usr/local/bin
rm $HOME/.ojo-price-feeder -rf
mkdir $HOME/.ojo-price-feeder
mv price-feeder.example.toml $HOME/.ojo-price-feeder/config.toml
```
### Check price-feeder version
```
price-feeder version
```
version: HEAD-5d46ed438d33d7904c0d947ebc6a3dd48ce0de59<br>
commit: 5d46ed438d33d7904c0d947ebc6a3dd48ce0de59

### Create new wallet for pricefeeder and save 24 word mnemonic phrase
```
ojod keys add pricefeeder-wallet --keyring-backend os
```
### Export keyring password
```
export KEYRING_PASSWORD="PRICE_FEEDER_KEY_PASSWORD_GOES_HERE"
```
### Set up variables
```
export KEYRING="os"
export LISTEN_PORT=7172
export RPC_PORT=59657
export GRPC_PORT=59090
export VALIDATOR_ADDRESS=$(ojod keys show wallet --bech val -a)
export MAIN_WALLET_ADDRESS=$(ojod keys show wallet -a)
export PRICEFEEDER_ADDRESS=$(echo -e $KEYRING_PASSWORD | ojod keys show pricefeeder-wallet --keyring-backend os -a)
```
### Fund the pricefeeder-wallet with some testnet tokens.
In order to make pricefeeder work, it needs some tokens to pay for transaction fees <br>
This command will send 1 OJO to pricefeeder-wallet from you main wallet
```
ojod tx bank send wallet $PRICEFEEDER_ADDRESS 1000000uojo --from wallet --chain-id ojo-devnet --gas-adjustment 1.4 --gas auto --gas-prices 0uojo -y
```
### Check the balance
```
ojod q bank balances $PRICEFEEDER_ADDRESS
```
### Delegate pricefeeder responsibility
As a validator, if you'd like another account to post prices on your behalf (i.e. you don't want your validator mnemonic sending txs), you can delegate pricefeeder responsibilities to another address.
```
ojod tx oracle delegate-feed-consent $MAIN_WALLET_ADDRESS $PRICEFEEDER_ADDRESS --from wallet --gas-adjustment 1.4 --gas auto --gas-prices 0uojo --chain-id ojo-devnet -y
```
### Check linked pricefeeder address
```
ojod q oracle feeder-delegation $VALIDATOR_ADDRESS
```
### Set pricefeeder configuration values
```
sed -i "s/^listen_addr *=.*/listen_addr = \"0.0.0.0:${LISTEN_PORT}\"/;\
s/^address *=.*/address = \"$PRICEFEEDER_ADDRESS\"/;\
s/^chain_id *=.*/chain_id = \"ojo-devnet\"/;\
s/^validator *=.*/validator = \"$VALIDATOR_ADDRESS\"/;\
s/^backend *=.*/backend = \"$KEYRING\"/;\
s|^dir *=.*|dir = \"$HOME/.ojo\"|;\
s|^grpc_endpoint *=.*|grpc_endpoint = \"localhost:${GRPC_PORT}\"|;\
s|^tmrpc_endpoint *=.*|tmrpc_endpoint = \"http://localhost:${RPC_PORT}\"|;\
s|^global-labels *=.*|global-labels = [[\"chain_id\", \"ojo-devnet\"]]|;\
s|^service-name *=.*|service-name = \"ojo-price-feeder\"|;" $HOME/.ojo-price-feeder/config.toml
```
### Setup the systemd service
```
sudo tee /etc/systemd/system/ojo-price-feeder.service > /dev/null <<EOF
[Unit]
Description=Ojo Price Feeder
After=network-online.target

[Service]
User=$USER
ExecStart=$(which price-feeder) $HOME/.ojo-price-feeder/config.toml
Restart=on-failure
RestartSec=30
LimitNOFILE=65535
Environment="PRICE_FEEDER_PASS=$KEYRING_PASSWORD"

[Install]
WantedBy=multi-user.target
EOF
```
## Register and start the systemd service
```
sudo systemctl daemon-reload
sudo systemctl enable ojo-price-feeder
sudo systemctl restart ojo-price-feeder
```
### View pricefeeder logs
```
journalctl -fu ojo-price-feeder -o cat
```

## Useful commands
### Check current voting windows progress
```
ojod q oracle slash-window
```
### Check missed oracle votes per slashing window
```
ojod q oracle miss-counter $VALIDATOR_ADDRESS
```

Source and thanks to [kjnodes#8455](https://services.kjnodes.com/home/testnet/ojo/installation#set-up-price-feeder)
