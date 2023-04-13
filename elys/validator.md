## Create or Recover wallet

```javascript
elysd keys add <walletname>
  OR
elysd keys add <walletname> --recover
```

## Create validator

Delegate for 10uelys
```javascript
elysd tx staking create-validator \
  --amount 10000000uelys \
  --from wallet_name \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --moniker "Your_moniker" \
  --identity "" \
  --details "" \
  --website "" \
  --commission-rate "0.05" \
  --min-self-delegation "1" \
  --pubkey  $(elysd tendermint show-validator) \
  --gas=auto \
  --gas-adjustment=1.2 \
  --gas-prices=0uelys \
  --chain-id elystestnet-1
```
### Sync Info
```javascript
elysd status 2>&1 | jq .SyncInfo
```
### NodeINfo
```javascript
elysd status 2>&1 | jq .NodeInfo
```
### Check node logs
```javascript
elysd journalctl -u elysd -f -o cat
```
### Check Balance
```javascript
elysd query bank balances $(elysd keys show wallet -a)
```
