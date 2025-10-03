# Mawari Network TestNet - Setting up and running the Guardian Node application

This guide outlines the steps to set up and run the Guardian Node client application on the Mawari Network TestNet.

## 1. Setting up Wallet, Tokens, and Guardian NFT

1. **Create a Wallet**  
   If you don’t have a wallet, follow [this detailed guide](https://docs.mawari.net/wallet-setup-guide) to set one up.

2. **Access the Mawari Network TestNet**  
   Visit the [Mawari Network TestNet page](https://testnet.mawari.net/) and perform the following steps:

   - **Connect Your Wallet**: At the top of the screen, connect your wallet and switch to the Mawari Network TestNet to enable the following steps.
   - **Copy Your Wallet Address**: Click the button at the top right to copy your wallet address.
   - **Fund Your Wallet**: Go to the [Mawari Chain Hub](https://hub.testnet.mawari.net/), paste your wallet address into the Faucet field, and request testing MAWARI tokens.  
     **Note**: You can request up to 2 tokens on the TestNet. Start by adding them to the main wallet created earlier. Later, you’ll need to transfer 1 token from the main wallet to the burner wallet (also known as the operator wallet).
   - **Mint Guardian NFTs**: Return to the [Mawari Network TestNet page](https://testnet.mawari.net/) and mint up to 3 testing Guardian NFTs.

These steps prepare your account for setting up and running the Guardian Node client.

## 2. Setting up the Guardian Node Client Application

1. **Install Docker**  
   Ensure Docker is installed on your computer. If not, follow the [Docker installation guide](https://docs.docker.com/engine/install/).

2. **Open a Terminal**  
   Open the Console/Terminal app on macOS/Linux or PowerShell on Windows to run the commands for setting up the Guardian Node client as a container application.

3. **Set Environment Variables**  
   Configure the following environment variables based on your operating system:

   ### Linux and macOS

   ```bash
   # Set your node image name
   export MNTESTNET_IMAGE=us-east4-docker.pkg.dev/mawarinetwork-dev/mwr-net-d-car-uses4-public-docker-registry-e62e/mawari-node:latest
   ```

   ```bash
   # Set your owner wallet address from which Guardian NFTs will be delegated
   export OWNER_ADDRESS=0x123abc
   ```

   ### Windows

   ```powershell
   set "MNTESTNET_IMAGE=us-east4-docker.pkg.dev/mawarinetwork-dev/mwr-net-d-car-uses4-public-docker-registry-e62e/mawari-node:latest"
   ```

   ```powershell
   set "OWNER_ADDRESS=0x123abc"
   ```

4. **Run the Client**  
   Execute the following command to create a folder and start the Docker container:

   ### Linux and macOS

   ```bash
   mkdir -p ~/mawari && docker run --pull always -v ~/mawari:/app/cache -e OWNERS_ALLOWLIST=$OWNER_ADDRESS $MNTESTNET_IMAGE
   ```

   ### Windows

   ```powershell
   mkdir %USERPROFILE%\mawari 2>nul & docker run --pull always -v %USERPROFILE%\mawari:/app/cache -e OWNERS_ALLOWLIST=%OWNER_ADDRESS% %MNTESTNET_IMAGE%
   ```

   You’ll see logs similar to the following:

   ```log
   2025-09-09T13:54:16.086Z	[INFO]	loaded config	{"config": {"network": "ethereum", "heartbeatPeriodSeconds": 43200, "checkJobsPeriodSeconds": 10, "ownersAllowList": ["0xF9c7082CA8a6A882859f16661701c1ad65d12812"], "checkDelegationOfferPeriodSeconds": 10, "disableHeartbeats": false, "disableAutomaticDelegations": false, "eth.rpcNodes": ["https://mawari-network-testnet.rpc.caldera.xyz/http"], "eth.privateKey": "...", "eth.chainId": 629274, "eth.registry": "0xB33d2D2a2c84b3a1DEf601608a1F019213a02774", "eth.paginationChunkSize": 100, "eth.gasLimit": 6000000}}
   2025-09-09T13:54:16.086Z	[DEBUG]	initializing node ...
   2025-09-09T13:54:16.088Z	[DEBUG]	generating burner wallet
   2025-09-09T13:54:16.090Z	[DEBUG]	node cache does not exist, initializing empty node cache
   2025-09-09T13:54:16.098Z	[DEBUG]	burner wallet cache written	{"file": "/app/cache/flohive-cache.json"}
   2025-09-09T13:54:16.098Z	[DEBUG]	generated burner wallet	{"address": "<your burner wallet appears here>"}
   2025-09-09T13:54:16.102Z	[INFO]	Using burner wallet	{"address": "<your burner wallet appears here>"}
   2025-09-09T13:54:16.106Z	[DEBUG]	loaded plugin	{"path": "plugins/signature-scan-demo.so", "supported types": ["signature-scan-demo"]}
   2025-09-09T13:54:17.403Z	[DEBUG]	using delegation hub	{"address": "0x3F1BD1Abc350eD6313Ff7Eaab561DCAbbcc61071"}
   2025-09-09T13:54:17.404Z	[DEBUG]	starting delegations offer check
   2025-09-09T13:54:17.683Z	[DEBUG]	received delegation offers count	{"delegation offers": "0"}
   2025-09-09T13:54:17.683Z	[DEBUG]	received delegation offers	{"delegation offers": 0}
   2025-09-09T13:54:17.683Z	[DEBUG]	successfully checked delegation offers
   2025-09-09T13:54:17.683Z	[DEBUG]	starting heartbeats
   2025-09-09T13:54:18.300Z	[DEBUG]	sending heartbeat	{"tokenIds": []}
   2025-09-09T13:54:18.300Z	[WARN]	no delegations, skipping heartbeat
   ```

5. **Fund the Burner/Operator Wallet**  
   Locate your burner wallet address in the node logs, specifically in the line:

   ```log
   [INFO]  Using burner wallet {"address": "<your burner wallet appears here>"}
   ```

   - If you requested 2 tokens earlier, transfer 1 token from your main wallet to the burner wallet.
   - If you requested only 1 token, visit the [Mawari Chain Hub](https://hub.testnet.mawari.net/) and request an additional token directly for your burner wallet address.

## 3. Activating the Guardian Node Client

1. **Delegate to the Burner/Operator Wallet**  
   Open the [Mawari Guardian Dashboard](https://app.testnet.mawari.net/) and follow these steps:

   - Connect your testing wallet created at the beginning of this guide.
   - Select all the NFT IDs, click "Delegate," enter your burner/operator wallet address, and confirm by clicking "Delegate" again.
   - Sign the transaction in your wallet to initiate the delegation on-chain.

2. **Verify Delegation**  
   Check the node client logs for messages indicating successful delegation detection, such as:

   ```log
   [DEBUG]	received delegation offers count	{"delegation offers": "1"}
   [DEBUG]	received delegations offers from eth	{"length": 1}
   [DEBUG]	received delegation offers	{"delegation offers": 1}
   [DEBUG]	accepting delegation offer	{"hash": "cf9cede70c3934d7952af1e94d8ca631aff3528375cd94765abb66e11f8f1de9"}
   [DEBUG]	received unfinished jobs count	{"delegations": "0"}
   [DEBUG]	received unfinished jobs	{"jobs": 0}
   [DEBUG]	transaction submitted	{"hash": "0x7f396478018f2b1e56cc2d0ea456ec15e6462936161fa2dcbf89bbb30d7c63b2"}
   [INFO]	delegation offer accepted	{"hash": "cf9cede70c3934d7952af1e94d8ca631aff3528375cd94765abb66e11f8f1de9"}
   ```

3. **Confirm Node Status**  
   Open the [Mawari Guardian Dashboard](https://app.testnet.mawari.net/) to verify that your node is displayed as running.

## Feedback and Support

If you encounter issues or want to share feedback about setting up the Guardian Node client on TestNet, please fill out [this form](https://docs.mawari.net/feedback-form).

*Last updated: October 2025*
