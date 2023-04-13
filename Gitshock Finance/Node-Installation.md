# $${\color{lightgreen}Gitshock \space Node \space Installation \space Geth - Lighthouse}$$

## ${\color{orange}Preparing}$	
### ${\color{lightblue}Update - Install\space Golang-Ethereum}$
```javascript
sudo apt update && sudo apt upgrade -y
sudo apt install build-essential -y
sudo apt install micro -y
sudo add-apt-repository -y ppa:ethereum/ethereum
sudo apt update -y
sudo apt install ethereum -y
sudo apt install jq
```
```javascript
sudo rm -rf /usr/local/go && \
curl -Ls https://go.dev/dl/go1.20.2.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
go install github.com/protolambda/eth2-testnet-genesis@latest
go install github.com/protolambda/eth2-val-tools@latest
```
### ${\color{lightblue}Installing \space Lighthouse}$ 
```javascript
cd $HOME
wget https://github.com/sigp/lighthouse/releases/download/v4.0.1/lighthouse-v4.0.1-x86_64-unknown-linux-gnu-portable.tar.gz
tar zxvf lighthouse-v4.0.1-x86_64-unknown-linux-gnu-portable.tar.gz && rm lighthouse-v4.0.1-x86_64-unknown-linux-gnu-portable.tar.gz
sudo mv -s $HOME/lighthouse /usr/local/bin/lighthouse
```
### ${\color{lightblue}Cloning \space Source-Code}$ 
```javascript
git clone https://github.com/gitshock-labs/testnet-list 
cd testnet-list
git checkout master 
mkdir data
mkdir logs 
mkdir beacon-1
mkdir beacon-2 
mkdir validator
```
- Create a JWT Secret to create a new secret key.
```javascript
openssl rand -hex 32 | tr -d "\n" > "jwt.hex" 
```
- Run this command to create a new execution layer account.
```javascript
geth account new --datadir "data" 
```
``data`` is folder PATH for your keystore will be stored.
- write custom genesis block 
```javascript
geth --datadir $HOME/testnet-list/data init $HOME/testnet-list/execution/genesis.json
```
``HOME/testnet-list/data`` is folder where you stored genesis block. Use your own folder setting if you want.

## ${\color{orange}GETH \space Execution-Layer}$   
#### ${\color{lightblue}Creating \space GETH-Service}$
- You have to replace this configuration with your own.<br>
```diff
--identity "use_your_own_moniker_name" \
```
```javascript
sudo tee /etc/systemd/system/gethd.service > /dev/null << EOF
[Unit]
Description=GETH Service
After=network-online.target
[Service]
User=$USER
ExecStart=geth \
--datadir "$HOME/testnet-list/data" \
--http --http.api="engine,eth,web3,net,admin" \
--ws --ws.api="engine,eth,web3,net" \
--http.port 8545 \
--http.addr 0.0.0.0 \
--http.corsdomain "*" \
--identity "<REPLACE-WITH-YOUR-MONIKER>" \
--networkid=1881 \
--syncmode=full \
--authrpc.jwtsecret="$HOME/testnet-list/jwt.hex" \
--authrpc.port 8551 \
--bootnodes "enode://0e2b41699b95e8c915f4f5d18962c0d2db35dc22d3abbebbd25fc48221d1039943240ad37a6e9d853c0b4ea45da7b6b5203a7127b5858c946fc040cace8d2d63@147.75.71.217:30303,enode://45b4fff6ab970e1e490deea8a5f960d806522fafdb33c8eaa38bc0ae970efc2256fc5746f0ecfec770af24c44864a3e6772a64f2e9f031f96fd4af7fd0483110@147.75.71.217:30304"
WorkingDirectory=$HOME
Restart=always
RestartSec=5
LimitNOFILE=10000
[Install]
WantedBy=multi-user.target
EOF
```
#### ${\color{lightblue}Starting \space GETH-Service}$
```javascript
sudo systemctl daemon-reload
sudo systemctl enable gethd
sudo systemctl start gethd && sudo journalctl -fu gethd -o cat
```
#### ${\color{lightblue}Add \space Peer-Node}$
```javascript
geth attach http://localhost:8545
```
After entering the Geth Interactive Console Javascript, you must add a Manual Peer & Trusted Peer of at least 1 to connect to the blockchain.
```javascript
admin.addPeer("enode://0e2b41699b95e8c915f4f5d18962c0d2db35dc22d3abbebbd25fc48221d1039943240ad37a6e9d853c0b4ea45da7b6b5203a7127b5858c946fc040cace8d2d63@147.75.71.217:30303")
admin.addTrustedPeer("enode://0e2b41699b95e8c915f4f5d18962c0d2db35dc22d3abbebbd25fc48221d1039943240ad37a6e9d853c0b4ea45da7b6b5203a7127b5858c946fc040cace8d2d63@147.75.71.217:30303")
```
Output: 
> True
### ${\color{lightblue}Get \space Bootnodes}$
Get the Bootnode Inside the Geth Interactive Console.
```javascript
admin.nodeInfo.enode
```
Example Output: 
```javascript
"enode://03fc89e2035b52a609715a15dacad4179f57c0b1e51b3464a931f0fa913b9169d06df1b23515f41e4ed6d9be0e50f33175cbf836e7b6738c62eee00ad45250b0@212.47.241.173:30303"
```
## ${\color{orange}Consensus \space Layer (BEACON-1)}$ 
#### ${\color{lightblue}Creating \space BEACON-1-Service}$
- Replace this configuration with your own.<br>
```diff
--graffiti "use_your_own_moniker_name"
--suggested-fee-recipient= use wallet when you created new account before.
```
```javascript
sudo tee /etc/systemd/system/beacon1.service > /dev/null << EOF
[Unit]
Description=Beacon-1 Consensus Layer
After=network-online.target
[Service]
User=$USER
ExecStart=lighthouse beacon \
--eth1 \
--http \
--testnet-dir $HOME/testnet-list/consensus \
--datadir "$HOME/testnet-list/beacon-1" \
--http-allow-sync-stalled \
--execution-endpoints http://127.0.0.1:8551 \
--http-port=5052 \
--enr-udp-port=9000 \
--enr-tcp-port=9000 \
--discovery-port=9000 \
--graffiti "<REPLACE-WITH-YOUR-MONIKER>" \
--execution-jwt "$HOME/testnet-list/jwt.hex" \
--suggested-fee-recipient=<REPLACE-WITH-YOUR-ADDRESS> 
WorkingDirectory=$HOME
Restart=always
RestartSec=5
LimitNOFILE=10000
[Install]
WantedBy=multi-user.target
EOF
```
#### ${\color{lightblue}Starting \space BEACON-1-Service}$
```javascript
sudo systemctl daemon-reload
sudo systemctl enable beacon1
sudo systemctl start beacon1 && sudo journalctl -fu beacon1 -o cat
```
#### ${\color{lightblue}Getting \space ENR-Key}$ 
```javascript
curl http://localhost:5052/eth/v1/node/identity | jq 
```

Example Output: 
```javascript

{
  "data": {
    "peer_id": "16Uiu2HAmGLNCWcsWgVR6Q2YvmGvGyyaEM3y5Tcq2rjmpv4dqo8JJ",
    "enr": "enr:-L24QCeN4AtWKWQfWMl9ubWXj_rWzxmVLG54qJtNQuXfEd9xPjPHrafoC3iyt_NUf4RNyiGaA-5yumvmSitnsd_Cx4SBiodhdHRuZXRziP__________hGV0aDKQWOqKlwJndzn__________4JpZIJ2NIJpcITUL_GtiXNlY3AyNTZrMaEDNqlqq2_MsiLqswykyIsqFIX10A-1COt29ZTpmlxf9teIc3luY25ldHMPg3RjcIIjKIN1ZHCCIyg",
    "p2p_addresses": [
      "/ip4/212.47.241.173/tcp/9000/p2p/16Uiu2HAmGLNCWcsWgVR6Q2YvmGvGyyaEM3y5Tcq2rjmpv4dqo8JJ"
    ],
    "discovery_addresses": [
      "/ip4/212.47.241.173/udp/9000/p2p/16Uiu2HAmGLNCWcsWgVR6Q2YvmGvGyyaEM3y5Tcq2rjmpv4dqo8JJ"
    ],
    "metadata": {
      "seq_number": "205",
      "attnets": "0xffffffffffffffff",
      "syncnets": "0x0f"
    }
  }
}
```
### ${\color{red}Save \space Your\space ENR - KEY!!}$  

## ${\color{orange}Othe \space Consensus \space Layer (BEACON-2)}$ 
#### ${\color{lightblue}Creating \space BEACON-2-Service}$
- Replace this configuration with your own.<br>
```diff
--enr-address <your-Node-IP-Address>
--boot-nodes="<REPLACE-WITH-YOUR-ENR-KEY>
--suggested-fee-recipient=<REPLACE-WITH-YOUR-ADDRESS>
```
```javascript
sudo tee /etc/systemd/system/beacon2.service > /dev/null << EOF
[Unit]
Description=Beacon-2 Consensus Layer
After=network-online.target
[Service]
User=$USER
ExecStart=lighthouse \
--testnet-dir $HOME/testnet-list/consensus
bn \
--datadir "$HOME/testnet-list/beacon-2" \
--eth1 \
--http \
--http-allow-sync-stalled \
--execution-endpoints "http://127.0.0.1:8551/" \
--eth1-endpoints "http://127.0.0.1:8545/" \
--http-address 0.0.0.0 \
--http-port 5053 \
--http-allow-origin="*" \
--listen-address 0.0.0.0 \
--enr-udp-port 9001 \
--enr-tcp-port 9001 \
--port 9001 \
--enr-address <your-Node-IP-Address> \
--execution-jwt "$HOME/testnet-list/jwt.hex" \
--boot-nodes="<REPLACE-WITH-YOUR-ENR-KEY>,enr:-MS4QHXShZPtKwtexK2p9yCxMxDwQ-EvdH_VemoxyVyweuaBLOC_8cmOzyx7Gy-q6-X8KGT1d_rhAn_ekXnhpCkA_REHh2F0dG5ldHOIAAAAAAAAAACEZXRoMpBMfxReAmd2k___________gmlkgnY0gmlwhJNLR9mJc2VjcDI1NmsxoQJB10N42nK6rr7Q_NIJNkJFi2uo6itMTOQlPZDcCy09T4hzeW5jbmV0c4gAAAAAAAAAAIN0Y3CCIyiDdWRwgiMo,enr:-MS4QEw_RpORuoXgJ0279QuVLLFAiXevNdYtU7vR8S1CY7X9CS6tceMbaxdIIJYRmHN43ClqHtE2b0H0maSb18cm9D0Hh2F0dG5ldHOIAAAAAAAAAACEZXRoMpBMfxReAmd2k___________gmlkgnY0gmlwhJNLR9mJc2VjcDI1NmsxoQOkQIyCVHLbLjIFMjqNSJEUsbYMe4Tsv9blUWvN6Rsft4hzeW5jbmV0c4gAAAAAAAAAAIN0Y3CCIymDdWRwgiMp,enr:-LS4QExQqM_G3y2CfedjrGEbapN5Vprdy7Iq2gzfylwLW8PQf4Tf82XnQxLg9PbH8QLwsMaoWwYjTo7xHQ4oy4eCn7kBh2F0dG5ldHOIAAAAAAAAAACEZXRoMpBMfxReAmd2k___________gmlkgnY0iXNlY3AyNTZrMaEDec2pARmw1GLJHiXIDaG-6J74gZ1SyDcF_CuVUzRsmX2Ic3luY25ldHMAg3RjcIIjKoN1ZHCCIyo" \
--suggested-fee-recipient=<REPLACE-WITH-YOUR-ADDRESS>
WorkingDirectory=$HOME
Restart=always
RestartSec=5
LimitNOFILE=10000
[Install]
WantedBy=multi-user.target
EOF
```
#### ${\color{lightblue}Starting \space BEACON-2-Service}$
```javascript
sudo systemctl daemon-reload
sudo systemctl enable beacon2
sudo systemctl start beacon2 && sudo journalctl -fu beacon2 -o cat
```
