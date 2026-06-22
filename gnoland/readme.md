# Gnoland test13

## 1. Binaries
```shell
git clone https://github.com/gnolang/gno.git
cd gno && git checkout chain/test13
make -C gno.land install.gnoland install.gnokey
```

## 2. Genesis

Download `genesis.json`

```shell
wget -O genesis.json https://github.com/gnolang/gno/releases/download/chain/test13/genesis.json
```
## 3. Configure your node
Generate a default config and your keys:

```shell
gnoland config init       # writes a default config.toml in your node directory
gnoland secrets init      # generates your validator + node keys
```
```shell
gnoland config set moniker your_noniker
gnoland config set consensus.peer_gossip_sleep_duration 10ms
gnoland config set consensus.timeout_commit 3s
gnoland config set mempool.size 10000
gnoland config set p2p.flush_throttle_timeout 10ms
gnoland config set p2p.max_num_outbound_peers 40
gnoland config set p2p.persistent_peers g142k7zc2qym3c0u6jmkf6rv26llgr2f4nakmlmt@sentry-1.test13.testnets.gno.land:26656,g1lxkf9gn7kddrr26c640ww5wg3ezsm22we8cjpc@sentry-2.test13.testnets.gno.land:26656
```
## 4. Wallet
```shell
# 4. Create Wallet
gnokey add wallet
```
```shell
# Recover wallet
gnokey add wallet -recover 
```
## 5. Create Service
```shell
sudo tee /etc/systemd/system/gnoland.service > /dev/null <<EOF
[Unit]
Description=Gnoland node
After=network-online.target
[Service]
User=$USER
WorkingDirectory=$HOME/gno
Environment="GNOROOT=$HOME/gno"
ExecStart=$(which gnoland) start --gnoroot-dir $HOME/gno --genesis $HOME/gno/genesis.json --chainid test-13  --data-dir $HOME/gno/gnoland-data/ --skip-genesis-sig-verification
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```
```shell
sudo systemctl daemon-reload
sudo systemctl restart gnoland && sudo journalctl -u gnoland -f
```

## 6. Register as a validator candidate
The registration transaction costs a gas fee, so your operator account needs GNOT. If it's empty, request a drip for your `g1...` address from the test13 faucet at <https://test13.testnets.gno.land/faucet>.

Get your node's consensus public key and wallet address
```shell
gnoland secrets get validator_key | jq -r '.pub_key'
gnokey list
```
`check your account balance, account_number and sequence`

```shell
gnokey query -remote "https://rpc.test13.testnets.gno.land" auth/accounts/WALLET_ADDRESS

example :
gnokey query -remote "https://rpc.test13.testnets.gno.land" auth/accounts/g1etf3rd7dqghksj8r5yun12qwefspdfszdjhuvga
```


```shell
gnokey maketx call -pkgpath "gno.land/r/gnops/valopers" -func "Register" -args $'' -args $'' -args $'' -args $'' -args $'' -gas-fee 1000000ugnot -gas-wanted 50_000_000 -send "" -broadcast=false WALLET_ADDRESS > call.tx

example ;
gnokey maketx call -pkgpath "gno.land/r/gnops/valopers" -func "Register" -args $'dnsarz' -args $'sample descriptions' -args $'data-center' -args $'g1etf3rd7dqghksj8r5yun12qwefspdfszdjhuvga' -args $'gpub1pggj7ard9eg82cjtv4u52epjx56nzwgjyg9zps250jghhjoiffnhzhf0vqrk3dyhgwvy0p3vszz2hqs2spqnmgxtp2vuz' -gas-fee 1000000ugnot -gas-wanted 1_000_000_000 -send "" -broadcast=false g1etf3rd7dqghksj8r5yun12qwefspdfszdjhuvga > call.tx
```
```shell
gnokey sign -tx-path call.tx -chainid "test-13" -account-number ACCOUNTNUMBER -account-sequence SEQUENCENUMBER WALLET_ADDRESS

example:
gnokey sign -tx-path call.tx -chainid "test-13" -account-number 12345678 -account-sequence 2 g1etf3rd7dqghksj8r5yun12qwefspdfszdjhuvga
```
```shell
gnokey broadcast -remote "https://rpc.test13.testnets.gno.land" call.tx
```
You can review registered valopers and the current set at <https://test13.testnets.gno.land/r/gnops/valopers> and <https://test13.testnets.gno.land/r/sys/validators/v3>.
