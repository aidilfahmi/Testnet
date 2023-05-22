## Create or Recover wallet

```javascript
banksyd keys add <walletname>
  OR
banksyd keys add <walletname> --recover
```

## Create validator

Delegate for 1 C4E
```javascript
c4ed tx staking create-validator \
  --amount 1000000uc4e \
  --from wallet_name \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --moniker "Your_moniker" \
  --identity "" \
  --details "" \
  --website "" \
  --commission-rate "0.05" \
  --min-self-delegation "1" \
  --pubkey  $(c4ed tendermint show-validator) \
  --gas=auto \
  --chain-id babajaga-1
```
### Sync Info
```javascript
c4ed status 2>&1 | jq .SyncInfo
```
### NodeINfo
```javascript
c4ed status 2>&1 | jq .NodeInfo
```
### Check node logs
```javascript
journalctl -u c4ed -f -o cat
```
### Check Balance
```javascript
c4ed query bank balances $(c4ed keys show wallet -a)
```
