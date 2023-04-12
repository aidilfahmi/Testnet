# $${\color{lightgreen}Cheatsheet}$$

## ${\color{orange}Key \space management}$ 
#### Add new key
```javascript
elysd keys add wallet
```
#### Recover existing key
```javascript
elysd keys add wallet --recover
```
#### List all keys
```javascript
elysd keys list
```
#### Delete key
```javascript
elysd keys delete wallet
```
#### Export key to the file
```javascript
elysd keys export wallet
```
#### Import key from the file
```javascript
elysd keys import wallet wallet.backup
```
#### Query wallet balance
```javascript
elysd q bank balances $(elysd keys show wallet -a)
```
## ${\color{orange}Validator \space Management}$   
Please make sure you have adjusted moniker, identity, details and website to match your values.
#### Create new validator
```javascript
elysd tx staking create-validator \
--amount 1000000uelys \
--pubkey $(elysd tendermint show-validator) \
--moniker "YOUR_MONIKER_NAME" \
--identity "YOUR_KEYBASE_ID" \
--details "YOUR_DETAILS" \
--website "YOUR_WEBSITE_URL" \
--chain-id elystestnet-1 \
--commission-rate 0.05 \
--commission-max-rate 0.20 \
--commission-max-change-rate 0.01 \
--min-self-delegation 1 \
--from wallet \
--gas-adjustment 1.4 \
--gas auto \
--gas-prices 0uelys \
-y
```
#### Edit existing validator
```javascript
elysd tx staking edit-validator \
--new-moniker "YOUR_MONIKER_NAME" \
--identity "YOUR_KEYBASE_ID" \
--details "YOUR_DETAILS" \
--website "YOUR_WEBSITE_URL" \
--chain-id elystestnet-1 \
--commission-rate 0.05 \
--from wallet \
--gas-adjustment 1.4 \
--gas auto \
--gas-prices 0uelys \
-y
```
#### Unjail validator
```javascript
elysd tx slashing unjail --from wallet --chain-id elystestnet-1 --gas-adjustment 1.4 --gas auto --gas-prices 0uelys -y
```
#### Jail reason
```javascript
elysd query slashing signing-info $(elysd tendermint show-validator)
```
#### List all active validators
```javascript
elysd q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```
#### List all inactive validators
```javascript
elysd q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```
#### View validator details
```javascript
elysd q staking validator $(elysd keys show wallet --bech val -a)
```
## ${\color{orange}Token \space Management}$
#### Withdraw rewards from all validators
```javascript
elysd tx distribution withdraw-all-rewards --from wallet --chain-id elystestnet-1 --gas-adjustment 1.4 --gas auto --gas-prices 0uelys -y
```
#### Withdraw commission and rewards from your validator
```javascript
elysd tx distribution withdraw-rewards $(elysd keys show wallet --bech val -a) --commission --from wallet --chain-id elystestnet-1 --gas-adjustment 1.4 --gas auto --gas-prices 0uelys -y
```
#### Delegate tokens to yourself
```javascript
elysd tx staking delegate $(elysd keys show wallet --bech val -a) 1000000uelys --from wallet --chain-id elystestnet-1 --gas-adjustment 1.4 --gas auto --gas-prices 0uelys -y
```
#### Delegate tokens to validator
```javascript
elysd tx staking delegate <TO_VALOPER_ADDRESS> 1000000uelys --from wallet --chain-id elystestnet-1 --gas-adjustment 1.4 --gas auto --gas-prices 0uelys -y
```
#### Redelegate tokens to another validator
```javascript
elysd tx staking redelegate $(elysd keys show wallet --bech val -a) <TO_VALOPER_ADDRESS> 1000000uelys --from wallet --chain-id elystestnet-1 --gas-adjustment 1.4 --gas auto --gas-prices 0uelys -y
```
#### Unbond tokens from your validator
```javascript
elysd tx staking unbond $(elysd keys show wallet --bech val -a) 1000000uelys --from wallet --chain-id elystestnet-1 --gas-adjustment 1.4 --gas auto --gas-prices 0uelys -y
```
#### Send tokens to the wallet
```javascript
elysd tx bank send wallet <TO_WALLET_ADDRESS> 1000000uelys --from wallet --chain-id elystestnet-1 --gas-adjustment 1.4 --gas auto --gas-prices 0uelys -y
```
## ${\color{orange}Utility}$
### Update Indexer
#### Disable indexer
```javascript
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.elys/config/config.toml
```
#### Enable indexer
```javascript
sed -i -e 's|^indexer *=.*|indexer = "kv"|' $HOME/.elys/config/config.toml
```
## ${\color{orange}Maintenance}$ 
#### Get validator info
```javascript
elysd status 2>&1 | jq .ValidatorInfo
```
#### Get sync info
```javascript
elysd status 2>&1 | jq .SyncInfo
```
#### Set minimum gas price
```javascript
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0uelys\"/" $HOME/.elys/config/app.toml
```
#### Enable prometheus
```javascript
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.elys/config/config.toml
```
#### Reset chain data
```javascript
elysd tendermint unsafe-reset-all --home $HOME/.elys --keep-addr-book
```
## Remove node
Please, before proceeding with the next step! All chain data will be lost! Make sure you have backed up your priv_validator_key.json!
```javascript
cd $HOME
sudo systemctl stop elysd
sudo systemctl disable elysd
sudo rm /etc/systemd/system/elysd.service
sudo systemctl daemon-reload
rm -f $(which elysd)
rm -rf $HOME/.elys
rm -rf $HOME/elys
```
