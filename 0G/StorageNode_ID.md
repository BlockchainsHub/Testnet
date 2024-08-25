![0G Github Banner](https://github.com/BlockchainsHub/Testnet/assets/77204008/34a32724-b411-41e4-8696-e390dfa01cab)

# Panduan Storage Node 0G
Panduan ini akan membantu anda dalam proses instalasi storage node 0G.

-----------------------------------------------------------------

## Spesifikasi Hardware Yang Dibutuhkan
| Spesifikasi | Hardware |
|-|-
| CPU | 4 Cores |
| Memory | 16 GB |
| Storage | 500 GB NVMe SSD |
| Bandwidth | 100mbps |
| OS | Linux |

-----------------------------------------------------------------

## Instalasi Storage Node
### 1. Instal Packages Yang Dibutuhkan
```bash
sudo apt-get update
sudo apt-get install git cargo clang cmake build-essential libssl-dev
sudo apt install pkg-config
```

### 2. Instal Rustup
```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

Anda akan ditanyakan opsi instalasi Rust, cukup pakai yang default. Tekan `1` kemudian `enter` untuk melanjutkan proses instalasi.
![CleanShot 2024-04-13 at 15 07 52@2x](https://github.com/BlockchainsHub/Testnet/assets/77204008/bcb81284-8235-4cf2-a4f1-50821044cc21)

Setelah proses instalasi selesai, jalankan command berikut, untuk restart shell.
```bash
. "$HOME/.cargo/env"
```
![CleanShot 2024-04-13 at 15 13 17@2x](https://github.com/BlockchainsHub/Testnet/assets/77204008/f8f94656-0f1f-4d27-b347-3842b2b77a6f)

### 3. Instal GO
```bash
cd $HOME && \
ver="1.22.0" && \
sudo rm -rf /usr/local/go && \
sudo curl -fsSL "https://golang.org/dl/go$ver.linux-amd64.tar.gz" | sudo tar -C /usr/local -xzf - && \
grep -qxF 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' ~/.bash_profile || echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> ~/.bash_profile && \
source ~/.bash_profile && \
go version
```

### 4. Build Binary
```bash
cd $HOME
git clone -b v0.4.5 https://github.com/0glabs/0g-storage-node.git
cd 0g-storage-node
git submodule update --init
cargo build --release
sudo mv "$HOME/0g-storage-node/target/release/zgs_node" /usr/local/bin
```

### 5. Menyiapkan Environment Variables
Simpan `miner_key` ke dalam variabel.
```bash
read -p "Masukkan private key anda untuk konfigurasi miner_key: " PRIVATE_KEY && echo
```

Simpan json-rpc validator Anda ke dalam variabel. Anda dapat menggunakan RPC `http://18.166.164.232:8545/` Node3 untuk melakukan sinkronisasi awal dan mengubahnya ke milik Anda setelah sinkronisasi penuh.
```bash
read -p "Masukkan json-rpc validator Anda untuk konfigurasi blockchain_rpc_endpoint:" JSONRPC && echo
```

Simpan `ENR Address` anda ke dalam variabel.
```bash
ENR_ADDRESS=$(wget -qO- eth0.me)
echo "export ENR_ADDRESS=${ENR_ADDRESS}"
```

```bash
cat <<EOF >> ~/.bash_profile
export ENR_ADDRESS=${ENR_ADDRESS}
export ZGS_CONFIG_FILE="$HOME/0g-storage-node/run/config.toml"
export ZGS_LOG_DIR="$HOME/0g-storage-node/run/log"
export ZGS_LOG_CONFIG_FILE="$HOME/0g-storage-node/run/log_config"
EOF

source ~/.bash_profile
```

### 6. Buat Directory Network & DB
```bash
mkdir -p "$HOME/0g-storage-node/network" "$HOME/0g-storage-node/db"
```

### 7. Buat file config.toml
```bash
cp $HOME/0g-storage-node/run/config-testnet-turbo.toml $HOME/0g-storage-node/run/config.toml
```

### 8. Update Konfigurasi
```bash
sed -i 's|^\s*#\?\s*network_dir\s*=.*|network_dir = "/root/0g-storage-node/network"|' "$ZGS_CONFIG_FILE"

sed -i "s|^\s*#\?\s*network_enr_address\s*=.*|network_enr_address = \"$ENR_ADDRESS\"|" "$ZGS_CONFIG_FILE"

sed -i "s|^\s*#\?\s*blockchain_rpc_endpoint\s*=.*|blockchain_rpc_endpoint = \"$JSONRPC\"|" "$ZGS_CONFIG_FILE"

sed -i 's|^\s*#\?\s*db_dir\s*=.*|db_dir = "/root/0g-storage-node/db"|' "$ZGS_CONFIG_FILE"

sed -i 's|^\s*#\?\s*log_config_file\s*=.*|log_config_file = "/root/0g-storage-node/run/log_config"|' "$ZGS_CONFIG_FILE"

sed -i 's|^\s*#\?\s*log_directory\s*=.*|log_directory = "/root/0g-storage-node/run/log"|' "$ZGS_CONFIG_FILE"

sed -i "s|^\s*#\?\s*miner_key\s*=.*|miner_key = \"$PRIVATE_KEY\"|" "$ZGS_CONFIG_FILE"
```

### 9. Buat File Service
Buat file service untuk menjalankan storage node di background.
```bash
sudo tee /etc/systemd/system/zgs.service > /dev/null <<EOF
[Unit]
Description=0G Storage Node
After=network.target

[Service]
User=$USER
Type=simple
ExecStart=/usr/local/bin/zgs_node --config $HOME/0g-storage-node/run/config.toml
Restart=on-failure
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

### 10. Start Storage Node
```bash
sudo systemctl daemon-reload && \
sudo systemctl enable zgs && \
sudo systemctl start zgs && \
sudo systemctl status zgs
```

Setelah memeriksa log node Anda, pastikan sudah aktif dan berjalan, selanjutnya Anda dapat melanjutkan ke [Storage Node CLI](#storage-node-cli) untuk upload file test.

-----------------------------------------------------------------

## Daftar Command
### Cek Log Terbaru
```bash
tail -n 100 "$ZGS_LOG_DIR/$(ls -Art $ZGS_LOG_DIR | tail -n 1)"
```

### Cek Log Peers
```bash
grep 'peers' "$ZGS_LOG_DIR/$(ls -Art $ZGS_LOG_DIR | tail -n 1)"
```

### Cek Log Tx Sequence
```bash
grep 'tx_seq' "$ZGS_LOG_DIR/$(ls -Art $ZGS_LOG_DIR | tail -n 1)"
```

### Cek Error
```bash
grep "Error" $ZGS_LOG_DIR/zgs.log.*
```

### Daftar Log Berdasarkan Tanggal
```bash
ls -lt $ZGS_LOG_DIR
```

### Lihat Log Tanggal Tertentu
```bash
cat $ZGS_LOG_DIR/zgs.log.2024-04-15
```

### Restart Node
```bash
sudo systemctl restart zgs
```

### Stop Node
```bash
sudo systemctl stop zgs
```

### Backup Miner Key
Anda dapat membackup `miner_key` dengan menyimpan nilai output dari command di bawah ini.
```bash
grep 'miner_key' $ZGS_CONFIG_FILE
```

### Hapus Node
> [!CAUTION]
> **Pastikan untuk membackup `miner_key` anda sebelum menghapus node!**
```bash
sudo systemctl stop zgs
sudo systemctl disable zgs
sudo rm /etc/systemd/system/zgs.service
sudo rm /usr/local/bin/zgs_node
rm -rf $HOME/0g-storage-node
```

-----------------------------------------------------------------

## Upgrade Storage Node
## 1. Stop Node
```bash
sudo systemctl stop zgs
```

## 2. Upgrade Node
```bash
cd $HOME/0g-storage-node
git stash
git fetch --all --tags
git checkout v0.4.5
git submodule update --init
cargo build --release
sudo mv "$HOME/0g-storage-node/target/release/zgs_node" /usr/local/bin
```

## 3. Menyiapkan Environment Variables
Simpan `miner_key` ke dalam variabel.
```bash
read -p "Masukkan private key anda untuk konfigurasi miner_key: " PRIVATE_KEY && echo
```

Simpan json-rpc validator Anda ke dalam variabel. Anda dapat menggunakan RPC `http://18.166.164.232:8545/` Node3 untuk melakukan sinkronisasi awal dan mengubahnya ke milik Anda setelah sinkronisasi penuh.
```bash
read -p "Masukkan json-rpc validator Anda untuk konfigurasi blockchain_rpc_endpoint:" JSONRPC && echo
```

Simpan `ENR Address` anda ke dalam variabel.
```bash
ENR_ADDRESS=$(wget -qO- eth0.me)
echo "export ENR_ADDRESS=${ENR_ADDRESS}"
```

```bash
cat <<EOF >> ~/.bash_profile
export ENR_ADDRESS=${ENR_ADDRESS}
export ZGS_CONFIG_FILE="$HOME/0g-storage-node/run/config.toml"
export ZGS_LOG_DIR="$HOME/0g-storage-node/run/log"
export ZGS_LOG_CONFIG_FILE="$HOME/0g-storage-node/run/log_config"
EOF

source ~/.bash_profile
```

## 4. Buat file config.toml
```bash
cp $HOME/0g-storage-node/run/config-testnet-turbo.toml $HOME/0g-storage-node/run/config.toml
```

### 5. Update Konfigurasi
```bash
sed -i 's|^\s*#\?\s*network_dir\s*=.*|network_dir = "/root/0g-storage-node/network"|' "$ZGS_CONFIG_FILE"

sed -i "s|^\s*#\?\s*network_enr_address\s*=.*|network_enr_address = \"$ENR_ADDRESS\"|" "$ZGS_CONFIG_FILE"

sed -i "s|^\s*#\?\s*blockchain_rpc_endpoint\s*=.*|blockchain_rpc_endpoint = \"$JSONRPC\"|" "$ZGS_CONFIG_FILE"

sed -i 's|^\s*#\?\s*db_dir\s*=.*|db_dir = "/root/0g-storage-node/db"|' "$ZGS_CONFIG_FILE"

sed -i 's|^\s*#\?\s*log_config_file\s*=.*|log_config_file = "/root/0g-storage-node/run/log_config"|' "$ZGS_CONFIG_FILE"

sed -i 's|^\s*#\?\s*log_directory\s*=.*|log_directory = "/root/0g-storage-node/run/log"|' "$ZGS_CONFIG_FILE"

sed -i "s|^\s*#\?\s*miner_key\s*=.*|miner_key = \"$PRIVATE_KEY\"|" "$ZGS_CONFIG_FILE"
```

## 6. Start Node
```bash
systemctl start zgs
```

## 7. Cek Log
```
tail -n 100 "$ZGS_LOG_DIR/$(ls -Art $ZGS_LOG_DIR | tail -n 1)"
```

-----------------------------------------------------------------

## Storage Node CLI
### Build Binary
```bash
cd $HOME
git clone https://github.com/0glabs/0g-storage-client.git
cd 0g-storage-client
go build
sudo mv "$HOME/0g-storage-client/0g-storage-client" /usr/local/bin
```

### Buat file test
```bash
0g-storage-client gen --file test.txt
```

### Upload file
Mohon untuk merubah `YOURVALIDATORIP:8545` dengan IP address dan port validator anda dan `YOURSTORAGEIP:5678` dengan IP address dan port storage node anda.
```bash
0g-storage-client upload \
--url http://YOURVALIDATORIP:8545 \
--contract "0xbD2C3F0E65eDF5582141C35969d66e34629cC768" \
--key $PRIVATE_KEY \
--node http://YOURSTORAGEIP:5678 \
--file test.txt
```

### Download file
Mohon untuk merubah `YOURSTORAGEIP:5678` dengan IP address dan port storage node anda dan `$ROOTH_HASH` dengan root hash yang ada dapatkan pada saat membuat file test.
```bash
0g-storage-client download \
--node http://YOURSTORAGEIP:5678 \
--root $ROOTH_HASH \
--file test_downloaded.txt \
--proof
```

-----------------------------------------------------------------

<p align="center">
  &copy; 2024 BlockHub. All rights reserved.
</p>
