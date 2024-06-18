![0G Github Banner](https://github.com/BlockchainsHub/Testnet/assets/77204008/34a32724-b411-41e4-8696-e390dfa01cab)

# Panduan Node Storage KV 0G
Panduan ini akan membantu anda dalam proses instalasi node storage kv 0G.

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

## Instalasi Node Storage KV
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
git clone https://github.com/0glabs/0g-storage-kv.git
cd 0g-storage-kv
git submodule update --init
cargo build --release
sudo mv "$HOME/0g-storage-kv/target/release/zgs_kv" /usr/local/bin
```

### 5. Buat Directory DB & KV-DB
```bash
mkdir -p "$HOME/0g-storage-kv/db" "$HOME/0g-storage-kv/kv-db"
```

### 6. Buat File Konfigurasi
```bash
cp $HOME/0g-storage-kv/run/config_example.toml $HOME/0g-storage-kv/run/config.toml
```

### 7. Menyiapkan Environment Variables
Jalankan perintah di bawah ini dan masukkan IP dan port storage node Anda dalam format ini `http://x.x.x.x:5678`. Jika Anda menginstal Storage Node dan Storage KV di server yang sama, Anda dapat menggunakan `http://127.0.0.1:5678`.
```bash
read -p "Masukkan IP dan port storage node Anda untuk konfigurasi zgs_node_urls: " ZGS_NODE_URLS
```

Pastikan RPC Validator yang akan Anda gunakan memiliki block `334797` sebelum anda menjalankan Storage KV dengan menjalankan perintah di bawah ini pada validator node Anda. Ganti `yourvalidatorip:port` dengan IP dan port jsonrpc validator anda.
```bash
curl -X POST http://yourvalidatorip:port -H "Content-Type: application/json" -d '{"jsonrpc":"2.0","method":"eth_getBlockByNumber","params":["0x51bcd",false],"id":1}'
```

Jalankan perintah di bawah ini dan masukkan IP dan port validator node Anda dalam format ini `http://x.x.x.x:5678`. 
```bash
read -p "Masukkan IP dan port validator node Anda untuk konfigurasi blockchain_rpc_endpoint: " BLOCKCHAIN_RPC_ENDPOINT
```

```bash
echo 'export DB_DIR="$HOME/0g-storage-kv/db"' >> ~/.bash_profile

echo 'export ZGSKV_DB_DIR="$HOME/0g-storage-kv/kv-db"' >> ~/.bash_profile

echo 'export ZGS_NODE_URLS="'$ZGS_NODE_URLS'"' >> ~/.bash_profile

echo 'export ZGSKV_CONFIG_FILE="$HOME/0g-storage-kv/run/config.toml"' >> ~/.bash_profile

echo 'export ZGSKV_LOG_CONFIG_FILE="$HOME/0g-storage-kv/run/log_config"' >> ~/.bash_profile

echo 'export BLOCKCHAIN_RPC_ENDPOINT="'$BLOCKCHAIN_RPC_ENDPOINT'"' >> ~/.bash_profile

source ~/.bash_profile
```

### 8. Update Konfigurasi
```bash
sed -i "s|^\s*#\?\s*db_dir\s*=.*|db_dir = \"$DB_DIR\"|" "$ZGSKV_CONFIG_FILE"

sed -i "s|^\s*#\?\s*kv_db_dir\s*=.*|kv_db_dir = \"$ZGSKV_DB_DIR\"|" "$ZGSKV_CONFIG_FILE"

sed -i 's|^\s*#\?\s*rpc_listen_address\s*=.*|rpc_listen_address = "0.0.0.0:6789"|' "$ZGSKV_CONFIG_FILE"

sed -i "s|^\s*#\?\s*zgs_node_urls\s*=.*|zgs_node_urls = \"$ZGS_NODE_URLS\"|" "$ZGSKV_CONFIG_FILE"

sed -i "s|^\s*#\?\s*log_config_file\s*=.*|log_config_file = \"$ZGSKV_LOG_CONFIG_FILE\"|" "$ZGSKV_CONFIG_FILE"

sed -i "s|^\s*#\?\s*blockchain_rpc_endpoint\s*=.*|blockchain_rpc_endpoint = \"$BLOCKCHAIN_RPC_ENDPOINT\"|" "$ZGSKV_CONFIG_FILE"

sed -i 's|^\s*#\?\s*log_contract_address\s*=.*|log_contract_address = "0xb8F03061969da6Ad38f0a4a9f8a86bE71dA3c8E7"|' "$ZGSKV_CONFIG_FILE"

sed -i 's|^\s*#\?\s*log_sync_start_block_number\s*=.*|log_sync_start_block_number = 334797|' "$ZGSKV_CONFIG_FILE"
```

### 9. Buat File Service
Buat file service untuk menjalankan storage node di background.
```bash
sudo tee /etc/systemd/system/zgskv.service > /dev/null <<EOF
[Unit]
Description=Node Storage KV 0G
After=network.target

[Service]
User=$USER
Type=simple
ExecStart=/usr/local/bin/zgs_kv --config $HOME/0g-storage-kv/run/config.toml
Restart=on-failure
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

### 10. Start Storage Node
```bash
sudo systemctl daemon-reload && \
sudo systemctl enable zgskv && \
sudo systemctl restart zgskv && \
sudo systemctl status zgskv
```

-----------------------------------------------------------------

## Daftar Command
### Cek Log
```bash
sudo journalctl -u zgskv -f -o cat
```

### Restart Node
```bash
sudo systemctl restart zgskv
```

### Stop Node
```bash
sudo systemctl stop zgskv
```

### Hapus Node
```bash
sudo systemctl stop zgskv
sudo systemctl disable zgskv
sudo rm /etc/systemd/system/zgskv.service
sudo rm /usr/local/bin/zgs_kv
rm -rf $HOME/0g-storage-kv
```

-----------------------------------------------------------------

<p align="center">
  &copy; 2024 BlockHub. All rights reserved.
</p>