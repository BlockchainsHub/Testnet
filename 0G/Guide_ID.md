![0G Github Banner](https://github.com/BlockchainsHub/Testnet/assets/77204008/34a32724-b411-41e4-8696-e390dfa01cab)

# Panduan Node 0G
Panduan ini akan membantu anda dalam proses instalasi node 0G.

-----------------------------------------------------------------

## Spesifikasi Hardware Yang Dibutuhkan
| Spesifikasi | Hardware |
|-|-
| CPU | 4 Cores |
| Memory | 8 GB |
| Storage | 500 GB NVMe SSD |
| Bandwidth | 100mbps |
| OS | Linux |

-----------------------------------------------------------------

## Instalasi Node Otomatis
Jalankan command berikut ini untuk menginstal node secara otomatis.
```
curl -o 0g.sh https://gist.githubusercontent.com/botxx15/b122ad56138a9805f54a7dc12c25059f/raw/442771b03ea81bf16dd5677db091051795cd602f/0g.sh && bash 0g.sh
```

### Cek Status Sinkronisasi
Jika status `"catching_up": false` berarti node sudah tersinkronisasi, sebaliknya jika status `"catching_up": true` berarti node belum selesai melakukan sinkronisasi.
```bash
evmosd status | jq .SyncInfo
``` 

> [!NOTE]
> Setelah node tersinkronisasi, anda dapat melanjutkan ke tahap [14. Buat Wallet Untuk Validator](#14-Buat-Wallet-Untuk-Validator)

## Instalasi Node Manual
### 1. Instal Packages Yang Dibutuhkan
```bash
sudo apt update && \
sudo apt install curl git jq build-essential gcc unzip wget lz4 -y
```

### 2. Instal GO
```bash
cd $HOME && \
ver="1.21.3" && \
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
git clone https://github.com/0glabs/0g-evmos.git
cd 0g-evmos
git checkout v1.0.0-testnet
make install
evmosd version
```

### 4. Mengatur Variable
Anda bisa melakukan beberapa perubahan yang dibutuhkan. Seperti `My_Node` yang ada pada variabel `MONIKER="My_Node"` bisa dirubah dengan nama node apa saja yang ingin anda gunakan dan `wallet` yang ada pada variabel `WALLET_NAME="wallet"` bisa dirubah dengan nama wallet apa saja yang ingin anda gunakan.
```bash
echo 'export MONIKER="My_Node"' >> ~/.bash_profile
echo 'export CHAIN_ID="zgtendermint_9000-1"' >> ~/.bash_profile
echo 'export WALLET_NAME="wallet"' >> ~/.bash_profile
echo 'export RPC_PORT="26657"' >> ~/.bash_profile
source $HOME/.bash_profile
```

### 5. Inisialisasi Node
```bash
cd $HOME
evmosd init $MONIKER --chain-id $CHAIN_ID
evmosd config chain-id $CHAIN_ID
evmosd config node tcp://localhost:$RPC_PORT
evmosd config keyring-backend test
```

### 6. Download file genesis.json
```bash
wget https://github.com/0glabs/0g-evmos/releases/download/v1.0.0-testnet/genesis.json -O $HOME/.evmosd/config/genesis.json
```

### 7. Tambahkan seeds dan peers pada file config.toml
```bash
PEERS="1248487ea585730cdf5d3c32e0c2a43ad0cda973@peer-zero-gravity-testnet.trusted-point.com:26326" && \
SEEDS="8c01665f88896bca44e8902a30e4278bed08033f@54.241.167.190:26656,b288e8b37f4b0dbd9a03e8ce926cd9c801aacf27@54.176.175.48:26656,8e20e8e88d504e67c7a3a58c2ea31d965aa2a890@54.193.250.204:26656,e50ac888b35175bfd4f999697bdeb5b7b52bfc06@54.215.187.94:26656" && \
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.evmosd/config/config.toml
```

### 8. Konfigurasi Prunning (Opsional)
Untuk menghemat penyimpanan anda bisa menggunakan konfigurasi prunning dibawah ini.
```bash
sed -i.bak -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.evmosd/config/app.toml
sed -i.bak -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.evmosd/config/app.toml
sed -i.bak -e "s/^pruning-interval *=.*/pruning-interval = \"10\"/" $HOME/.evmosd/config/app.toml
```

### 9. Mengatur Min Gas Price 
```bash
sed -i "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.00252aevmos\"/" $HOME/.evmosd/config/app.toml
```

### 10. Mengaktifkan Indexer (Opsional)
```bash
sed -i "s/^indexer *=.*/indexer = \"kv\"/" $HOME/.evmosd/config/config.toml
```

### 11. Membuat File Service
```bash
sudo tee /etc/systemd/system/ogd.service > /dev/null <<EOF
[Unit]
Description=OG Node
After=network.target

[Service]
User=$USER
Type=simple
ExecStart=$(which evmosd) start --home $HOME/.evmosd
Restart=on-failure
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

### 12. Memuat Ulang Konfigurasi Systemd
```bash
sudo systemctl daemon-reload && \
sudo systemctl enable ogd
```

> [!TIP]
> Sebelum menjalankan node anda bisa menggunakan [State Sync](#state-sync) atau [Snapshot](#download-snapshot) untuk mempercepat sinkronisasi.

### 13. Menjalankan Node
```bash
sudo systemctl restart ogd && \
sudo journalctl -u ogd -f -o cat
```

### 14. Buat Wallet Untuk Validator
Pada saat pembuatan wallet baru anda akan diberikan **seed phrase**. **Jangan lupa untuk menyimpan seed phrase anda!**. Anda juga akan
langsung diberikan alamat wallet dalam format EVM (0x), pastikan untuk menyimpan alamat tersebut karena akan digunakan pada saat anda 
melakukan request faucet.
```bash
evmosd keys add $WALLET_NAME
echo "0x$(evmosd debug addr $(evmosd keys show $WALLET_NAME -a) | grep hex | awk '{print $3}')"
```
> [!CAUTION]
> **JANGAN LUPA UNTUK MENYIMPAN SEED PHRASE ANDA!**

Contoh output:
![CleanShot 2024-04-09 at 20 13 29@2x](https://github.com/BlockchainsHub/Testnet/assets/77204008/0d8cf252-a925-45b8-99f2-e772997c420a)

### 16. Request Token Melalui Faucet
Silahkan klik tombol dibawah ini untuk request faucet.

<a href="https://faucet.0g.ai/" target="_blank">
  <img src="https://github.com/BlockchainsHub/Testnet/assets/77204008/12866a81-cac7-451a-96b5-4b202e8f1194" alt="Faucet Button" width="150" height="36.94" border="10" />
</a>

### 17. Cek Saldo Wallet
```bash
evmosd q bank balances $(evmosd keys show $WALLET_NAME -a) 
```
Contoh output:
![CleanShot 2024-04-09 at 21 18 52@2x](https://github.com/BlockchainsHub/Testnet/assets/77204008/870464cb-a6f5-4d42-8712-9fc29cc92436)

> [!NOTE]
> Note: Anda akan mendapatkan *100000000000000000aevmos* dari faucet. Untuk membuat validator bergabung dengan set aktif, anda memerlukan setidaknya *1000000000000000000aevmos* (**10 kali lebih banyak**).

### 18. Buat Validator
```bash
evmosd tx staking create-validator \
  --amount=10000000000000000aevmos \
  --pubkey=$(evmosd tendermint show-validator) \
  --moniker=$MONIKER \
  --chain-id=$CHAIN_ID \
  --commission-rate=0.05 \
  --commission-max-rate=0.10 \
  --commission-max-change-rate=0.01 \
  --min-self-delegation=1 \
  --from=$WALLET_NAME \
  --identity="" \
  --website="" \
  --details="0G to the moon!" \
  --gas=500000 \
  --gas-prices=99999aevmos \
  -y
```
> [!CAUTION]
> Jangan lupa menyimpan file `priv_validator_key.json` yang terletak di $HOME/.evmosd/config/

-----------------------------------------------------------------

## State Sync
### 1. Backup File priv_validator_state.json 
```bash
cp $HOME/.evmosd/data/priv_validator_state.json $HOME/.evmosd/priv_validator_state.json.backup
```

### 2. Reset Database
```bash
evmosd tendermint unsafe-reset-all --home $HOME/.evmosd --keep-addr-book
```

### 3. Mengatur Variabel Yang Dibutuhkan
```bash
PEERS="1248487ea585730cdf5d3c32e0c2a43ad0cda973@peer-zero-gravity-testnet.trusted-point.com:26326" && \
RPC="https://rpc-zero-gravity-testnet.trusted-point.com:443" && \
LATEST_HEIGHT=$(curl -s --max-time 3 --retry 2 --retry-connrefused $RPC/block | jq -r .result.block.header.height) && \
TRUST_HEIGHT=$((LATEST_HEIGHT - 1500)) && \
TRUST_HASH=$(curl -s --max-time 3 --retry 2 --retry-connrefused "$RPC/block?height=$TRUST_HEIGHT" | jq -r .result.block_id.hash) && \

if [ -n "$PEERS" ] && [ -n "$RPC" ] && [ -n "$LATEST_HEIGHT" ] && [ -n "$TRUST_HEIGHT" ] && [ -n "$TRUST_HASH" ]; then
    sed -i.bak \
        -e "/\[statesync\]/,/^\[/{s/\(enable = \).*$/\1true/}" \
        -e "/^rpc_servers =/ s|=.*|= \"$RPC,$RPC\"|;" \
        -e "/^trust_height =/ s/=.*/= $TRUST_HEIGHT/;" \
        -e "/^trust_hash =/ s/=.*/= \"$TRUST_HASH\"/" \
        -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" \
        $HOME/.evmosd/config/config.toml
    echo -e "\nLATEST_HEIGHT: $LATEST_HEIGHT\nTRUST_HEIGHT: $TRUST_HEIGHT\nTRUST_HASH: $TRUST_HASH\nPEERS: $PEERS\n\nALL IS FINE"
else
    echo -e "\nError: One or more variables are empty. Please try again or change RPC\nExiting...\n"
fi
```

### 4. Memindahkan File priv_validator_state.json back
```bash
mv $HOME/.evmosd/priv_validator_state.json.backup $HOME/.evmosd/data/priv_validator_state.json
```

### 5. Menjalankan Node
```bash
sudo systemctl restart ogd && sudo journalctl -u ogd -f -o cat
```

Setelah menjalankan node anda akan melihat log berikut. Diperlukan waktu hingga 5 menit agar snapshot bisa ditemukan. Jika tidak berhasil, coba download [Snapshot](#download-snapshot).
```py
2:39PM INF sync any module=statesync msg="Discovering snapshots for 15s" server=node
2:39PM INF Discovered new snapshot format=3 hash="?^��I��\r�=�O�E�?�CQD�6�\x18�F:��\x006�" height=602000 module=statesync server=node
2:39PM INF Discovered new snapshot format=3 hash="%���\x16\x03�T0�v�f�C��5�<TlLb�5��l!�M" height=600000 module=statesync server=node
2:42PM INF VerifyHeader hash=CFC07DAB03CEB02F53273F5BDB6A7C16E6E02535B8A88614800ABA9C705D4AF7 height=602001 module=light server=node
```

Setelah beberapa waktu kalian akan melihat log berikut. Diperlukan waktu 5 menit bagi node untuk mengejar sisa blok.
```py
2:43PM INF indexed block events height=602265 module=txindex server=node
2:43PM INF executed block height=602266 module=state num_invalid_txs=0 num_valid_txs=0 server=node
2:43PM INF commit synced commit=436F6D6D697449447B5B31313720323535203139203132392031353920313035203136352033352031353320313220353620313533203139352031372036342034372033352034372032333220373120313939203720313734203620313635203338203336203633203235203136332039203134395D3A39333039417D module=server
2:43PM INF committed state app_hash=75FF13819F69A523990C3899C311402F232FE847C707AE06A526243F19A30995 height=602266 module=state num_txs=0 server=node
2:43PM INF indexed block events height=602266 module=txindex server=node
2:43PM INF executed block height=602267 module=state num_invalid_txs=0 num_valid_txs=0 server=node
2:43PM INF commit synced commit=436F6D6D697449447B5B323437203134322032342031313620323038203631203138362032333920323238203138312032333920313039203336203420383720323238203236203738203637203133302032323220313431203438203337203235203133302037302032343020313631203233372031312036365D3A39333039427D module=server
```

### 6. Cek Status Sinkronisasi
Jika status `"catching_up": false` berarti node sudah tersinkronisasi, sebaliknya jika status `"catching_up": true` berarti node belum selesai melakukan sinkronisasi.
```bash
evmosd status | jq .SyncInfo
```

### 7. Menonaktifkan State Sync
Jika node sudah selesai tersinkronisasi anda bisa menonaktifkan State Sync.
```bash
sed -i.bak -e "/\[statesync\]/,/^\[/{s/\(enable = \).*$/\1false/}" $HOME/.evmosd/config/app.toml
```

-----------------------------------------------------------------

## Download Snapshot
### 1. Download Snapshot Terbaru
```bash
wget https://rpc-zero-gravity-testnet.trusted-point.com/latest_snapshot.tar.lz4
```

### 2. Stop the node
```bash
sudo systemctl stop ogd
```

### 3. Backup file priv_validator_state.json 
```bash
cp $HOME/.evmosd/data/priv_validator_state.json $HOME/.evmosd/priv_validator_state.json.backup
```

### 4. Reset Database
```bash
evmosd tendermint unsafe-reset-all --home $HOME/.evmosd --keep-addr-book
```

### 5. Ekstrak File Dari Arsip 
```bash
lz4 -d -c ./latest_snapshot.tar.lz4 | tar -xf - -C $HOME/.evmosd
```

### 6. Pindahkan Kembali File priv_validator_state.json
```bash
mv $HOME/.evmosd/priv_validator_state.json.backup $HOME/.evmosd/data/priv_validator_state.json
```

### 7. Restart Node
```bash
sudo systemctl restart ogd && sudo journalctl -u ogd -f -o cat
```

### 8. Cek Status Sinkronisasi
Snapshot diperbarui setiap 3 jam.
```bash
evmosd status | jq .SyncInfo
```

-----------------------------------------------------------------

# Daftar Command
## Pengelolaan Key / Akun
Generate key baru
```
evmosd keys add wallet
```

Pemulihan key
```
evmosd keys add wallet --recover
```

Lihat semua daftar key
```
evmosd keys list
```

Hapus key
```
evmosd keys delete wallet
```

Export key
```
evmosd keys export wallet
```

Import key
```
evmosd keys import wallet wallet.backup
```

Cek saldo wallet
```
evmosd q bank balances $(evmosd keys show wallet -a)
```

-----------------------------------------------------------------

## Pengelolaan Validator
Buat validator
```
evmosd tx staking create-validator \
--amount 10000000000000000aevmos \
--pubkey $(evmosd tendermint show-validator) \
--moniker "nama-moniker" \
--identity "keybase-id" \
--details "info-detail" \
--website "link-website" \
--security-contact "alamat-email" \
--chain-id zgtendermint_9000-1 \
--commission-rate 0.05 \
--commission-max-rate 0.10 \
--commission-max-change-rate 0.01 \
--min-self-delegation 1 \
--from $WALLET_NAME \
--gas 500000 \
--gas-prices 99999aevmos \
-y
```

Edit validator
```
evmosd tx staking edit-validator \
--new-moniker "nama-moniker" \
--identity "keybase-id" \
--details "info-detail" \
--website "link-website" \
--security-contact "alamat-email" \
--chain-id zgtendermint_9000-1 \
--commission-rate 0.05 \
--from $WALLET_NAME \
--gas 500000 \
--gas-prices 99999aevmos \
-y
```

Unjail validator
```
evmosd tx slashing unjail --from $WALLET_NAME --chain-id zgtendermint_9000-1 --gas 500000 --gas-prices 99999aevmos -y
```

Info jail validator
```
evmosd q slashing signing-info $(evmosd tendermint show-validator)
```

Daftar validator aktif
```
evmosd q staking validators -o json --limit=1000 \
| jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' \
| jq -r '.tokens + " - " + .description.moniker' \
| sort -gr | nl
```

Daftar validator inaktif
```
evmosd q staking validators -o json --limit=1000 \
| jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED")' \
| jq -r '.tokens + " - " + .description.moniker' \
| sort -gr | nl
```

Info detail validator
```
evmosd q staking validator $(evmosd keys show $WALLET_NAME --bech val -a) 
```

-----------------------------------------------------------------

## Pengelolaan Token
Penarikan reward dari semua validator
```
evmosd tx distribution withdraw-all-rewards --from $WALLET_NAME --chain-id zgtendermint_9000-1 --gas 500000 --gas-prices 99999aevmos -y
```

Penarikan reward dan komisi
```
evmosd tx distribution withdraw-rewards $(evmosd keys show wallet --bech val -a) --commission --from $WALLET_NAME --chain-id zgtendermint_9000-1 --gas 500000 --gas-prices 99999aevmos -y
```

Delegasikan token ke validator anda
```
evmosd tx staking delegate $(evmosd keys show $WALLET_NAME --bech val -a) 1000000000000000000aevmos --from $WALLET_NAME --chain-id zgtendermint_9000-1 --gas 500000 --gas-prices 99999aevmos -y
```

Delegasikan token ke validator lain, ubah `<to-valoper-address>` dengan alamat validator lain
```
evmosd tx staking delegate <to-valoper-address> 1000000000000000000aevmos --from $WALLET_NAME --chain-id zgtendermint_9000-1 --gas 500000 --gas-prices 99999aevmos -y
```

Pindahkan delegasi token ke validator lain
```
evmosd tx staking redelegate $(evmosd keys show $WALLET_NAME --bech val -a) <to-valoper-address> 1000000000000000000aevmos --from $WALLET_NAME --chain-id zgtendermint_9000-1 --gas 500000 --gas-prices 99999aevmos -y
```

Unbond token dari validator anda
```
evmosd tx staking unbond $(evmosd keys show $WALLET_NAME --bech val -a) 1000000000000000000aevmos --from $WALLET_NAME --chain-id zgtendermint_9000-1 --gas 500000 --gas-prices 99999aevmos -y
```

Kirim token antar wallet
```
evmosd tx bank send wallet <to-wallet-address> 1000000000000000000aevmos --from $WALLET_NAME --chain-id zgtendermint_9000-1 --gas 500000 --gas-prices 99999aevmos -y
```

-----------------------------------------------------------------

<p align="center">
  &copy; 2024 BlockHub. All rights reserved.
</p>