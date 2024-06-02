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

## Service
### Public Endpoint
| Public | Endpoint |
|-|-
| RPC | https://initia-testnet-rpc.blockhub.id |
| API | https://initia-testnet-api.blockhub.id |
| gRPC | https://initia-testnet-grpc.blockhub.id |

### Connections
Peer
```bash
5b39ab3cc4c4096aa0411fbad01fd1e3aba50d9b@2607:5300:203:673e:::26656
```

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
git checkout v0.2.15
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
PEERS="5b39ab3cc4c4096aa0411fbad01fd1e3aba50d9b@2607:5300:203:673e:::26656,067da4f5921dbecb61a126f98f90ecb99daecd06@144.76.32.69:23596,e1d4774eb1913a600801e27c83b07eb47bc843ad@84.247.190.83:26656,4b46df8b64fcacc2ad5253bc0831369d6bb8d4d5@185.234.65.45:26656,6307db840eb8e97536465d3d9f6ba581dfc323f6@95.217.62.210:26433,c78059c16af7d899b48d6db7fb131186d693ecca@149.50.107.225:14656,0ac5727d219ee09a12cf3e2b61a25ae51bcbca70@49.12.174.227:26656,3b944bcae9db0b88d8419adde8e26188a6a5ef5d@65.109.59.22:25756,5042b5b842d9fb4c48d66722caa07a39fd7f37da@149.50.99.67:26656,5cd075abbda4cc6d0c45bc7cc1cddb43ed0a5dca@23.88.65.98:26656,ff6e66a6df9fdc3e6966e594352775a4c52d8277@109.123.249.213:26656,140f463230963c69d3ca7bd80a38ee18c6aecf8e@220.79.63.226:26656,a98e47c02763d05f2a9623cb67e8e40e0d06504a@5.9.70.180:15674,f59bfd2739a39b61cfdb62f379e2f37dc6243b0e@37.27.69.160:29356,007ed503cdcb85ad7b4d68dc8b222ad3015a88e9@193.233.164.243:26656,f398117804c638ace3c189781d08db77caa20aa9@65.108.109.48:10656,52f3786f0df62d1fb411ab6ccbb5eda448d8ebee@5.9.1.90:53456,89ba0093c88e581754874ca66915ae86c4f50ba3@78.46.76.145:35656,b2dfeb8dd56b6c5aa1f4a5c4323d70b3cd9347a0@164.92.229.20:26656,bd943906483fafa2b783879f810ff8ed0b92d34a@88.198.17.120:33756,77726417daeff2b53b6bec9fdb17e8bd9367ec4d@162.55.4.42:27739,c99f1967ebcd5edb8eb9e624712e7372384a5917@57.129.28.107:46656,34d468714a3193d73bef5225d9c507cbf682b9c4@148.113.6.134:26656,1c5910b1a601a6efb3d7aa83545db0eb2b921097@89.117.144.37:33756,ca1724a769b0d8df5eefe6d5c3be398f3b0e50d5@65.108.58.227:26656,5d49858aff3d23d9f4ba6c5bf7a799dd78259f2d@95.217.37.139:17956,c86acc6936b8cb85d75c68880092f99da8d7f380@5.9.85.89:22656,1a0519078a0cd46e444847df9a9be396bbb70ac9@46.4.50.247:36256,908aa7043abc1599bb869b7209f260228c47ba37@162.55.80.21:26656,bd947b86c59588b9e1a98d93df4f34a3ab45ab7c@84.247.168.95:26656,7b3c2bd99e7f6bcca8c6c073f919a267b52bb867@54.189.184.82:26656,1e07b92a4c3bc1d66c2257a3b2378f80a7bdc1a2@135.181.239.149:26656,7b3a7e99b34aa9bfa6f8cad95337c7582a7b3150@84.247.129.208:26656,fc3329f887d5e11465976ae966cc7145eee8eff6@176.9.48.151:20756,9ec13340ca1bdb05b530214da8dad9690d4e910d@23.88.66.147:11756,58535a18c64744100f5feb5b54c4c8e5fa328ece@38.242.136.164:26656,64928cf0b8414a42e7f6cfd7f11cfaa7eb32d37d@78.46.76.134:25756,62775997caa3d814c5ad91492cb9d411aea91c58@51.38.53.103:26856,e8545ccc3d76d253204da9ffdeb9f8e94b284443@49.12.174.232:26656,0c51b92a123bd4310dfe964a969106458e45b713@63.250.54.71:26656,e44e11c6f229a571f4239781f249a25e4257c179@185.84.224.160:25756,b2d980cf7b48fde5edda8331b45da1ab6ad32d17@38.58.183.44:51656,b41b9b0b398059bd483123043fafd1e311f05cbe@65.21.197.228:25756,f7b49eeb291020ccfde60626a7d64653162da6da@104.237.2.204:15656,5998a540cbbcc0b01a82fc66b9ab9f5802521af5@5.9.61.237:39392,0b3fb058e598a78db2ee24c0492cc590f788c209@162.19.235.92:26656,344a7551d237efddf3c65bedfff035e315c557c2@86.48.7.24:26656,09b675e1b10830919143b760c578db83bd8d8965@176.9.16.233:26656,2f74b8917a6a376e505af232da24b74448f9f019@86.48.6.224:26656,6cf83954175cd5569f8df38a5e5bd25bcb4b0cb4@86.48.7.48:26656,4d7f588103a3a51e47bd4c7812a8c1dae6bff6b9@84.247.170.45:13056,fb1d48a7e4c2b75fae1d27a6ba2ef4b0352c34a1@86.48.7.42:26656,84cc9ed55941bb69d2d98da1a69a6ba599926423@46.250.229.196:26656,d37e6633cfc00b0db5087ed87c448edbcccdea95@37.60.234.74:11656,aad8ad2f802345745d328ff7a09be541c42514f5@86.48.7.30:26656,3897daecbbc96ef27c1a5c24db61cb4553d2aad7@86.48.7.41:26656,854f75cf7b777fb1426b15a6dec69d8fb72da3a7@86.48.7.34:26656,f1a37f5780ed955d9ea83f6c01de2ea83ea51df2@75.119.141.148:26656,00991ddb78fb4755ef11fb179595e1cc27a1886b@86.48.7.43:26656,a4ab0afc250a3c133f6e94df98a97689044bc31a@37.27.111.113:26656,a1fd0afa28bfb5df04d0eb3ccb81d3722ec419d7@86.48.7.49:26656,073b8e7b34b2d185cd0526e1dd3f395f1e67c1ff@84.46.247.217:26656,66abd758f6971eb8227fc54d11cb56ca1ca280e6@65.109.113.251:13656,454deb6b36e2b0e465827e60dea3d954f3efc787@43.133.30.52:26656,bb6b5982fcf2715461b8c3702bc1259b66f340cb@65.108.96.173:26656,136bc4cc4c88efbcd05140b4d0857d80c95b5b40@89.117.54.151:26656,2c729d33d22d8cdae6658bed97b3097241ca586c@195.14.6.129:26019,15dadcdb79ad05ef7b3cf8d0cc9cf270c005f191@57.128.74.22:56116,5af4465b2a9cc7be2589ccbca20098a207698780@37.60.224.144:26656,1d34feab3765f17801ac6053d101b1671e6ad8b1@31.220.72.104:26656,3f485e77240d0c47252fadbc95470315d10cd7e1@135.181.246.31:25756,4770ec44af4b91f66f06ee5221bc4e8e82a93047@212.90.120.194:26656,3f6682ce84e5316441f4c8c614476d445256261a@194.113.67.157:26656,d912bf78113975fb97acda5c947b0812ccabc8c5@103.50.33.195:26656,277ae7258c9ac789262ef125cfdbf1c02958510a@37.27.71.199:22656,e4285d88ae124a6ba1446bdc5340ec8446aa46e8@193.34.213.76:26656,cd69bcb00a6ecc1ba2b4a3465de4d4dd3e0a3db1@65.108.231.124:51656,548e26b95b895efc964b08a6b2e991c6d5a6791d@142.132.151.35:15674,17b0fb616bae3fc2e6babf717e2ec132353142db@51.195.88.136:15674,a98484ac9cb8235bd6a65cdf7648107e3d14dab4@95.217.74.22:17956,72a02983f3ab49ca64d55fc1ba18eea2ff8b53c8@152.42.242.129:33756,98ebab28d0f78d43e2260e2eb040ac035f6a6798@194.163.170.251:16656,2cb4ac6d783ad0e4691c09636c51647f33c2b968@167.86.85.135:14656,e6e71f252fea56684d825b1c47df37d6f5437354@5.9.104.206:26656,d5c26625396e411a8388823cda8bf0f38675c1df@84.247.130.65:26656,a1107be642519cf682f6826e16285a469d0b167c@184.174.39.156:26656,31de0f9d8d100bd8f94987f54efcaa28415e2806@162.0.226.227:26656,1032b1b8bbac49fa32666ac20a2de002b5b0d038@89.58.13.159:6969" && \
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
--amount 1000000uinit \
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
initiad tx mstaking delegate $(initiad keys show wallet --bech val -a) 1000000uinit --from wallet --chain-id $CHAIN_ID --gas-adjustment 1.4 --gas auto --gas-prices 0.15uinit -y
```

Delegate token to other validator, change `<to-valoper-address>` as you like
```
initiad tx mstaking delegate <to-valoper-address> 1000000uinit --from wallet --chain-id $CHAIN_ID --gas-adjustment 1.4 --gas auto --gas-prices 0.15uinit -y
```

Redelegate to another validator
```
initiad tx mstaking redelegate $(initiad keys show wallet --bech val -a) <to-valoper-address> 1000000uinit --from wallet --chain-id $CHAIN_ID --gas-adjustment 1.4 --gas auto --gas-prices 0.15uinit -y
```

Unbond token from your own validator
```
initiad tx mstaking unbond $(initiad keys show wallet --bech val -a) 1000000uinit --from wallet --chain-id $CHAIN_ID --gas-adjustment 1.4 --gas auto --gas-prices 0.15uinit -y
```

Send token to the wallet
```
initiad tx bank send wallet <to-wallet-address> 1000000uinit --from wallet --chain-id $CHAIN_ID --gas-adjustment 1.4 --gas auto --gas-prices 0.15uinit -y
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
