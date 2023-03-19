## Create or Recover wallet

```
bonus-blockd keys add <walletname>
  OR
bonus-blockd keys add <walletname> --recover
```

## Create validator

Delegate for 10bonus
```
bonus-blockd tx staking create-validator \
  --amount 10000000ubonus \
  --from wallet_name \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --moniker "Your_moniker" \
  --identity "" \
  --details "" \
  --website "" \
  --commission-rate "0.05" \
  --min-self-delegation "1" \
  --pubkey  $(bonus-blockd tendermint show-validator) \
  --gas=auto \
  --gas-adjustment=1.2 \
  --gas-prices=0.025ubonus \
  --chain-id blocktopia-01
```
### Sync Info
```
bonus-blockd status 2>&1 | jq .SyncInfo
```
### NodeINfo
```
bonus-blockd status 2>&1 | jq .NodeInfo
```
### Check node logs
```
bonus-blockd journalctl -u ojod -f -o cat
```
### Check Balance
```
bonus-blockd query bank balances $(bonus-blockd keys show wallet_name -a)
```
