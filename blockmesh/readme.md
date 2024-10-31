<div align="center">
<img src="https://github.com/user-attachments/assets/c9814751-e3d7-4b17-afaf-29a3ee62be48" width="200">
</div>

# Blockmesh Installation

## Register Account

[Register](https://app.blockmesh.xyz/register?invite_code=dnsarz)

## Installing Docker
```bash
cd $HOME
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```
```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

## Installing Binary
```bash
cd $HOME
curl -L https://github.com/block-mesh/block-mesh-monorepo/releases/download/v0.0.316/blockmesh-cli-x86_64-unknown-linux-gnu.tar.gz -o blockmesh-cli.tar.gz
tar -xzf blockmesh-cli.tar.gz
```

## Running
### Create Parameter
```bash
EMAIL="email_register"
PASSWORD="password email register"
```
### Run Docker
```bash
sudo docker run -it --rm \
    --name blockmesh-cli-container \
    -v $(pwd)/target/release:/app \
    -e EMAIL="$EMAIL" \
    -e PASSWORD="$PASSWORD" \
    --workdir /app \
    ubuntu:22.04 ./blockmesh-cli --email "$EMAIL" --password "$PASSWORD"
```
### Check Log
```bash
sudo docker logs -f blockmesh-cli-container
```
