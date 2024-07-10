![0G Github Banner](https://github.com/BlockchainsHub/Testnet/assets/77204008/34a32724-b411-41e4-8696-e390dfa01cab)

# Panduan 0G DA
Panduan ini akan membantu Anda dalam proses instalasi node 0G DA.

-----------------------------------------------------------------

## Spesifikasi Hardware Yang Dibutuhkan
| Spesifikasi | Hardware |
|-|-
| CPU | 8 Cores |
| Memory | 16 GB |
| Storage | 1 TB NVMe SSD |
| Bandwidth | 100mbps |
| OS | Linux |

-----------------------------------------------------------------

## Instalasi Node DA
### 1. Instal Packages Yang Dibutuhkan
```bash
sudo apt-get update
sudo apt-get install git cargo clang cmake build-essential pkg-config openssl libssl-dev protobuf-compiler
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

### 3. Build Binary
```bash
git clone https://github.com/0glabs/0g-da-node.git
cd $HOME/0g-da-node
cargo build --release
./dev_support/download_params.sh
sudo mv "$HOME/0g-da-node/target/release/server" /usr/local/bin
```

### 4. Buat Directory DB
```bash
mkdir -p "$HOME/0g-da-node/db"
```

### 5. Buat File Konfigurasi
```bash
cp $HOME/0g-da-node/config_example.toml $HOME/0g-da-node/config.toml
```

### 6. Buat BLS Private Key
Jalankan perintah di bawah ini untuk membuat BLS private key. **Harap simpan BLS private key yang dihasilkan dengan hati-hati**.
```bash
cargo run --bin key-gen
```
Contoh output:
![CleanShot 2024-07-10 at 18 05 17@2x](https://github.com/BlockchainsHub/Testnet/assets/77204008/aaa9ab86-3bb0-4445-9d4a-b75deb2d686d)
> [!CAUTION]
> **JANGAN LUPA UNTUK MENYIMPAN BLS PRIVATE KEY ANDA!**

### 7. Menyiapkan Environment Variables
Jalankan perintah di bawah ini dan masukkan IP dan port node validator Anda dalam format ini `http://x.x.x.x:8545`. 
```bash
read -p "Masukkan IP dan port node validator Anda untuk konfigurasi eth_rpc_endpoint: " BLOCKCHAIN_RPC_ENDPOINT
```

Jalankan perintah di bawah ini dan masukkan IP dan port node DA Anda dalam format ini `http://x.x.x.x:34000`. 
```bash
read -p "Masukkan IP dan port node DA Anda untuk konfigurasi socket_address: " SOCKET_ADDRESS
```

Jalankan perintah di bawah ini dan masukkan BLS private key anda. 
```bash
read -p "Masukkan BLS private key anda untuk konfigurasi signer_bls_private_key: " SIGNER_BLS_PRIVATE_KEY
```

Jalankan perintah di bawah ini dan masukkan private key wallet anda. 
```bash
read -p "Masukkan private key Anda untuk konfigurasi signer_eth_private_key: " SIGNER_ETH_PRIVATE_KEY
```

```bash
echo 'export ZGDA_CONFIG_FILE="$HOME/0g-da-node/config.toml"' >> ~/.bash_profile

echo 'export DB_DIR="$HOME/0g-da-node/db"' >> ~/.bash_profile

echo 'export ZGDA_ENCODER_PARAMS_DIR="$HOME/0g-da-node/params"' >> ~/.bash_profile

echo 'export BLOCKCHAIN_RPC_ENDPOINT="'$BLOCKCHAIN_RPC_ENDPOINT'"' >> ~/.bash_profile

echo 'export SOCKET_ADDRESS="'$SOCKET_ADDRESS'"' >> ~/.bash_profile

echo 'export SIGNER_BLS_PRIVATE_KEY="'$SIGNER_BLS_PRIVATE_KEY'"' >> ~/.bash_profile

echo 'export SIGNER_ETH_PRIVATE_KEY="'$SIGNER_ETH_PRIVATE_KEY'"' >> ~/.bash_profile
```

### 8. Update Konfigurasi
```bash
sed -i "s|^\s*#\?\s*data_path\s*=.*|data_path = \"$DB_DIR\"|" "$ZGDA_CONFIG_FILE"

sed -i "s|^\s*#\?\s*encoder_params_dir\s*=.*|encoder_params_dir = \"$ZGDA_ENCODER_PARAMS_DIR\"|" "$ZGDA_CONFIG_FILE"

sed -i "s|^\s*#\?\s*eth_rpc_endpoint\s*=.*|eth_rpc_endpoint = \"$BLOCKCHAIN_RPC_ENDPOINT\"|" "$ZGDA_CONFIG_FILE"

sed -i "s|^\s*#\?\s*socket_address\s*=.*|socket_address = \"$SOCKET_ADDRESS\"|" "$ZGDA_CONFIG_FILE"

sed -i "s|^\s*#\?\s*da_entrance_address\s*=.*|da_entrance_address = "0xDFC8B84e3C98e8b550c7FEF00BCB2d8742d80a69"|" "$ZGDA_CONFIG_FILE"

sed -i "s|^\s*#\?\s*signer_bls_private_key\s*=.*|signer_bls_private_key = \"$SIGNER_BLS_PRIVATE_KEY\"|" "$ZGDA_CONFIG_FILE"

sed -i "s|^\s*#\?\s*signer_eth_private_key\s*=.*|signer_eth_private_key = \"$SIGNER_ETH_PRIVATE_KEY\"|" "$ZGDA_CONFIG_FILE"
```

### 9. Buat File Service
Buat file service untuk menjalankan node DA di background.
```bash
sudo tee /etc/systemd/system/zgda.service > /dev/null <<EOF
[Unit]
Description=0G DA Node
After=network.target

[Service]
User=$USER
Type=simple
ExecStart=/usr/local/bin/server --config $HOME/0g-da-node/config.toml
Restart=on-failure
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

### 10. Start DA Node
```bash
sudo systemctl daemon-reload && \
sudo systemctl enable zgda && \
sudo systemctl start zgda && \
sudo systemctl status zgda
```

-----------------------------------------------------------------

## Daftar Command
### Cek Log
```bash
sudo journalctl -u zgda -f -o cat
```

### Restart Node
```bash
sudo systemctl restart zgda
```

### Stop Node
```bash
sudo systemctl stop zgda
```

### Hapus Node
```bash
sudo systemctl stop zgda
sudo systemctl disable zgda
sudo rm /etc/systemd/system/zgda.service
sudo rm /usr/local/bin/server
rm -rf $HOME/0g-da-node
```

-----------------------------------------------------------------

<p align="center">
  &copy; 2024 BlockHub. All rights reserved.
</p>
