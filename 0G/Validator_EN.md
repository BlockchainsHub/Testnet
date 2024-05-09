![0G Github Banner](https://github.com/BlockchainsHub/Testnet/assets/77204008/34a32724-b411-41e4-8696-e390dfa01cab)

# 0G Node Guide
This guide will assist you in the installation process of the 0G node.

-----------------------------------------------------------------

## Required Specifications
| Required | Specifications |
|-|-
| CPU | 8 Cores |
| Memory | 64 GB |
| Storage | 1 TB NVMe SSD |
| Bandwidth | 100mbps |
| OS | Linux |

-----------------------------------------------------------------

## Public Endpoint
| Public | Endpoint |
|-|-
| RPC | https://og-testnet-rpc.blockhub.id |
| API | https://og-testnet-api.blockhub.id |
| gRPC | https://og-testnet-grpc.blockhub.id |

-----------------------------------------------------------------

## Manual Node Installation
### 1. Install Required Packages
```bash
sudo apt update && \
sudo apt install curl git jq build-essential gcc unzip wget lz4 -y
```

### 2. Install GO
```bash
cd $HOME && \
ver="1.22.0" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile && \
source ~/.bash_profile && \
go version
```

### 3. Build Binary
```bash
git clone -b v0.1.0 https://github.com/0glabs/0g-chain.git
./0g-chain/networks/testnet/install.sh
source .profile
```
### 4. Set Variables
You can make several required changes, such as changing `My_Node` in the variable `MONIKER="My_Node"` with any node name you want to use, and `wallet` in the variable `WALLET_NAME="wallet"` with any wallet name you want to use.
```bash
echo 'export MONIKER="My_Node"' >> ~/.bash_profile
echo 'export CHAIN_ID="zgtendermint_16600-1"' >> ~/.bash_profile
echo 'export WALLET_NAME="wallet"' >> ~/.bash_profile
source $HOME/.bash_profile
```

### 5. Initialize Node
```bash
cd $HOME
0gchaind config chain-id $CHAIN_ID
0gchaind init $MONIKER --chain-id $CHAIN_ID
```

### 6. Download genesis.json & addrbook.json files
```bash
curl -Ls https://snapshots.liveraven.net/snapshots/testnet/zero-gravity/genesis.json > $HOME/.0gchain/config/genesis.json

curl -Ls https://snapshots.liveraven.net/snapshots/testnet/zero-gravity/addrbook.json > $HOME/.0gchain/config/addrbook.json
```

Then verify if the genesis configuration file is correct.
```bash
0gchaind validate-genesis
```

### 7. Add seeds and peers to the config.toml file
```bash
PEERS="a4055b828e59832c7a06d61fc51347755a160d0b@157.90.33.62:21656" && \
SEEDS="c4d619f6088cb0b24b4ab43a0510bf9251ab5d7f@54.241.167.190:26656,44d11d4ba92a01b520923f51632d2450984d5886@54.176.175.48:26656,f2693dd86766b5bf8fd6ab87e2e970d564d20aff@54.193.250.204:26656,f878d40c538c8c23653a5b70f615f8dccec6fb9f@54.215.187.94:26656" && \
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.0gchaind/config/config.toml
```

### 8. Create a Service File
```bash
sudo tee /etc/systemd/system/0gchaind.service > /dev/null <<EOF
[Unit]
Description=0gchaind Validator Node
After=network.target

[Service]
User=root
ExecStart=/root/go/bin/0gchaind start
Restart=on-failure
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF
```

### 9. Reload Systemd Configuration
```bash
sudo systemctl daemon-reload
sudo systemctl enable 0gchaind 
sudo systemctl start 0gchaind
```

### 10. Check Logs
```bash
sudo journalctl -u 0gchaind -f -o cat
```

### 11. Create Wallet for Validator
```bash
0gchaind keys add $WALLET_NAME --eth
```
> [!CAUTION]
> **DO NOT FORGET TO SAVE YOUR SEED PHRASE!**

Export the private key and add it to Metamask.
```bash
0gchaind keys unsafe-export-eth-key $WALLET_NAME
```

To add the 0G network in Metamask, visit the [0G scanner website](https://scan-testnet.0g.ai/) and connect your wallet. The 0G network will be automatically added to your Metamask wallet.

### 12. Request Tokens via Faucet
Please click the button below to request from the faucet.

<a href="https://faucet.0g.ai/" target="_blank">
  <img src="https://github.com/BlockchainsHub/Testnet/assets/77204008/12866a81-cac7-451a-96b5-4b202e8f1194" alt="Faucet Button" width="150" height="36.94" border="10" />
</a>

### 13. Create a Validator
```bash
0gchaind tx staking create-validator \
  --amount=1000000ua0gi \
  --pubkey=$(0gchaind tendermint show-validator) \
  --moniker=$MONIKER \
  --chain-id=$CHAIN_ID \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1" \
  --from=$WALLET_NAME \
  --gas=auto \
  --gas-adjustment=1.4
  --fees=800ua0gi
  -y
```
> [!CAUTION]
> Do not forget to save the `priv_validator_key.json` file located in $HOME/.0gchain/config/

-----------------------------------------------------------------

<p align="center">
  &copy; 2024 BlockHub. All rights reserved.
</p>
