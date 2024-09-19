# Hemi Miner Incentivized Testnet

![image](https://github.com/user-attachments/assets/996c7d95-8be3-457b-a920-270fc337c6e1)
> Hemi is a modular protocol for superior scaling, security, and interoperability, powered by Bitcoin and Ethereum.
 
#

* You can run miner using web browser on windows here: https://pop-miner.hemi.xyz/

## Custom User (optional)
```bash
sudo adduser hemi
sudo adduser hemi sudo
su - hemi
```
## Install Miner on Linux
**1. Download Binaries**
```bash
wget https://github.com/hemilabs/heminetwork/releases/download/v0.4.3/heminetwork_v0.4.3_linux_amd64.tar.gz
```

**2. Extract Binaries**
```bash
tar -xvf heminetwork_v0.4.3_linux_amd64.tar.gz && rm heminetwork_v0.4.3_linux_amd64.tar.gz
mv heminetwork_v0.4.3_linux_amd64 heminetwork
cd heminetwork
```

## Wallet
**1. Create tBTC wallet**
```bash
./keygen -secp256k1 -json -net="testnet" > ~/popm-address.json
```

**2. Get tBTC address**
```bash
cat $HOME/popm-address.json
```
* Save the output
* `pubkey_hash` is your tBTC address

**3. Fund your wallet**
* Get tBTC faucet in [Discord](https://discord.gg/hemixyz) for your bitcoin wallet
* Check your txhash in explorer until it get CONFIRMED

## Start Miner
**Create Service**
```bash
sudo tee /etc/systemd/system/hemi.service > /dev/null <<EOF
[Unit]
Description=Mining Cuk
After=network-online.target
StartLimitIntervalSec=0

[Service]
User=$USER
Restart=always
RestartSec=3
LimitNOFILE=65535
ExecStart=$HOME/heminetwork/popmd
Environment="POPM_BTC_PRIVKEY=your_private_key"
Environment="POPM_STATIC_FEE=50"
Environment="POPM_BFG_URL=wss://testnet.rpc.hemi.network/v1/ws/public"
[Install]
WantedBy=multi-user.target
EOF
```

**Start your node**
```bash
sudo systemctl restart hemi && sudo journalctl -fu hemi -o cat
```

![image](https://github.com/user-attachments/assets/76dc9867-a0b3-4d11-9baf-cd1d5a94f695)

# View your points
* Currently, the tHEMI token payout serves as Hemi PoP mining points. You will get tHEMI tokens for each block you min with your Hemi Pop Node
* To view your tHEMI balance, you may need to add the contract address `0x4200000000000000000000000000000000000042` as a "custom token" in your metamask wallet on Hemi Sepolia Network.
* You can also import your `private_key` here to check tHEMI balance: https://pop-miner.hemi.xyz/
* Gaining tHEMI is delayed rn, you can check later
