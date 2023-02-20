## Create or Recover wallet

```
andromedad keys add <walletname>
  OR
andromedad keys add <walletname> --recover
```

## Apply SnapShot 
```
cd $HOME
sudo apt install lz4
sudo systemctl stop andromedad
cp $HOME/.andromedad/data/priv_validator_state.json $HOME/.andromedad/priv_validator_state.json.backup
rm -rf $HOME/.andromedad/data
curl -o - -L http://andromedad.snapshot.stavr.tech:1021/andromedad/andromedad-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.andromedad --strip-components 2
curl -o - -L http://andromedad.wasm.stavr.tech:1002/wasm-andromedad.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.andromedad --strip-components 2
mv $HOME/.andromedad/priv_validator_state.json.backup $HOME/.andromedad/data/priv_validator_state.json
wget -O $HOME/.andromedad/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/AndromedaProtocol/addrbook.json"
sudo systemctl restart andromedad && journalctl -u andromedad -f -o cat
```

## Create validator
```
andromedad tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 1 \
--commission-max-change-rate 1 \
--min-self-delegation "1" \
--amount 1000000uandr \
--pubkey $(andromedad tendermint show-validator) \
--from <wallet> \
--moniker="moniker_name" \
--chain-id galileo-3 \
--gas 350000 \
--identity="" \
--website="" \
--details=""
```

### Sync Info
```
andromedad status 2>&1 | jq .SyncInfo
```
### NodeINfo
```
andromedad status 2>&1 | jq .NodeInfo
```
### Check node logs
```
sudo journalctl -u andromedad -f -o cat
```
### Check Balance
```
andromedad query bank balances $(planqd keys show wallet_name -a)
```
