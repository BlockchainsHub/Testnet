![Kamus Crypto](https://github.com/Kamus-Crypto/Testnet/assets/77204008/5a53beb5-11e6-485a-af72-eb6983bd5343)

# Artela Network Node Guide
This document provides an introduction to joining the Artela Testnet.

## Hardware recommended requirements:
It is recommended to use **ubuntu** operating system.
* 8 CPU Cores
* 16GB of Memory
* 1TB SSD
* 200mbps Network Bandwidth

## Run a full node

### 1. Preparation
```
apt update && apt upgrade -y
apt install curl iptables build-essential git wget jq make gcc nano tmux htop lz4 nvme-cli pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y
```

#### Install Go
You can skip this if you already installed Go. Run command ```go version``` on your terminal to verify it.
```
ver="1.20.3"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```

### 2. Installation
#### Clone and build the code
Make sure you install the latest version. Modify the ```git checkout vx.x.x``` with the latest version of Artela. You can find the latest version **[here](https://github.com/artela-network/artela)**.
```
cd $HOME
rm -rf artela
git clone https://github.com/artela-network/artela
cd artela
git checkout vx.x.x
make install
```
Example:
```
cd $HOME
rm -rf artela
git clone https://github.com/artela-network/artela
cd artela
git checkout v0.4.7-rc4
make install
```

#### Node chain id configuration
```
artelad config chain-id artela_11822-1
```

#### Initialize node
Modify the ```$MONIKER``` with your node name.
```
artelad init "$MONIKER" --chain-id artela_11822-1
```
Example:
```
artelad init kamuscrypto --chain-id artela_11822-1
```

#### Add genesis file and addrbook
```
wget -qO $HOME/.artelad/config/genesis.json https://docs.artela.network/assets/files/genesis-314f4b0294712c1bc6c3f4213fa76465.json
wget -qO $HOME/.artelad/config/addrbook.json https://snapshots.theamsolutions.info/artela-addrbook.json
```

#### Seeds and peers configuration
```
SEEDS="bec6934fcddbac139bdecce19f81510cb5e02949@47.254.24.106:26656,32d0e4aec8d8a8e33273337e1821f2fe2309539a@47.88.58.36:26656,1bf5b73f1771ea84f9974b9f0015186f1daa4266@47.251.14.47:26656"
PEERS="a996136dcb9f63c7ddef626c70ef488cc9e263b8@144.217.68.182:22256,de5612c035bd1875f0bd36d7cbf5d660b0d1e943@5.78.64.11:26656,bec6934fcddbac139bdecce19f81510cb5e02949@47.254.24.106:26656,30fb0055aced21472a01911353101bc4cd356bb3@47.89.230.117:26656,a03ae11a093c67e2554b73d174c4168fe715af10@57.128.103.184:26656,146d6011cce0423f564c9277c6a3390657c53730@157.90.226.23:26656,0188a9bcff4f411b29dbddda527d77803396e1c6@185.245.182.180:26656,b23bc610c374fd071c20ce4a2349bf91b8fbd7db@65.108.72.233:11656,aa416d3628dcce6e87d4b92d1867c8eca36a70a7@47.254.93.86:26656,978dee673bd447147f61aa5a1bdaabdfb8f8b853@47.88.57.107:26656,35ce36af33e289a29787eedb3127d21bf10edcff@81.0.218.194:45656,32d0e4aec8d8a8e33273337e1821f2fe2309539a@47.88.58.36:26656,1b73ac616d74375932fb6847ec67eee4a98174e9@116.202.85.52:25556,9e2fbfc4b32a1b013e53f3fc9b45638f4cddee36@47.254.66.177:26656,b23bc610c374fd071c20ce4a2349bf91b8fbd7db@65.108.72.233:11656,30fb0055aced21472a01911353101bc4cd356bb3@47.89.230.117:26656,9e2fbfc4b32a1b013e53f3fc9b45638f4cddee36@47.254.66.177:26656,978dee673bd447147f61aa5a1bdaabdfb8f8b853@47.88.57.107:26656,aa416d3628dcce6e87d4b92d1867c8eca36a70a7@47.254.93.86:26656"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.artelad/config/config.toml
```

#### Pruning, Prometheus, gas prices and indexer configuration
```
sed -i 's|^pruning *=.*|pruning = "custom"|g' $HOME/.artelad/config/app.toml
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $HOME/.artelad/config/app.toml
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|g' $HOME/.artelad/config/app.toml
sed -i 's|^snapshot-interval *=.*|snapshot-interval = 0|g' $HOME/.artelad/config/app.toml

sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.025art"|g' $HOME/.artelad/config/app.toml
sed -i 's|^prometheus *=.*|prometheus = true|' $HOME/.artelad/config/config.toml
sed -i 's|^indexer *=.*|indexer = "null"|' $HOME/.artelad/config/config.toml
```

#### Artelad service configuration
```
sudo tee /etc/systemd/system/artelad.service > /dev/null << EOF
[Unit]
Description=artela node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which artelad) start
Restart=on-failure
RestartSec=10
LimitNOFILE=10000
[Install]
WantedBy=multi-user.target
EOF
```

#### Download latest snapshot
```
artelad tendermint unsafe-reset-all --home $HOME/.artelad --keep-addr-book

curl -L https://t-ss.nodeist.net/artela/snapshot_latest.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.artelad --strip-components 2
```

#### Start the node
```
sudo systemctl daemon-reload
sudo systemctl enable artelad
sudo systemctl start artelad
```

#### Check Log
```
sudo journalctl -fu artelad -o cat
```

## Validator Guide
Before setting up a validator node, make sure you have completed the **[Run a Full Node](https://github.com/Kamus-Crypto/Testnet/edit/botxx15-artela-guide/Artela%20Network/Guide_EN.md#run-a-full-node)** guide.

### Create your validator operator account
Replace ```<account name>``` with your validator operator account name.
```
artelad keys add <account name>
```
Example:
```
artelad keys add kamuscrypto
```
Enter your password and don't forget to **securely save your mnemonic phrase**. Becareful when you enter your password, it's not gonna show up on the CLI when you input your password.

![Validator Guide Artela](https://github.com/Kamus-Crypto/Testnet/assets/77204008/9f25478d-0de9-4bb4-ad3f-9eede1749b61)

### Convert bech32(cosmos format) address to hex(evm format) address
Before requesting a faucet you have to convert your Artelar address to EVM address format.
```
artelad debug addr <betch32 address/Artela address>
```
Example:
```
artelad debug addr art1f491dnx3kdqplfyjjpwl5cap4x0nlsgtcvvyca
```
![Validator Guide Art](https://github.com/Kamus-Crypto/Testnet/assets/77204008/699fb5b6-d38f-41f0-8d0b-fa9c4327bd1b)

### Request Faucet
Join **[Artela Network Discord](https://discord.gg/artela)** server to request faucet. After verify yourself, go to **#🚰|testnet-faucet** and enter the command below to request faucet.
```
$request <your converted EVM addres>
```
Example:
```
$request 0xCd62eFd6959612b9196C1a22EaCE66B48cFe9953
```

### Create your validator
```
artelad tx staking create-validator \
--amount 1000000000000000000uart \
--pubkey=$(artelad tendermint show-validator) \
--moniker "your validator name" \
--website "your website" \
--details "your validator details" \
--commission-rate "0.10" \
--commission-max-rate "0.20" \
--commission-max-change-rate "0.01" \
--min-self-delegation 1 \
--gas auto \
--gas-adjustment 1.5 \
--chain-id <chain_id> \
--from <your validator account name> \
-y
```
Example:
```
artelad tx staking create-validator \
--amount 1000000000000000000uart \
--pubkey=$(artelad tendermint show-validator) \
--moniker "kamuscrypto" \
--website "https://kamuscrypto.super.site" \
--details "Indonesia crypto community" \
--commission-rate "0.10" \
--commission-max-rate "0.20" \
--commission-max-change-rate "0.01" \
--min-self-delegation 1 \
--gas auto \
--gas-adjustment 1.5 \
--chain-id artela_11822-1 \
--from kamuscrypto \
-y
```

## Useful commands
### Restart Service
```
sudo systemctl restart artelad
```

### Delegate to yourself
You can change the ```wallet``` to your wallet name. You can see the example for reference.
```
artelad tx staking delegate $(artelad keys show wallet --bech val -a) 1000000000000000000uart --from wallet --chain-id artela_11822-1 --gas-prices 0.02uart  --gas-adjustment 1.5 --gas auto -y
```
Example:
```
artelad tx staking delegate $(artelad keys show kamuscrypto --bech val -a) 1000000000000000000uart --from kamuscrypto --chain-id artela_11822-1 --gas-prices 0.02uart  --gas-adjustment 1.5 --gas auto -y
```

### Remove Node
```
cd $HOME
sudo systemctl stop artela
sudo systemctl disable artela
sudo rm /etc/systemd/system/artela.service
sudo systemctl daemon-reload
sudo rm -f $(which artela)
sudo rm -rf $HOME/.artelad
sudo rm -rf $HOME/artela
sudo rm -rf $HOME/go
```
