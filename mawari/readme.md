# Mawari Testnet

## Faucet and Guardian NFT

- Using metamask, create new wallet from metamask then claim the faucet here.
- Faucet: https://hub.testnet.mawari.net/

<img width="1023" height="301" alt="image" src="https://github.com/user-attachments/assets/0fbb79c7-5dd7-46ab-84a5-a724c5820c84" />

- After get 2 Mawari token, mint Guardiang NFT ; 
- Link: https://testnet.mawari.net/mint We Make 3 Mints.
<img width="1294" height="874" alt="image" src="https://github.com/user-attachments/assets/bc3efc75-ffb3-4a41-8ede-6f6a7626c3c4" />


# Setting up the Guardian Node 

### Docker Installation (if you not installing yet); 
```shell
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

```shell
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

```shell
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
### Adduser to docker group
```shell
sudo usermod -aG docker $USER
```

# Running Node
#### Create variable
```shell
export MNTESTNET_IMAGE=us-east4-docker.pkg.dev/mawarinetwork-dev/mwr-net-d-car-uses4-public-docker-registry-e62e/mawari-node:latest
export OWNER_ADDRESS=0xb123343(Replace with your wallet u just create with metamask before)
```
#### Create mawari directory
```shell
mkdir -p ~/mawari
```
#### Run Docker
```shell
docker run -d --pull always -v ~/mawari:/app/cache -e OWNERS_ALLOWLIST=$OWNER_ADDRESS $MNTESTNET_IMAGE
```

# Cheatsheet;

- Docker ID
```
docker ps -a
```
<img width="438" height="69" alt="image" src="https://github.com/user-attachments/assets/8401134f-225f-46c3-9264-4b33ebf2f41a" />


### Find your burner/operator wallet and add tokens to prepare for delegation.

- If you requested 2 tokens earlier, send 1 token from your main wallet (the one you created at the beginning of this guide) to your burner wallet.
- If you requested only 1 token earlier, go back to the Mawari chain Hub at https://hub.testnet.mawari.net/  and request an additional token directly for your burner wallet address.

To get the burner wallet address, check your node logs for the following output:
```shell
docker logs f9f11ca82335 | grep burner
```
<img width="991" height="66" alt="image" src="https://github.com/user-attachments/assets/f0b585d9-b10e-4a14-a2eb-438b624ba6a2" />


# Activating the Guardian Node Client
- To activate your Guardian Node, you must delegate it to the burner/operator wallet address of your machine (the address you retrieved in the previous step). To proceed, open the Mawari Guardian Dashboard at https://app.testnet.mawari.net/ 
- Connect your testing wallet created for this setup at the beginning of the guide:
<img width="768" height="483" alt="image" src="https://github.com/user-attachments/assets/dd971949-2ce2-43db-b813-c25b31e0a03d" />

- Select all the IDs, and click on “Delegate”, enter your burner/operator wallet address in the field, and confirm by clicking “Delegate” again.
<img width="1361" height="918" alt="image" src="https://github.com/user-attachments/assets/c9108259-4013-4a93-a2fc-76173a674f11" />
<img width="768" height="523" alt="image" src="https://github.com/user-attachments/assets/cba4574e-ed03-47e5-ad65-d425a1071c07" />

- Make sure you sign the transaction on your Wallet, so the delegation can effectively be initiated on-chain.
<img width="397" height="616" alt="image" src="https://github.com/user-attachments/assets/7a9785d4-b16f-4736-be38-36a7fe2ea514" />

e. Check the output of your node client. If you see messages in the expected format, it means your delegations have been successfully detected.

```
[DEBUG]	received delegation offers count	{"delegation offers": "1"}
[DEBUG]	received delegations offers from eth	{"length": 1}
[DEBUG]	received delegation offers	{"delegation offers": 1}
[DEBUG]	accepting delegation offer	{"hash": "cf9cede70c3934d7952af1e94d8ca631aff3528375cd94765abb66e11f8f1de9"}
[DEBUG]	received unfinished jobs count	{"delegations": "0"}
[DEBUG]	received unfinished jobs	{"jobs": 0}
[DEBUG]	transaction submitted	{"hash": "0x7f396478018f2b1e56cc2d0ea456ec15e6462936161fa2dcbf89bbb30d7c63b2"}
[INFO]	delegation offer accepted	{"hash": "cf9cede70c3934d7952af1e94d8ca631aff3528375cd94765abb66e11f8f1de9"}
```
- You can also confirm node status in the Guardian Dashboard. Open the Guardian Dashboard and make sure your node is displayed as running.
<img width="1350" height="721" alt="image" src="https://github.com/user-attachments/assets/15d83d73-6c8f-40e2-ae12-64b40914b1f0" />


