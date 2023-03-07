## Create or Recover wallet

```
ojod keys add <walletname>
  OR
ojod keys add <walletname> --recover
```

## Create validator
```
ojod tx staking create-validator \
--amount=1000000uojo \
--pubkey=$(ojod tendermint show-validator) \
--moniker="Your_moniker" \
--website "" \
--identity "" \
--details "" \
--chain-id=ojo-devnet \
--commission-rate=0.05 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.01 \
--min-self-delegation=1 \
--from=wallet \
--gas-adjustment=1.4 \
--gas auto \
--gas-prices 0uojo
```

### Sync Info
```
ojod status 2>&1 | jq .SyncInfo
```
### NodeINfo
```
ojod status 2>&1 | jq .NodeInfo
```
### Check node logs
```
sudo journalctl -u ojod -f -o cat
```
### Check Balance
```
ojod query bank balances $(ojod keys show wallet_name -a)
```
