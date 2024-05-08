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
PEERS="a4055b828e59832c7a06d61fc51347755a160d0b@157.90.33.62:21656" && \
SEEDS="c4d619f6088cb0b24b4ab43a0510bf9251ab5d7f@54.241.167.190:26656,44d11d4ba92a01b520923f51632d2450984d5886@54.176.175.48:26656,f2693dd86766b5bf8fd6ab87e2e970d564d20aff@54.193.250.204:26656,f878d40c538c8c23653a5b70f615f8dccec6fb9f@54.215.187.94:26656" && \
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.0gchaind/config/config.toml
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
  --amount=100000000000000000ua0gi \
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

<p align="center">
  &copy; 2024 BlockHub. All rights reserved.
</p>
