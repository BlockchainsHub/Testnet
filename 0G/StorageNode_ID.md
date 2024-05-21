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

## Instalasi Storage Node Otomatis
```
curl -o zgs_ID.sh https://gist.githubusercontent.com/botxx15/c32f78e4e30bf3efa13d5c544b1c6cc7/raw/e0f3618db4167000cc113ab5a1215dca2a2d0038/zgs_ID.sh && bash zgs_ID.sh
```

Setelah instalasi selesai, anda dapat melanjutkan ke langkah [Cek Log](#Cek-Log-Terbaru) untuk memeriksa log Anda.

-----------------------------------------------------------------

## Instalasi Storage Node
### 1. Instal Packages Yang Dibutuhkan
```bash
sudo apt-get update
sudo apt-get install git cargo clang cmake build-essential
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
git clone --recurse-submodules https://github.com/0glabs/0g-storage-node.git
cd 0g-storage-node
cargo build --release
sudo mv "$HOME/0g-storage-node/target/release/zgs_node" /usr/local/bin
```

### 5. Menyiapkan Environment Variables
```bash
cat <<EOF >> ~/.bash_profile
export ZGS_CONFIG_FILE="$HOME/0g-storage-node/run/config.toml"
export ZGS_LOG_DIR="$HOME/0g-storage-node/run/log"
export ZGS_LOG_CONFIG_FILE="$HOME/0g-storage-node/run/log_config"
EOF

source ~/.bash_profile
```

### 6. Masukkan miner_key Dalam Variabel
```bash
read -p "Masukkan private key anda untuk konfigurasi miner_key: " PRIVATE_KEY && echo
```

### 7. Update Konfigurasi
```bash
sed -i 's|#* network_dir = "network"|network_dir = "network"|' "$ZGS_CONFIG_FILE"

sed -i 's|#* network_libp2p_port = 1234|network_libp2p_port = 1234|' "$ZGS_CONFIG_FILE"

if grep -q '# network_boot_nodes' "$ZGS_CONFIG_FILE"; then
    sed -i 's|# network_boot_nodes = \["/ip4/54.219.26.22/udp/1234/p2p/16Uiu2HAmPxGNWu9eVAQPJww79J32pTJLKGcpjRMb4Qb8xxKkyuG1","/ip4/52.52.127.117/udp/1234/p2p/16Uiu2HAm93Hd5azfhkGBbkx1zero3nYHvfjQYM2NtiW4R3r5bE2g"\]|network_boot_nodes = \["/ip4/54.219.26.22/udp/1234/p2p/16Uiu2HAmTVDGNhkHD98zDnJxQWu3i1FL1aFYeh9wiQTNu4pDCgps","/ip4/52.52.127.117/udp/1234/p2p/16Uiu2HAkzRjxK2gorngB1Xq84qDrT4hSVznYDHj6BkbaE4SGx9oS"\]|' "$ZGS_CONFIG_FILE"
elif grep -q 'network_boot_nodes' "$ZGS_CONFIG_FILE"; then
    sed -i 's|network_boot_nodes = \["/ip4/54.219.26.22/udp/1234/p2p/16Uiu2HAmPxGNWu9eVAQPJww79J32pTJLKGcpjRMb4Qb8xxKkyuG1","/ip4/52.52.127.117/udp/1234/p2p/16Uiu2HAm93Hd5azfhkGBbkx1zero3nYHvfjQYM2NtiW4R3r5bE2g"\]|network_boot_nodes = \["/ip4/54.219.26.22/udp/1234/p2p/16Uiu2HAmTVDGNhkHD98zDnJxQWu3i1FL1aFYeh9wiQTNu4pDCgps","/ip4/52.52.127.117/udp/1234/p2p/16Uiu2HAkzRjxK2gorngB1Xq84qDrT4hSVznYDHj6BkbaE4SGx9oS"\]|' "$ZGS_CONFIG_FILE"
fi

if grep -q '# blockchain_rpc_endpoint' "$ZGS_CONFIG_FILE"; then
    sed -i 's|# blockchain_rpc_endpoint =.*|blockchain_rpc_endpoint = "https://og-testnet-jsonrpc.blockhub.id:443"|' "$ZGS_CONFIG_FILE"
elif grep -q 'blockchain_rpc_endpoint' "$ZGS_CONFIG_FILE"; then
    sed -i 's|blockchain_rpc_endpoint =.*|blockchain_rpc_endpoint = "https://og-testnet-jsonrpc.blockhub.id:443"|' "$ZGS_CONFIG_FILE"
else
    echo 'blockchain_rpc_endpoint = "https://og-testnet-jsonrpc.blockhub.id:443"' >> "$ZGS_CONFIG_FILE"
fi

if grep -q '# log_sync_start_block_number' "$ZGS_CONFIG_FILE"; then
    sed -i 's|# log_sync_start_block_number =.*|log_sync_start_block_number = 80981|' "$ZGS_CONFIG_FILE"
elif grep -q 'log_sync_start_block_number' "$ZGS_CONFIG_FILE"; then
    sed -i 's|log_sync_start_block_number =.*|log_sync_start_block_number = 80981|' "$ZGS_CONFIG_FILE"
fi

sed -i 's|#* db_dir = "db"|db_dir = "db"|' "$ZGS_CONFIG_FILE"

if ! grep -q "^log_config_file" "$ZGS_CONFIG_FILE"; then
    sed -i "/^#* log_config_file/c\log_config_file = \"$ZGS_LOG_CONFIG_FILE\"" "$ZGS_CONFIG_FILE"
fi

if ! grep -q "^log_directory" "$ZGS_CONFIG_FILE"; then
    sed -i "/^#* log_directory/c\log_directory = \"$ZGS_LOG_DIR\"" "$ZGS_CONFIG_FILE"
fi

if grep -q 'miner_key\|# miner_key' "$ZGS_CONFIG_FILE"; then
    sed -i "/#*miner_key/c\miner_key = \"$PRIVATE_KEY\"" "$ZGS_CONFIG_FILE"
fi
```

### 8. Buat File Service
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

### 9. Start Storage Node
```bash
sudo systemctl daemon-reload && \
sudo systemctl enable zgs && \
sudo systemctl restart zgs && \
sudo systemctl status zgs
```

-----------------------------------------------------------------

## Daftar Command
### Cek Log Terbaru
```bash
tail -n 100 "$ZGS_LOG_DIR/$(ls -Art $ZGS_LOG_DIR | tail -n 1)"
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

### Upgrade Node
```bash
cd $HOME/0g-storage-node
git fetch --tags
git checkout $(git describe --tags $(git rev-list --tags --max-count=1))
git submodule update --init
cargo build --release
sudo mv $HOME/0g-storage-node/target/release/zgs_node /usr/local/bin
zgs_node --version
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
rm -rf $HOME/0g-storage-node
```

-----------------------------------------------------------------

<p align="center">
  &copy; 2024 BlockHub. All rights reserved.
</p>