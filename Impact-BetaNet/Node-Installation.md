# $${\color{lightgreen}Manual \space Node \space Installation}$$

## ${\color{lightblue}Optional \space Configuration}$
### Custom User

```javascript
sudo adduser impact
sudo adduser impact sudo
```
login to user
```javascript
sudo - impact
```
 
## ${\color{orange}Preparing}$	
### Installing Dependencies

```javascript
sudo apt-get update
sudo apt install --assume-yes git clang curl libssl-dev llvm libudev-dev make protobuf-compiler
sudo apt install build-essential
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env
rustc --version
rustup default stable
rustup update
rustup update nightly
rustup target add wasm32-unknown-unknown --toolchain nightly
rustup show
rustup +nightly show
```
```
Select 1 when you see this Installation menu
==================================
Current installation options:


   default host triple: x86_64-unknown-linux-gnu
     default toolchain: stable (default)
               profile: default
  modify PATH variable: yes

1) Proceed with installation (default)
2) Customize installation
3) Cancel installation
>
=================================
```

### Install Binary
```javascript
rm -rf impactprotocol
git clone https://github.com/GlobalBoost/impactprotocol
cd impactprotocol
cargo build --release
```

### Generate the Mining Key
```javascript
$HOME/impactprotocol/target/release/impact generate-mining-key --chain=impact-testnet
```
Result
```
Public key: 0x000000000000000000000000000000000000000000
Secret seed: menemonic phrase
Address: 5DMKxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

## ${\color{orange}IMPORT-KEY}$

Edit :<br>
suri : Replace with your mnemonic<br>
node_name : replace with your node name<br>
When asking for password, then write your password
```
Input your key
$HOME/impactprotocol/target/release/impact key insert \
  --base-path ~/.impactnode01 \
  --chain=impact-testnet \
  --scheme Ed25519 \
  --suri "CHANGE_THIS_WITH_YOUR_SEED_PHRASE//node_name///impact" \
  --password-interactive \
  --key-type gran
$HOME/impactprotocol/target/release/impact key insert \
  --base-path ~/.impactnode01 \
  --chain=impact-testnet \
  --scheme Sr25519 \
  --suri "CHANGE_THIS_WITH_YOUR_SEED_PHRASE//node_name///impact" \
  --password-interactive \
  --key-type imon
$HOME/impactprotocol/target/release/impact key insert \
  --base-path ~/.impactnode01 \
  --chain=impact-testnet \
  --scheme Sr25519 \
  --suri "CHANGE_THIS_WITH_YOUR_SEED_PHRASE//node_name///impact" \
  --password-interactive \
  --key-type auth
```

## ${\color{orange}CREATE-SERVICE}$
Edit :<br>
--author : Public key<br>
--name : your node name<br>
--password : Password that you're using for Input Key<br>

```javascript
sudo tee /etc/systemd/system/impact.service > /dev/null << EOF
[Unit]
Description=Impact Protocol Node
After=network-online.target

[Service]
User=$USER
Restart=on-failure
RestartSec=10
ExecStart=$HOME/impactprotocol/target/release/impact \
--base-path /tmp/impactnode \
--chain=impact-testnet \
--port 30333 \
--ws-port 9945 \
--rpc-port 9933 \
--telemetry-url "wss://telemetry.polkadot.io/submit/ 0" \
--validator \
--author Public_key \
--rpc-methods Unsafe \
--name "your_name" \
--password your_password

[Install]
WantedBy=multi-user.target
EOF
```

### Register And Start Service
```javascript
sudo systemctl enable impact
sudo systemctl daemon-reload
sudo systemctl start impact && sudo journalctl -fu impact -o cat
```


### Restart Node and Check the logs
```javascript
sudo systemctl restart impact && sudo journalctl -fu impact -o cat
```
### Check Your Node on Telementry
```javascript
https://telemetry.polkadot.io/#list/0x1133ca761f24222cb0811f34641dba07acd88c77bd9f30a23a99c2cba233cb91
```
