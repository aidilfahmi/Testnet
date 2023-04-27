# $${\color{lightgreen}Elixir \space Node \space Installation}$$

## ${\color{lightblue}Optional \space Configuration}$
### Custom User

```javascript
sudo adduser elixir
sudo adduser elixir sudo
su - elixir
```
## ${\color{lightblue}Installing \space Dependencies}$
```javascript
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg lsb-release
```

## ${\color{lightblue}Installing \space Docker}$

### Skip if You Already Docker Installed
```javascript
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
## ${\color{lightblue}Installing \space Node}$
## Download Docker FIle
```javascript
wget https://files.elixir.finance/Dockerfile
```
### Edit Docker FIle
```javascript
nano Dockerfile
```
Edit this line 
ENV ADDRESS=Your_Wallet_Address
ENV PRIVATE_KEY=Private_Key
ENV VALIDATOR_NAME=Your_Discord_Username

### Compiling Node
```javascript
sudo docker kill ev
sudo docker rm ev
sudo docker pull elixirprotocol/validator:testnet-2
sudo docker build . -f Dockerfile -t elixir-validator
```
### Start Node
```javascript
docker run -d --restart unless-stopped --name ev elixir-validator
```
