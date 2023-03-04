# Daily Snapshot
### Stop the service and reset the data
```
sudo systemctl stop nolusd
cp $HOME/.nolus/data/priv_validator_state.json $HOME/.nolus/priv_validator_state.json.backup
rm -rf $HOME/.nolus/data
```
### Download Snapshot
```
SNAP_NAME=$(curl -s http://dnsarz.xyz/snapshot/nolus/ | egrep -o ">nolus-snapshot.*\.tar.lz4" | tr -d ">")
curl http://dnsarz.xyz/snapshot/planq/${SNAP_NAME} | lz4 -dc - | tar -xf - -C $HOME/.nolusd
mv $HOME/.nolus/priv_validator_state.json.backup $HOME/.nolus/data/priv_validator_state.json
```
### Restart the service and check the log
```
sudo systemctl restart nolusd && journalctl -u nolusd -f --no-hostname -o cat
```
