![Initia Banner 3000x1000](https://github.com/BlockchainsHub/Testnet/assets/77204008/1ea3aade-f38d-4057-8e6e-ab349bf0ab7e)

# Initia Node Guide
This guide will assist you in the installation process of the Initia node.

-----------------------------------------------------------------

## Required Specifications
| Required | Specifications |
|-|-
| CPU | 4 Cores |
| Memory | 16 GB |
| Storage | 1 TB NVMe SSD |
| Bandwidth | 100mbps |
| OS | Linux |

-----------------------------------------------------------------

## Public Endpoint
| Public | Endpoint |
|-|-
| RPC | https://initia-testnet-rpc.blockhub.id |
| API | https://initia-testnet-api.blockhub.id |
| gRPC | https://initia-testnet-grpc.blockhub.id |

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
git clone https://github.com/initia-labs/initia.git
cd initia
git checkout v0.2.12
make install
```

### 4. Set Variables
You can make several required changes, such as changing `My_Node` in the variable `MONIKER="My_Node"` with any node name you want to use, and `wallet` in the variable `WALLET_NAME="wallet"` with any wallet name you want to use.
```bash
echo 'export MONIKER="My_Node"' >> ~/.bash_profile
echo 'export CHAIN_ID="initiation-1"' >> ~/.bash_profile
echo 'export WALLET_NAME="wallet"' >> ~/.bash_profile
source $HOME/.bash_profile
```

### 5. Initialize Node
```bash
cd $HOME
initiad init $MONIKER --chain-id $CHAIN_ID
```

### 6. Download genesis.json
```bash
wget https://initia.s3.ap-southeast-1.amazonaws.com/initiation-1/genesis.json -O $HOME/.initia/config/genesis.json
```

### 7. Add seeds and peers to the config.toml file
```bash
PEERS="40d3f977d97d3c02bd5835070cc139f289e774da@168.119.10.134:26313,841c6a4b2a3d5d59bb116cc549565c8a16b7fae1@23.88.49.233:26656,e6a35b95ec73e511ef352085cb300e257536e075@37.252.186.213:26656,2a574706e4a1eba0e5e46733c232849778faf93b@84.247.137.184:53456,ff9dbc6bb53227ef94dc75ab1ddcaeb2404e1b0b@178.170.47.171:26656,edcc2c7098c42ee348e50ac2242ff897f51405e9@65.109.34.205:36656,07632ab562028c3394ee8e78823069bfc8de7b4c@37.27.52.25:19656,028999a1696b45863ff84df12ebf2aebc5d40c2d@37.27.48.77:26656,140c332230ac19f118e5882deaf00906a1dba467@185.219.142.119:53456,1f6633bc18eb06b6c0cab97d72c585a6d7a207bc@65.109.59.22:25756,065f64fab28cb0d06a7841887d5b469ec58a0116@84.247.137.200:53456,767fdcfdb0998209834b929c59a2b57d474cc496@207.148.114.112:26656,093e1b89a498b6a8760ad2188fbda30a05e4f300@35.240.207.217:26656,12526b1e95e7ef07a3eb874465662885a586e095@95.216.78.111:26656" && \
SEEDS="2eaa272622d1ba6796100ab39f58c75d458b9dbc@34.142.181.82:26656,c28827cb96c14c905b127b92065a3fb4cd77d7f6@testnet-seeds.whispernode.com:25756" && \
sed -i \
    -e "s/^seeds *=.*/seeds = \"$SEEDS\"/" \
    -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" \
    "$HOME/.initia/config/config.toml"
```

### 8. Set minimum gas price
```bash
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.15uinit,0.01uusdc\"/" $HOME/.initia/config/app.toml
```

### 9. Set custom pruning
```bash
sed -i \
    -e "s/^pruning *=.*/pruning = \"custom\"/" \
    -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" \
    -e "s/^pruning-interval *=.*/pruning-interval = \"10\"/" \
    "$HOME/.initia/config/app.toml"
```

### 10. Create a Service File
```bash
sudo tee /etc/systemd/system/initiad.service > /dev/null <<EOF
[Unit]
Description=initia node
After=network-online.target

[Service]
User=$USER
ExecStart=$(which initiad) start
Restart=on-failure
RestartSec=10
LimitNOFILE=10000

[Install]
WantedBy=multi-user.target
EOF
```

### 11. Reload Systemd Configuration
```bash
sudo systemctl daemon-reload
sudo systemctl enable initiad 
sudo systemctl start initiad
```

### 12. Check Logs
```bash
sudo journalctl -u initiad -f -o cat
```

### 13. Create Wallet for Validator
```bash
initiad keys add $WALLET_NAME
```
> [!CAUTION]
> **DO NOT FORGET TO SAVE YOUR SEED PHRASE!**

### 14. Request Tokens via Faucet
Please click the button below to request from the faucet.

<a href="https://faucet.testnet.initia.xyz/" target="_blank">
  <img src="https://github.com/BlockchainsHub/Testnet/assets/77204008/58473daf-a9e6-4fba-9449-ef7fff50ed1a" alt="Initia Faucet Button" width="150" height="36.94" border="10" />
</a>

### 15. Create a Validator
```bash
initiad tx mstaking create-validator \
  --amount=10000000uinit \
  --pubkey=$(initiad tendermint show-validator) \
  --moniker=$MONIKER \
  --chain-id=$CHAIN_ID \
  --identity="keybase-id" \
  --website="website-link" \
  --details="info-detail" \
  --security-contact="email-address" \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --from=$WALLET_NAME \
  --gas=auto \
  --gas-adjustment=1.4 \
  --fees=300000uinit \
  -y
```
> [!CAUTION]
> Do not forget to save the `priv_validator_key.json` file located in $HOME/.initia/config/

-----------------------------------------------------------------

# Command List
## Managing keys
Generate new key
```
initiad keys add wallet
```

Recover key
```
initiad keys add wallet --recover
```

List all key
```
initiad keys list
```

Delete key
```
initiad keys delete wallet
```

Export key
```
initiad keys export wallet
```

Import key
```
initiad keys import wallet wallet.backup
```

Query wallet balances
```
initiad q bank balances $(initiad keys show wallet -a)
```

-----------------------------------------------------------------

## Validator Management
Create validator
```
initiad tx mstaking create-validator \
--amount 1000000ua0gi \
--pubkey $(initiad tendermint show-validator) \
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
initiad tx mstaking edit-validator \
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
initiad tx slashing unjail --from wallet --chain-id $CHAIN_ID --gas auto --gas-adjustment 1.4 fees 800ua0gi -y
```

Validator jail reason
```
initiad q slashing signing-info $(initiad tendermint show-validator)
```

List active validator
```
initiad q mstaking validators -o json --limit=1000 \
| jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' \
| jq -r '.tokens + " - " + .description.moniker' \
| sort -gr | nl
```

List incative validator
```
initiad q mstaking validators -o json --limit=1000 \
| jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED")' \
| jq -r '.tokens + " - " + .description.moniker' \
| sort -gr | nl
```

View validator details
```
initiad q mstaking validator $(initiad keys show wallet --bech val -a) 
```

-----------------------------------------------------------------

## Managing Tokens
Withdraw reward from all validator
```
initiad tx distribution withdraw-all-rewards --from wallet --chain-id $CHAIN_ID --gas auto --gas-adjustment 1.4 fees 800ua0gi -y
```

Withdraw reward and commission
```
initiad tx distribution withdraw-rewards $(initiad keys show wallet --bech val -a) --commission --from wallet --chain-id $CHAIN_ID --gas auto --gas-adjustment 1.4 fees 800ua0gi -y
```

Delegate tokens to your validator
```
initiad tx mstaking delegate $(initiad keys show wallet --bech val -a) 1000000ua0gi --from wallet --chain-id $CHAIN_ID --gas auto --gas-adjustment 1.4 fees 800ua0gi -y
```

Delegate token to other validator, change `<to-valoper-address>` as you like
```
initiad tx mstaking delegate <to-valoper-address> 1000000ua0gi --from wallet --chain-id $CHAIN_ID --gas auto --gas-adjustment 1.4 fees 800ua0gi -y
```

Redelegate to another validator
```
initiad tx mstaking redelegate $(initiad keys show wallet --bech val -a) <to-valoper-address> 1000000ua0gi --from wallet --chain-id $CHAIN_ID --gas auto --gas-adjustment 1.4 fees 800ua0gi -y
```

Unbond token from your own validator
```
initiad tx mstaking unbond $(initiad keys show wallet --bech val -a) 1000000ua0gi --from wallet --chain-id $CHAIN_ID --gas auto --gas-adjustment 1.4 fees 800ua0gi -y
```

Send token to the wallet
```
initiad tx bank send wallet <to-wallet-address> 1000000ua0gi --from wallet --chain-id $CHAIN_ID --gas auto --gas-adjustment 1.4 fees 800ua0gi -y
```

-----------------------------------------------------------------

## Governance
Query list proposal
```bash
initiad query gov proposals
```

View proposal by ID
```bash
initiad query gov proposal 1
```

Vote option yes
```bash
initiad tx gov vote 1 yes --from wallet --chain-id $CHAIN_ID --gas auto --gas-adjustment 1.4 fees 800ua0gi -y
```

Vote option no
```bash
initiad tx gov vote 1 no --from wallet --chain-id $CHAIN_ID --gas auto --gas-adjustment 1.4 fees 800ua0gi -y
```

Vote option asbtain
```bash
initiad tx gov vote 1 abstain --from wallet --chain-id $CHAIN_ID --gas auto --gas-adjustment 1.4 fees 800ua0gi -y
```

Vote option NoWithVeto
```bash
initiad tx gov vote 1 NoWithVeto --from wallet --chain-id $CHAIN_ID --gas auto --gas-adjustment 1.4 fees 800ua0gi -y
```
-----------------------------------------------------------------

## Maintenance
Get validator information
```bash
initiad status 2>&1 | jq .validator_info
```

Get sync information
```bash
initiad status 2>&1 | jq .sync_info
```

Get node peer
```bash
echo $(initiad tendermint show-node-id)'@'$(curl -s ifconfig.me)':'$(cat $HOME/.initia/config/config.toml | sed -n '/Address to listen for incoming connection/{n;p;}' | sed 's/.*://; s/".*//')
```

Check validator keys
```bash
[[ $(initiad q mstaking validator $(initiad keys show wallet --bech val -a) -oj | jq -r .consensus_pubkey.key) = $(initiad status | jq -r .validator_info.PubKey.value) ]] && echo -e "\n\e[1m\e[32mTrue\e[0m\n" || echo -e "\n\e[1m\e[31mFalse\e[0m\n"
```

Get live peers
```bash
curl -sS http://localhost:23457/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'
```

Enable prometheus
```bash
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.initia/config/config.toml
```

Reset chain data
```bash
initiad tendermint unsafe-reset-all --keep-addr-book --home $HOME/.initia --keep-addr-book
```

> [!CAUTION]
> Before moving on to the next step, be aware that all chain data will be erased. **Ensure you've created a backup of your priv_validator_key.json!**

Remove node
```bash
cd $HOME
sudo systemctl stop initiad
sudo systemctl disable initiad
sudo rm /etc/systemd/system/initiad.service
sudo systemctl daemon-reload
sudo rm -f $(which initiad)
sudo rm -rf $HOME/.initia
sudo rm -rf $HOME/initia
sudo rm -rf $HOME/go
```

-----------------------------------------------------------------

<p align="center">
  &copy; 2024 BlockHub. All rights reserved.
</p>
