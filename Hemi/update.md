# Update Binary v0.4.4
**1. Download Binaries**
```bash
cd $HOME
wget https://github.com/hemilabs/heminetwork/releases/download/v0.4.4/heminetwork_v0.4.4_linux_amd64.tar.gz
```

**2. Extract Binaries**
```bash
sudo systemctl stop hemi
rm -fr heminetwork
tar -zxvf heminetwork_v0.4.4_linux_amd64.tar.gz && rm heminetwork_v0.4.4_linux_amd64.tar.gz
mv heminetwork_v0.4.4_linux_amd64.tar.gz heminetwork
```

## Restart Service
```bash
sudo systemctl restart hemi && sudo journalctl -fu hemi -o cat
```
