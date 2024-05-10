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

-----------------------------------------------------------------

## Instalasi Node Otomatis
Jalankan command berikut ini untuk menginstal node secara otomatis.
```
curl -o zg_ID.sh https://gist.githubusercontent.com/botxx15/972b30bfa135b92c20438f48fc8a6e7d/raw/d8da1be11b125f8e70f371f14536d4b452376efb/zg_ID.sh && bash zg_ID.sh
``` 

> [!NOTE]
> Setelah node tersinkronisasi, anda dapat melanjutkan ke tahap [11. Buat Wallet Untuk Validator](#11-Buat-Wallet-Untuk-Validator)

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
git clone -b v0.1.0 https://github.com/0glabs/0g-chain.git
./0g-chain/networks/testnet/install.sh
source .profile
```
### 4. Mengatur Variable
Anda bisa melakukan beberapa perubahan yang dibutuhkan. Seperti `My_Node` yang ada pada variabel `MONIKER="My_Node"` bisa dirubah dengan nama node apa saja yang ingin anda gunakan dan `wallet` yang ada pada variabel `WALLET_NAME="wallet"` bisa dirubah dengan nama wallet apa saja yang ingin anda gunakan.
```bash
echo 'export MONIKER="My_Node"' >> ~/.bash_profile
echo 'export CHAIN_ID="zgtendermint_16600-1"' >> ~/.bash_profile
echo 'export WALLET_NAME="wallet"' >> ~/.bash_profile
source $HOME/.bash_profile
```

### 5. Inisialisasi Node
```bash
cd $HOME
0gchaind config chain-id $CHAIN_ID
0gchaind init $MONIKER --chain-id $CHAIN_ID
```

### 6. Download file genesis.json & addrbook.json
```bash
curl -Ls https://snapshots.liveraven.net/snapshots/testnet/zero-gravity/genesis.json > $HOME/.0gchain/config/genesis.json

curl -Ls https://snapshots.liveraven.net/snapshots/testnet/zero-gravity/addrbook.json > $HOME/.0gchain/config/addrbook.json
```

Kemudian verifikasi file konfigurasi genesis sudah benar atau belum.
```bash
0gchaind validate-genesis
```

### 7. Tambahkan seeds dan peers pada file config.toml
```bash
PEERS="9d7564df34efa146a94c073e5bf3f5e11f947b75@155.133.22.230:26656,a4055b828e59832c7a06d61fc51347755a160d0b@157.90.33.62:21656" && \
SEEDS="c4d619f6088cb0b24b4ab43a0510bf9251ab5d7f@54.241.167.190:26656,44d11d4ba92a01b520923f51632d2450984d5886@54.176.175.48:26656,f2693dd86766b5bf8fd6ab87e2e970d564d20aff@54.193.250.204:26656,f878d40c538c8c23653a5b70f615f8dccec6fb9f@54.215.187.94:26656" && \
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.0gchain/config/config.toml
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
Restart=on-failure
RestartSec=3
LimitNOFILE=4096

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
  --gas-adjustment=1.4
  --fees=800ua0gi
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
--amount 1000000ua0gi \
--pubkey $(0gchaind tendermint show-validator) \
--moniker $MONIKER \
--identity "keybase-id" \
--details "info-detail" \
--website "link-website" \
--security-contact "alamat-email" \
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
--details "info-detail" \
--website "link-website" \
--security-contact "alamat-email" \
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
0gchaind status 2>&1 | jq .ValidatorInfo
```

Informasi sinkronisasi
```bash
0gchaind status 2>&1 | jq .SyncInfo
```

Dapatkan node peer
```bash
echo $(0gchaind tendermint show-node-id)'@'$(curl -s ifconfig.me)':'$(cat $HOME/.0gchain/config/config.toml | sed -n '/Address to listen for incoming connection/{n;p;}' | sed 's/.*://; s/".*//')
```

Cek validator keys
```bash
[[ $(0gchaind q staking validator $(0gchaind keys show wallet --bech val -a) -oj | jq -r .consensus_pubkey.key) = $(0gchaind status | jq -r .ValidatorInfo.PubKey.value) ]] && echo -e "\n\e[1m\e[32mTrue\e[0m\n" || echo -e "\n\e[1m\e[31mFalse\e[0m\n"
```

Dapatkan live peers
```bash
curl -sS http://localhost:23457/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'
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
