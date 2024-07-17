![0G Github Banner](https://github.com/BlockchainsHub/Testnet/assets/77204008/34a32724-b411-41e4-8696-e390dfa01cab)

# 0G DA Client Guide
Panduan ini akan membantu Anda dalam proses instalasi Klien 0G DA.

-----------------------------------------------------------------

## Spesifikasi Hardware Yang Dibutuhkan
| Spesifikasi | Hardware |
|-|-
| CPU | 2 Cores |
| Memory | 8 GB |
| Bandwidth | 100mbps |
| OS | Linux |

-----------------------------------------------------------------

## Instalasi DA Client
### 1. Instal Packages Yang Dibutuhkan
```bash
sudo apt-get update
sudo apt-get install cmake screen
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
git clone https://github.com/0glabs/0g-da-client.git
cd 0g-da-client
make build
```

### 4. Buat File Service
Buat file service untuk menjalankan DA client di background. Silahkan ubah variabel berikut:

| Variabel | Keterangan |
|-|-
| $CHAIN_RPC | Ubah dengan IP dan Port validator Anda dalam format `http://ip_validator_anda:8545` |
| $CHAIN_PRIVATE_KEY | Ubah dengan private key validator Anda |
| $ENCODER_SOCKET | Ubah dengan IP dan Port Node DA Anda dalam format ini `http://ip_da_node_anda:8545` |

```bash
sudo tee /etc/systemd/system/zgdac.service > /dev/null <<EOF
[Unit]
Description=0G DA Client Service
After=network.target

[Service]
Type=simple
User=root
WorkingDirectory=/root/0g-da-client
ExecStart=/root/0g-da-client/disperser/bin/combined \
    --chain.rpc $CHAIN_RPC \
    --chain.private-key $CHAIN_PRIVATE_KEY \
    --chain.receipt-wait-rounds 180 \
    --chain.receipt-wait-interval 1s \
    --chain.gas-limit 2000000 \
    --combined-server.use-memory-db \
    --combined-server.storage.kv-db-path ./root/0g-storage-kv/run \
    --combined-server.storage.time-to-expire 300 \
    --disperser-server.grpc-port 51001 \
    --batcher.da-entrance-contract 0xDFC8B84e3C98e8b550c7FEF00BCB2d8742d80a69 \
    --batcher.da-signers-contract 0x0000000000000000000000000000000000001000 \
    --batcher.finalizer-interval 20s \
    --batcher.confirmer-num 3 \
    --batcher.max-num-retries-for-sign 3 \
    --batcher.finalized-block-count 50 \
    --batcher.batch-size-limit 500 \
    --batcher.encoding-interval 3s \
    --batcher.encoding-request-queue-size 1 \
    --batcher.pull-interval 10s \
    --batcher.signing-interval 3s \
    --batcher.signed-pull-interval 20s \
    --encoder-socket $ENCODER_SOCKET \
    --encoding-timeout 600s \
    --signing-timeout 600s \
    --chain-read-timeout 12s \
    --chain-write-timeout 13s \
    --combined-server.log.level-file trace \
    --combined-server.log.level-std trace
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
EOF
```

### 5. Start DA Client
```bash
sudo systemctl daemon-reload && \
sudo systemctl enable zgdac && \
sudo systemctl start zgdac && \
sudo systemctl status zgdac
```

-----------------------------------------------------------------

## Daftar Command
### Cek Log
```bash
sudo journalctl -u zgdac -f -o cat
```

### Restart DA Client
```bash
sudo systemctl restart zgdac
```

### Stop DA Client
```bash
sudo systemctl stop zgdac
```

### Hapus DA Client
```bash
sudo systemctl stop zgdac
sudo systemctl disable zgdac
sudo rm /etc/systemd/system/zgdac.service
rm -rf $HOME/0g-da-client
```

-----------------------------------------------------------------

<p align="center">
  &copy; 2024 BlockHub. All rights reserved.
</p>