![0G Github Banner](https://github.com/BlockchainsHub/Testnet/assets/77204008/34a32724-b411-41e4-8696-e390dfa01cab)

# Panduan Node 0G
Panduan ini akan membantu anda dalam proses instalasi node 0G.

-----------------------------------------------------------------

## Spesifikasi Hardware Yang Dibutuhkan
| Spesifikasi | Hardware |
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
| jsonRPC | https://og-testnet-jsonrpc.blockhub.id |

-----------------------------------------------------------------

## Instalasi Node Manual
### 1. Instal Packages Yang Dibutuhkan
```bash
sudo apt update && \
sudo apt install curl git jq build-essential gcc unzip wget lz4 -y
```

### 2. Instal GO
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
git clone -b v0.2.3 https://github.com/0glabs/0g-chain.git
./0g-chain/networks/testnet/install.sh
source .profile
```

### 4. Mengatur Variable
Anda bisa melakukan beberapa perubahan yang dibutuhkan. Seperti `My_Node` yang ada pada variabel `MONIKER="My_Node"` bisa dirubah dengan nama node apa saja yang ingin anda gunakan dan `wallet` yang ada pada variabel `WALLET_NAME="wallet"` bisa dirubah dengan nama wallet apa saja yang ingin anda gunakan.
```bash
echo 'export MONIKER="My_Node"' >> ~/.bash_profile
echo 'export CHAIN_ID="zgtendermint_16600-2"' >> ~/.bash_profile
echo 'export WALLET_NAME="wallet"' >> ~/.bash_profile
source $HOME/.bash_profile
```

### 5. Inisialisasi Node
```bash
cd $HOME
0gchaind config chain-id $CHAIN_ID
0gchaind init $MONIKER --chain-id $CHAIN_ID
```

### 6. Download file genesis.json
```bash
curl -Ls https://github.com/0glabs/0g-chain/releases/download/v0.2.3/genesis.json > $HOME/.0gchain/config/genesis.json
```

Kemudian verifikasi file konfigurasi genesis sudah benar atau belum.
```bash
0gchaind validate-genesis
```

### 7. Update Konfigurasi
```bash
PEERS="96d615925aee68b90bfaf18d461e799fdcb22211@45.10.162.96:26656,7253c5556119b84f581bf3479db33687c2ff5cfe@38.242.143.169:26656,0aa16751b6c1884e755997d08dc17f8582aa9e38@45.10.163.80:26656,85233db31304a69fb2dda924b5de31c22dfcff5a@45.10.161.188:26656,89e272c0e5007e391f420e4f45e1473f91995025@154.26.155.239:26656,df8947d0bd46f24590e5d4bc3c06c59d543572d0@65.109.92.18:36656,d7ca6521ee30f8cf9eaf32e9edee1101e44c48e9@45.10.161.5:26656,364c45b7cab8a095cb59443f3e91fd102ec9eb95@158.220.118.216:26656,c8807bba12fa67676319df8e049ae5fac690cf55@45.159.228.20:26656,cfd099ade96d82908b4ab185eddbf90379579bfc@84.247.149.9:26656,03619b6f90fab32cd5f0cadbe3021e6a3cda16e3@154.26.156.101:26656,b3411cfb89113055dce89277c7cc7029ce451090@195.201.242.107:26656,bed108e9ce56d84a574fa02df90c734281ae19ef@162.55.65.137:27856,057f64f293f0843c849aa3f1f1e20a1a0add29f8@45.159.222.237:26656,369666051d45ed28379db34a80dfdf13e43d3681@5.104.80.63:26656,bc8898c416f7b22e56782eb16803150fd90863b6@81.0.221.180:26656,6970d09a9e004f6132b30db6eb5e27b6bd53a1d8@158.220.89.199:26656,7ecfe8d9404a4e1ea36cba5d546650da2b97bfd2@45.90.122.129:26656,4d98cf3cb2a61238a0b1557596cdc4b306472cb9@95.216.228.91:13456" && \
SEEDS="81987895a11f6689ada254c6b57932ab7ed909b6@54.241.167.190:26656,010fb4de28667725a4fef26cdc7f9452cc34b16d@54.176.175.48:26656,e9b4bc203197b62cc7e6a80a64742e752f4210d5@54.193.250.204:26656,68b9145889e7576b652ca68d985826abd46ad660@18.166.164.232:26656" && \
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.0gchain/config/config.toml

sed -i 's|^\s*#\?\s*laddr\s*=\s*"tcp://127.0.0.1:26657"|laddr = "tcp://0.0.0.0:26657"|' $HOME/.0gchain/config/config.toml

sed -i 's|^\s*#\?\s*api\s*=.*|api = "eth,txpool,personal,net,debug,web3"|' $HOME/.0gchain/config/app.toml

sed -i 's|^\s*#\?\s*address\s*=\s*"127.0.0.1:8545"|address = "0.0.0.0:8545"|' $HOME/.0gchain/config/app.toml

sed -i 's|^\s*#\?\s*ws-address\s*=\s*"127.0.0.1:8546"|ws-address = "0.0.0.0:8546"|' $HOME/.0gchain/config/app.toml
```

### 8. Membuat File Service
```bash
sudo tee /etc/systemd/system/0gchaind.service > /dev/null <<EOF
[Unit]
Description=0gchaind Validator Node
After=network.target

[Service]
User=root
ExecStart=/root/go/bin/0gchaind start
Environment="G0GC=900"
Environment="G0MELIMIT=40GB"
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

### 9. Memuat Ulang Konfigurasi Systemd
```bash
sudo systemctl daemon-reload
sudo systemctl enable 0gchaind 
sudo systemctl start 0gchaind
```

### 10. Cek Log
```bash
sudo journalctl -u 0gchaind -f -o cat
```

### 11. Buat Wallet Untuk Validator
```bash
0gchaind keys add $WALLET_NAME --eth
```
> [!CAUTION]
> **JANGAN LUPA UNTUK MENYIMPAN SEED PHRASE ANDA!**

Export private key dan masukkan ke Metamask.
```bash
0gchaind keys unsafe-export-eth-key $WALLET_NAME
```

Untuk menambahkan jaringan 0G di Metamask, buka website [scanner 0G](https://scan-testnet.0g.ai/) dan connect wallet. Jaringan 0G akan ditambahkan secara otomatis ke wallet Metamask anda.

### 12. Request Token Melalui Faucet
Silahkan klik tombol dibawah ini untuk request faucet.

<a href="https://faucet.0g.ai/" target="_blank">
  <img src="https://github.com/BlockchainsHub/Testnet/assets/77204008/12866a81-cac7-451a-96b5-4b202e8f1194" alt="Faucet Button" width="150" height="36.94" border="10" />
</a>

### 13. Buat Validator
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
> Jangan lupa menyimpan file `priv_validator_key.json` yang terletak di $HOME/.0gchain/config/

-----------------------------------------------------------------

# Daftar Command
## Pengelolaan Key / Akun
Generate key baru
```
0gchaind keys add wallet
```

Pemulihan key
```
0gchaind keys add wallet --recover
```

Lihat semua daftar key
```
0gchaind keys list
```

Hapus key
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

Cek saldo wallet
```
0gchaind q bank balances $(0gchaind keys show wallet -a)
```

-----------------------------------------------------------------

## Pengelolaan Validator
Buat validator
```
0gchaind tx staking create-validator \
--amount=1000000ua0gi \
--pubkey=$(0gchaind tendermint show-validator) \
--moniker=$MONIKER \
--identity="keybase-id" \
--details="info-detail" \
--website="link-website" \
--security-contact="alamat-email" \
--chain-id=$CHAIN_ID \
--commission-rate="0.10" \
--commission-max-rate="0.20" \
--commission-max-change-rate="0.01" \
--min-self-delegation="1" \
--from=wallet \
--gas=auto \
--gas-adjustment=1.4 \
-y
```

Edit validator
```
0gchaind tx staking edit-validator \
--new-moniker="nama-moniker" \
--identity="keybase-id" \
--details="detailed-info" \
--website="website-link" \
--security-contact="email-address" \
--chain-id=$CHAIN_ID \
--commission-rate="0.10" \
--from=wallet \
--gas=auto \
--gas-adjustment=1.4 \
-y
```

Unjail validator
```
0gchaind tx slashing unjail --from wallet --chain-id $CHAIN_ID --gas auto --gas-adjustment 1.4 fees 800ua0gi -y
```

Info jail validator
```
0gchaind q slashing signing-info $(0gchaind tendermint show-validator)
```

Daftar validator aktif
```
0gchaind q staking validators -o json --limit=1000 \
| jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' \
| jq -r '.tokens + " - " + .description.moniker' \
| sort -gr | nl
```

Daftar validator inaktif
```
0gchaind q staking validators -o json --limit=1000 \
| jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED")' \
| jq -r '.tokens + " - " + .description.moniker' \
| sort -gr | nl
```

Info detail validator
```
0gchaind q staking validator $(0gchaind keys show wallet --bech val -a) 
```

-----------------------------------------------------------------

## Pengelolaan Token
Penarikan reward dari semua validator
```
0gchaind tx distribution withdraw-all-rewards --from wallet --chain-id $CHAIN_ID --gas auto --gas-adjustment 1.4 fees 800ua0gi -y
```

Penarikan reward dan komisi
```
0gchaind tx distribution withdraw-rewards $(0gchaind keys show wallet --bech val -a) --commission --from wallet --chain-id $CHAIN_ID --gas auto --gas-adjustment 1.4 fees 800ua0gi -y
```

Delegasikan token ke validator anda
```
0gchaind tx staking delegate $(0gchaind keys show wallet --bech val -a) 1000000ua0gi --from wallet --chain-id $CHAIN_ID --gas auto --gas-adjustment 1.4 fees 800ua0gi -y
```

Delegasikan token ke validator lain, ubah `<to-valoper-address>` dengan alamat validator lain
```
0gchaind tx staking delegate <to-valoper-address> 1000000ua0gi --from wallet --chain-id $CHAIN_ID --gas auto --gas-adjustment 1.4 fees 800ua0gi -y
```

Pindahkan delegasi token ke validator lain
```
0gchaind tx staking redelegate $(0gchaind keys show wallet --bech val -a) <to-valoper-address> 1000000ua0gi --from wallet --chain-id $CHAIN_ID --gas auto --gas-adjustment 1.4 fees 800ua0gi -y
```

Unbond token dari validator anda
```
0gchaind tx staking unbond $(0gchaind keys show wallet --bech val -a) 1000000ua0gi --from wallet --chain-id $CHAIN_ID --gas auto --gas-adjustment 1.4 fees 800ua0gi -y
```

Kirim token antar wallet
```
0gchaind tx bank send wallet <to-wallet-address> 1000000ua0gi --from wallet --chain-id $CHAIN_ID --gas auto --gas-adjustment 1.4 fees 800ua0gi -y
```

-----------------------------------------------------------------

## Governance
Melihat daftar proposal
```bash
0gchaind query gov proposals
```

Melihat proposal berdasarkan ID
```bash
0gchaind query gov proposal 1
```

Vote dengan opsi ya
```bash
0gchaind tx gov vote 1 yes --from wallet --chain-id $CHAIN_ID --gas auto --gas-adjustment 1.4 fees 800ua0gi -y
```

Vote dengan opsi tidak
```bash
0gchaind tx gov vote 1 no --from wallet --chain-id $CHAIN_ID --gas auto --gas-adjustment 1.4 fees 800ua0gi -y
```

Vote dengan opsi abstain
```bash
0gchaind tx gov vote 1 abstain --from wallet --chain-id $CHAIN_ID --gas auto --gas-adjustment 1.4 fees 800ua0gi -y
```

Vote dengan opsi tidak dengan veto
```bash
0gchaind tx gov vote 1 NoWithVeto --from wallet --chain-id $CHAIN_ID --gas auto --gas-adjustment 1.4 fees 800ua0gi -y
```
-----------------------------------------------------------------

## Maintenance
Informasi validator
```bash
0gchaind status 2>&1 | jq .validator_info
```

Informasi sinkronisasi
```bash
0gchaind status 2>&1 | jq .sync_info
```

Dapatkan node peer
```bash
echo $(0gchaind tendermint show-node-id)'@'$(curl -s ifconfig.me)':'$(cat $HOME/.0gchain/config/config.toml | sed -n '/Address to listen for incoming connection/{n;p;}' | sed 's/.*://; s/".*//')
```

Cek validator keys
```bash
[[ $(0gchaind q staking validator $(0gchaind keys show wallet --bech val -a) -oj | jq -r .consensus_pubkey.key) = $(0gchaind status | jq -r .validator_info.PubKey.value) ]] && echo -e "\n\e[1m\e[32mTrue\e[0m\n" || echo -e "\n\e[1m\e[31mFalse\e[0m\n"
```

Dapatkan live peers
```bash
curl -sS http://localhost:26657/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'
```

Aktifkan prometheus
```bash
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.0gchain/config/config.toml
```

Reset data chain
```bash
0gchaind tendermint unsafe-reset-all --keep-addr-book --home $HOME/.0gchain --keep-addr-book
```

> [!CAUTION]
> Sebelum melanjutkan ke langkah berikutnya, ketahuilah bahwa semua data chain akan dihapus. **Pastikan Anda telah membackup priv_validator_key.json Anda!**

Hapus node
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
