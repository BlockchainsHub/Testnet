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

## Connections
Peer

`007fd507f039e06bc8d4cb3169f005745dd01f84@158.220.89.92:26656`

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
git checkout v0.2.14
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
PEERS="a63a6f6eae66b5dce57f5c568cdb0a79923a4e18@peer-initia-testnet.trusted-point.com:26628,439fe02b731d7f99a62e44ac5b2e1b02353ca631@38.242.251.179:39656,4db605b6f399a173cfc30e843a7d6a10cd3222a3@158.220.86.6:17956,fd14410d3d6ba362a20d47c02e077da86017cadf@65.21.244.157:17956,a8638b4701f2d11e9269dfd4c2ed0509bd7b12d9@194.163.191.117:39656,0ce5a28686d961d0f1315069c03adb74c6fccc80@37.60.244.91:24556,de31968f3b35942b5a1123998ff0c4ebd3c3aae5@88.99.193.146:26656,f396faca04598721481e714dcb0e3c8ed05a406c@49.12.209.114:15656,fd06e3e5f03b31757ee2ce78d0bf85bb1c71a2d9@65.109.166.136:26656,0d5b90a3b620a7e602f099eb4da99fc03995874e@165.22.245.86:17956,028999a1696b45863ff84df12ebf2aebc5d40c2d@37.27.48.77:26656,7033bed7fa79360e24d5d0cf2f5fee8a683766a9@154.26.129.223:17956,1376a7400ee5400e226ebab384ad89de408163dc@62.171.179.87:13656,23251217584bc066c8027cc735ca1b2893896178@185.197.251.195:17956,277ae7258c9ac789262ef125cfdbf1c02958510a@37.27.71.199:22656,32fece76b6d278672fb73059764f5d6f77086f3a@148.251.3.125:19656,c612c1c6ad4a59fb62a31428782921591e8bb684@42.117.19.109:10656,fa69efe26762f987a1e1eaf4ea053b35380838dc@80.65.211.232:17956,0d6437ca9242b5878f6c784b88e918ba12f12c08@89.58.63.240:53456,f24e92c2b15ea8f212ec63ebae5451d8fcc7da8b@81.0.248.152:39656,ba053d26fe5c30842ddcc2c34c9893d78204ced0@157.90.154.36:17956,32f59b799e6e840fb47b363ba59e45c3519b3a5f@136.243.104.103:24556,5c2a752c9b1952dbed075c56c600c3a79b58c395@195.3.221.9:26686,862d16bec51e4e2751b00605416df94b7440b7f3@49.13.147.156:39656,1813a8de79d48674f184553800122f7bf794cd57@213.199.52.16:26656,a633694e4f10060023b3c8319ae130fa927f706b@207.180.251.85:17956,22c876f711032026c54d2ccfe81cb2cfe1ec9ac1@37.60.243.170:26656,15a9693fbcdd9d8aea48030be3b520b1d69e8d66@193.34.213.228:14656,98f0f8e9209aa0a8abad39b94b0d2663a3be24ec@95.216.70.202:26656,c2a36ef8b4aaef3acc7d7cbfd77d10cf4cedaa3d@77.237.235.205:53456,8999ddce339185140913a64c623d0cb2a0e104f5@185.202.223.117:17956,04538a79c786a781345533aecff034379023e661@65.108.126.173:53456,670d532665a0f93ccbba6d132109c207301d6353@194.163.170.113:17956,4d98be9bf94c8ec06f7bbd96a9b4de507d2035b8@37.60.252.43:39656,7d097908682ef4f4e168f2136da2612ec43da27c@85.215.181.21:26656,7f194243f4d9ffbe15412fc5a11eec5c914c9300@167.86.114.207:17956,a3f2bd6fcf79eec06a5f384b3edaf1fe6e4ac9ce@82.208.22.54:17956,6dbb770a4b19f685c1cfe3a16738022eb9ca12e2@101.44.82.135:53456,ef4a25ea7000773cb6094dd5d905686ab7426541@158.220.122.90:14656,2bc4ca9a821b56e5786378a4167c57ef6e0d174f@167.235.200.43:17956,e3ee807b6f4e5a5f76e3e3b73da23a07488f01fb@5.75.170.27:17956,9228bbd89be619dd943e44633585c1657051a7d0@173.212.193.103:17956,cbba1ec1e228e01b31d22864c36fb7039088a5aa@194.163.152.41:53456,ae241bcfd5fffef3173c5bd4c72b0b384db5db88@49.13.213.52:26656" && \
sed -i \
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
  --gas-prices 0.15uinit \
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
--gas-prices 0.15uinit \
-y
```

Edit validator
```
initiad tx mstaking edit-validator \
--new-moniker "your-new-moniker" \
--identity "keybase-id" \
--details "detailed-info" \
--website "website-link" \
--security-contact "email-address" \
--chain-id $CHAIN_ID \
--commission-rate 0.10 \
--from wallet \
--gas auto \
--gas-adjustment 1.4 \
--gas-prices 0.15uinit \
-y
```

Unjail validator
```
initiad tx slashing unjail --from wallet --chain-id $CHAIN_ID --gas-adjustment 1.4 --gas auto --gas-prices 0.15uinit -y
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
initiad tx distribution withdraw-all-rewards --from wallet --chain-id $CHAIN_ID --gas-adjustment 1.4 --gas auto --gas-prices 0.15uinit -y
```

Withdraw reward and commission
```
initiad tx distribution withdraw-rewards $(initiad keys show wallet --bech val -a) --commission --from wallet --chain-id $CHAIN_ID --gas-adjustment 1.4 --gas auto --gas-prices 0.15uinit -y
```

Delegate tokens to your validator
```
initiad tx mstaking delegate $(initiad keys show wallet --bech val -a) 1000000ua0gi --from wallet --chain-id $CHAIN_ID --gas-adjustment 1.4 --gas auto --gas-prices 0.15uinit -y
```

Delegate token to other validator, change `<to-valoper-address>` as you like
```
initiad tx mstaking delegate <to-valoper-address> 1000000ua0gi --from wallet --chain-id $CHAIN_ID --gas-adjustment 1.4 --gas auto --gas-prices 0.15uinit -y
```

Redelegate to another validator
```
initiad tx mstaking redelegate $(initiad keys show wallet --bech val -a) <to-valoper-address> 1000000ua0gi --from wallet --chain-id $CHAIN_ID --gas-adjustment 1.4 --gas auto --gas-prices 0.15uinit -y
```

Unbond token from your own validator
```
initiad tx mstaking unbond $(initiad keys show wallet --bech val -a) 1000000ua0gi --from wallet --chain-id $CHAIN_ID --gas-adjustment 1.4 --gas auto --gas-prices 0.15uinit -y
```

Send token to the wallet
```
initiad tx bank send wallet <to-wallet-address> 1000000ua0gi --from wallet --chain-id $CHAIN_ID --gas-adjustment 1.4 --gas auto --gas-prices 0.15uinit -y
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
initiad tx gov vote 1 yes --from wallet --chain-id $CHAIN_ID --gas-adjustment 1.4 --gas auto --gas-prices 0.15uinit -y
```

Vote option no
```bash
initiad tx gov vote 1 no --from wallet --chain-id $CHAIN_ID --gas-adjustment 1.4 --gas auto --gas-prices 0.15uinit -y
```

Vote option asbtain
```bash
initiad tx gov vote 1 abstain --from wallet --chain-id $CHAIN_ID --gas-adjustment 1.4 --gas auto --gas-prices 0.15uinit -y
```

Vote option NoWithVeto
```bash
initiad tx gov vote 1 NoWithVeto --from wallet --chain-id $CHAIN_ID --gas-adjustment 1.4 --gas auto --gas-prices 0.15uinit -y
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
curl -sS http://localhost:26657/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'
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
