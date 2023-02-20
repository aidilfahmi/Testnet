# Andromeda TestNet
![image](https://user-images.githubusercontent.com/16186519/220021849-1378e536-0ab2-4e53-8acb-c4bc3eab18c1.png)

## Setting up Variable

```
WALLET=wallet
BINARY=andromedad
CHAIN=galileo-3
FOLDER=.andromedad
DENOM=uandr
REPO=https://github.com/andromedaprotocol/andromedad.git
GENESIS=https://raw.githubusercontent.com/andromedaprotocol/testnets/galileo-3/genesis.json
ADDRBOOK=https://snapshots.nodeist.net/t/andromeda/addrbook.json
PORT=54

echo "export WALLET=${WALLET}" >> $HOME/.bash_profile
echo "export BINARY=${BINARY}" >> $HOME/.bash_profile
echo "export DENOM=${DENOM}" >> $HOME/.bash_profile
echo "export CHAIN=${CHAIN}" >> $HOME/.bash_profile
echo "export FOLDER=${FOLDER}" >> $HOME/.bash_profile
echo "export REPO=${REPO}" >> $HOME/.bash_profile
echo "export GENESIS=${GENESIS}" >> $HOME/.bash_profile
echo "export ADDRBOOK=${ADDRBOOK}" >> $HOME/.bash_profile
echo "export PORT=${PORT}" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
