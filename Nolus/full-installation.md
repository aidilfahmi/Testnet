# Updating
```
# Clone project repository
cd $HOME
rm -rf nolus-core
git clone https://github.com/Nolus-Protocol/nolus-core.git
cd nolus-core
git checkout v0.2.1-testnet

# Build binaries
make build

# Prepare binaries for Cosmovisor
mkdir -p $HOME/.nolus/cosmovisor/upgrades/v0.2.1/bin
mv target/release/nolusd $HOME/.nolus/cosmovisor/upgrades/v0.2.1/bin/
rm -rf build
```
