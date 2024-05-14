![Tanssi Banner](https://github.com/BlockchainsHub/Testnet/assets/77204008/dc2ec5eb-0d98-4e99-8254-9438ad7294fc)

# Panduan Block Producer Tanssi
Panduan ini akan membantu Anda dalam proses instalasi node Block Producer Tanssi, pastikan Anda memenuhi spesifikasi yang diperlukan dan mengikuti langkah instalasi yang benar untuk kinerja optimal.

-----------------------------------------------------------------

## Spesifikasi Hardware Yang Dibutuhkan
Pastikan sistem Anda memenuhi atau melampaui persyaratan berikut untuk dapat menghosting node Block Producer Tanssi secara efektif:
| Spesifikasi | Hardware |
|-|-
| CPU | Intel Xeon E-2386/2388 or Ryzen 9 5950x/5900x |
| Memory | 32 GB |
| Storage | 1 TB NVMe SSD |
| Bandwidth | 100mbps |
| OS | Linux |

-----------------------------------------------------------------

## Proses Instalasi Node
### 1. Instalasi Dependensi
Mulai dengan memperbarui sistem Anda dan menginstal dependensi yang diperlukan. Buka terminal Anda dan jalankan:
```bash
apt update && apt upgrade -y
apt install curl iptables build-essential git wget jq make gcc nano tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev libgmp3-dev tar clang bsdmainutils ncdu unzip llvm libudev-dev make protobuf-compiler -y
```

### 2. Download Rilis Terbaru
Download dan siapkan software node Tanssi:
```bash
wget https://github.com/moondance-labs/tanssi/releases/download/v0.6.0/tanssi-node && \
chmod +x ./tanssi-node
```

### 3. Buat Dompet Baru
Buat dompet baru dengan seed phrase 24 kata:
```bash
./tanssi-node key generate -w24
```

> [!IMPORTANT]
> Setelah membuat baru dompet Anda, Anda akan menerima seed phrase 24 kata. Sangat penting untuk menyimpan seed phrase ini dengan aman dan secara pribadi, karena ini memberikan akses ke dompet Anda dan tidak boleh dibagikan.

### 4. Siapkan Direktori Data Tanssi
Konfigurasikan system service user dan direktori data:
```bash
adduser tanssi_service --system --no-create-home
mkdir /var/lib/tanssi-data
sudo chown -R tanssi_service /var/lib/tanssi-data
mv ./tanssi-node /var/lib/tanssi-data
```

### 5. Siapkan Environment Variable `$TANSSI_NODE_NAME`
Untuk memberi nama yang tepat pada node Tanssi Anda, Anda perlu mengatur environment variable `$TANSSI_NODE_NAME` di sesi terminal Anda. Variabel ini akan menyimpan nama node Anda, yang akan digunakan dalam service file systemd. Jalankan perintah berikut di terminal Anda, ganti `MyTanssiNode` dengan nama yang diinginkan untuk node Tanssi Anda:
```bash
export TANSSI_NODE_NAME="MyTanssiNode"
```

> [!NOTE]
> Ganti `MyTanssiNode` dengan nama yang diinginkan untuk node Tanssi Anda. Nilai ini akan digunakan sepanjang proses pengaturan node.

### 6. Buat Service File systemd
Dengan variabel `$TANSSI_NODE_NAME` yang sudah diatur, Anda sekarang dapat memasukkan nama ini secara dinamis ke dalam service file systemd Anda. Gunakan perintah berikut untuk membuat dan menulis konfigurasi kustom Anda ke dalam service file systemd:
```bash
cat <<EOF | sudo tee /etc/systemd/system/tanssi.service > /dev/null
[Unit]
Description=Tanssi systemd service
After=network.target
StartLimitIntervalSec=0

[Service]
Type=simple
Restart=on-failure
RestartSec=10
User=tanssi_service
SyslogIdentifier=tanssi
SyslogFacility=local7
KillSignal=SIGHUP
ExecStart=/var/lib/tanssi-data/tanssi-node \
--chain=dancebox \
--name=$TANSSI_NODE_NAME \
--sync=warp \
--base-path=/var/lib/tanssi-data/para \
--state-pruning=2000 \
--blocks-pruning=2000 \
--collator \
--telemetry-url='wss://telemetry.polkadot.io/submit/ 0' \
--database paritydb \
-- \
--name=$TANSSI_NODE_NAME \
--base-path=/var/lib/tanssi-data/container \
--telemetry-url='wss://telemetry.polkadot.io/submit/ 0' \
-- \
--chain=westend_moonbase_relay_testnet \
--name=$TANSSI_NODE_NAME \
--sync=fast \
--base-path=/var/lib/tanssi-data/relay \
--state-pruning=2000 \
--blocks-pruning=2000 \
--telemetry-url='wss://telemetry.polkadot.io/submit/ 0' \
--database paritydb

[Install]
WantedBy=multi-user.target
EOF
```

### 7. Aktifkan dan Jalankan Service File
Setelah membuat file systemd, Anda perlu memastikan sistem mengenali dan mengelola service baru dengan benar:
```bash
sudo systemctl daemon-reload
sudo systemctl enable tanssi.service
sudo systemctl start tanssi.service
```

### 8. Verifikasi Status Service
Periksa apakah node Tanssi Anda berjalan dengan benar dengan konfigurasi baru:
```bash
sudo systemctl status tanssi.service
```

Anda juga dapat memantau log yang berjalan dari service Anda:
```bash
journalctl -f -u tanssi.service
```

-----------------------------------------------------------------

## Buat Session Keys
Buat session keys baru dengan mengirim permintaan JSON-RPC:
```bash
curl http://127.0.0.1:9944 -H \
"Content-Type:application/json;charset=utf-8" -d \
  '{
    "jsonrpc":"2.0",
    "id":1,
    "method":"author_rotateKeys",
    "params": []
  }'
```

-----------------------------------------------------------------

## Update Node
Untuk memperbarui node, hentikan service, download rilis terbaru, dan restart:
```bash
systemctl stop tanssi.service
wget https://github.com/moondance-labs/tanssi/releases/download/v0.6.0/tanssi-node && \
chmod +x ./tanssi-node
mv ./tanssi-node /var/lib/tanssi-data
systemctl restart tanssi.service && journalctl -f -u tanssi.service
```

Kunjungi [Halaman Tag GitHub Tanssi](https://github.com/moondance-labs/tanssi/tags) untuk melihat versi rilis terbaru dari node Tanssi.

-----------------------------------------------------------------

<p align="center">
  &copy; 2024 BlockHub. All rights reserved.
</p>

<div align="center">
  <a href="https://0g.ai/"><img src="https://github.com/BlockchainsHub/Assets/blob/df18afa279d057a963038450da568bd82e84f06f/Outlined/Twitter%20icon.png" alt="0G Website" width="16%"></a>
  <a href="https://github.com/0glabs"><img src="https://github.com/bartosian/celestia-tools/assets/20209819/229ec400-72ff-48ee-ac18-7bdb1f5e221a" alt="0G Github" width="16%"></a>
  <a href="https://twitter.com/0G_labs"><img src="https://github.com/bartosian/celestia-tools/assets/20209819/3978b7fc-d575-44a6-8d41-327c14c8ba31" alt="0G Twitter" width="16%"></a>
  <a href="https://discord.gg/0glabs"><img src="https://github.com/bartosian/celestia-tools/assets/20209819/944a0b87-548d-4109-ad0c-def572d307cb" alt="0G Discord" width="16%"></a>
  <a href="https://blog.0g.ai/"><img src="https://github.com/bartosian/celestia-tools/assets/20209819/ac52729b-64d7-44d1-9a66-1e0d159848f6" alt="0G Blog" width="16.2%"></a>
</div>