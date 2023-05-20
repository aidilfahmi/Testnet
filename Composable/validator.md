## Create or Recover wallet

```javascript
banksyd keys add <walletname>
  OR
banksyd keys add <walletname> --recover
```

## Create validator

Delegate for 10PICA
```javascript
banksyd tx staking create-validator \
  --amount 10000000upica \
  --from wallet_name \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --moniker "Your_moniker" \
  --identity "" \
  --details "" \
  --website "" \
  --commission-rate "0.05" \
  --min-self-delegation "1" \
  --pubkey  $(banksyd tendermint show-validator) \
  --gas=auto \
  --gas-adjustment=1.2 \
  --gas-prices=0upica \
  --chain-id banksy-testnet-2
```
### Sync Info
```javascript
banksyd status 2>&1 | jq .SyncInfo
```
### NodeINfo
```javascript
banksyd status 2>&1 | jq .NodeInfo
```
### Check node logs
```javascript
banksyd journalctl -u banksyd -f -o cat
```
### Check Balance
```javascript
banksyd query bank balances $(banksyd keys show wallet -a)
```
