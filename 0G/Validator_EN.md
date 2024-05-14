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

## Auto Node Installation
Run the following command to install node automatically.
```bash
curl -o zg_EN.sh https://gist.githubusercontent.com/botxx15/e8f60b199e74356ea8b8be532fb71880/raw/3675e7ab585b2d20092ea1eea90003b5d5b6a31e/zg_EN.sh && bash zg_EN.sh
```

You can press `ctrl+c` button to quit from logs.

> [!NOTE]
> Once the nodes are synchronized, you can proceed to step [11. Create Wallet for Validator](#11-Create-Wallet-For-Validator)

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
PEERS="9d7564df34efa146a94c073e5bf3f5e11f947b75@155.133.22.230:26656,a4055b828e59832c7a06d61fc51347755a160d0b@157.90.33.62:21656" && \
SEEDS="c4d619f6088cb0b24b4ab43a0510bf9251ab5d7f@54.241.167.190:26656,44d11d4ba92a01b520923f51632d2450984d5886@54.176.175.48:26656,f2693dd86766b5bf8fd6ab87e2e970d564d20aff@54.193.250.204:26656,f878d40c538c8c23653a5b70f615f8dccec6fb9f@54.215.187.94:26656" && \
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.0gchain/config/config.toml
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
  --gas-adjustment=1.4 \
  --fees=800ua0gi \
  -y
```
> [!CAUTION]
> Do not forget to save the `priv_validator_key.json` file located in $HOME/.0gchain/config/

-----------------------------------------------------------------

# Command List
## Managing keys
Generate new key
```
0gchaind keys add wallet
```

Recover key
```
0gchaind keys add wallet --recover
```

List all key
```
0gchaind keys list
```

Delete key
```
0gchaind keys delete wallet
```

Export key
```
0gchaind keys export wallet
```

Import key
```
0gchaind keys import wallet wallet.backup
```

Query wallet balances
```
0gchaind q bank balances $(0gchaind keys show wallet -a)
```

-----------------------------------------------------------------

## Pengelolaan Validator
Create validator
```
0gchaind tx staking create-validator \
--amount 1000000ua0gi \
--pubkey $(0gchaind tendermint show-validator) \
--moniker $MONIKER \
--identity "keybase-id" \
--details "detailed-info" \
--website "website-link" \
--security-contact "email-address" \
--chain-id $CHAIN_ID \
--commission-rate 0.10 \
--commission-max-rate 0.20 \
--commission-max-change-rate 0.01 \
--min-self-delegation 1 \
--from wallet \
--gas auto \
--gas-adjustment 1.4 \
--fees=800ua0gi \
-y
```

Edit validator
```
0gchaind tx staking edit-validator \
--new-moniker "nama-moniker" \
--identity "keybase-id" \
--details "detailed-info" \
--website "website-link" \
--security-contact "email-address" \
--chain-id $CHAIN_ID \
--commission-rate 0.10 \
--from wallet \
--gas auto \
--gas-adjustment 1.4 \
--fees=800ua0gi \
-y
```

Unjail validator
```
0gchaind tx slashing unjail --from wallet --chain-id $CHAIN_ID --gas auto --gas-adjustment 1.4 fees 800ua0gi -y
```

Validator jail reason
```
0gchaind q slashing signing-info $(0gchaind tendermint show-validator)
```

List active validator
```
0gchaind q staking validators -o json --limit=1000 \
| jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' \
| jq -r '.tokens + " - " + .description.moniker' \
| sort -gr | nl
```

List incative validator
```
0gchaind q staking validators -o json --limit=1000 \
| jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED")' \
| jq -r '.tokens + " - " + .description.moniker' \
| sort -gr | nl
```

View validator details
```
0gchaind q staking validator $(0gchaind keys show wallet --bech val -a) 
```

-----------------------------------------------------------------

## Managing Tokens
Withdraw reward from all validator
```
0gchaind tx distribution withdraw-all-rewards --from wallet --chain-id $CHAIN_ID --gas auto --gas-adjustment 1.4 fees 800ua0gi -y
```

Withdraw reward and commission
```
0gchaind tx distribution withdraw-rewards $(0gchaind keys show wallet --bech val -a) --commission --from wallet --chain-id $CHAIN_ID --gas auto --gas-adjustment 1.4 fees 800ua0gi -y
```

Delegate tokens to your validator
```
0gchaind tx staking delegate $(0gchaind keys show wallet --bech val -a) 1000000ua0gi --from wallet --chain-id $CHAIN_ID --gas auto --gas-adjustment 1.4 fees 800ua0gi -y
```

Delegate token to other validator, change `<to-valoper-address>` as you like
```
0gchaind tx staking delegate <to-valoper-address> 1000000ua0gi --from wallet --chain-id $CHAIN_ID --gas auto --gas-adjustment 1.4 fees 800ua0gi -y
```

Redelegate to another validator
```
0gchaind tx staking redelegate $(0gchaind keys show wallet --bech val -a) <to-valoper-address> 1000000ua0gi --from wallet --chain-id $CHAIN_ID --gas auto --gas-adjustment 1.4 fees 800ua0gi -y
```

Unbond token from your own validator
```
0gchaind tx staking unbond $(0gchaind keys show wallet --bech val -a) 1000000ua0gi --from wallet --chain-id $CHAIN_ID --gas auto --gas-adjustment 1.4 fees 800ua0gi -y
```

Send token to the wallet
```
0gchaind tx bank send wallet <to-wallet-address> 1000000ua0gi --from wallet --chain-id $CHAIN_ID --gas auto --gas-adjustment 1.4 fees 800ua0gi -y
```

-----------------------------------------------------------------

## Governance
Query list proposal
```bash
0gchaind query gov proposals
```

View proposal by ID
```bash
0gchaind query gov proposal 1
```

Vote option yes
```bash
0gchaind tx gov vote 1 yes --from wallet --chain-id $CHAIN_ID --gas auto --gas-adjustment 1.4 fees 800ua0gi -y
```

Vote option no
```bash
0gchaind tx gov vote 1 no --from wallet --chain-id $CHAIN_ID --gas auto --gas-adjustment 1.4 fees 800ua0gi -y
```

Vote option asbtain
```bash
0gchaind tx gov vote 1 abstain --from wallet --chain-id $CHAIN_ID --gas auto --gas-adjustment 1.4 fees 800ua0gi -y
```

Vote option NoWithVeto
```bash
0gchaind tx gov vote 1 NoWithVeto --from wallet --chain-id $CHAIN_ID --gas auto --gas-adjustment 1.4 fees 800ua0gi -y
```
-----------------------------------------------------------------

## Maintenance
Get validator information
```bash
0gchaind status 2>&1 | jq .validator_info
```

Get sync information
```bash
0gchaind status 2>&1 | jq .sync_info
```

Get node peer
```bash
echo $(0gchaind tendermint show-node-id)'@'$(curl -s ifconfig.me)':'$(cat $HOME/.0gchain/config/config.toml | sed -n '/Address to listen for incoming connection/{n;p;}' | sed 's/.*://; s/".*//')
```

Check validator keys
```bash
[[ $(0gchaind q staking validator $(0gchaind keys show wallet --bech val -a) -oj | jq -r .consensus_pubkey.key) = $(0gchaind status | jq -r .validator_info.PubKey.value) ]] && echo -e "\n\e[1m\e[32mTrue\e[0m\n" || echo -e "\n\e[1m\e[31mFalse\e[0m\n"
```

Get live peers
```bash
curl -sS http://localhost:23457/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'
```

Enable prometheus
```bash
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.0gchain/config/config.toml
```

Reset chain data
```bash
0gchaind tendermint unsafe-reset-all --keep-addr-book --home $HOME/.0gchain --keep-addr-book
```

> [!CAUTION]
> Before moving on to the next step, be aware that all chain data will be erased. **Ensure you've created a backup of your priv_validator_key.json!**

Remove node
```bash
cd $HOME
sudo systemctl stop 0gchaind
sudo systemctl disable 0gchaind
sudo rm /etc/systemd/system/0gchaind.service
sudo systemctl daemon-reload
sudo rm -f $(which 0gchaind)
sudo rm -rf $HOME/.0gchain
sudo rm -rf $HOME/0g-chain
sudo rm -rf $HOME/go
```

-----------------------------------------------------------------

<p align="center">
  &copy; 2024 BlockHub. All rights reserved.
</p>
