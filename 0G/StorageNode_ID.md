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
curl -o zgs_ID.sh https://gist.githubusercontent.com/botxx15/c32f78e4e30bf3efa13d5c544b1c6cc7/raw/32fe3e9b348b88651c935d7bb5b43b1608ee9f8e/zgs_ID.sh && bash zgs_ID.sh
```

Setelah instalasi selesai, anda dapat melanjutkan ke langkah [Cek Log](#Cek-Log) untuk memeriksa log Anda.

-----------------------------------------------------------------

## Instalasi Storage Node
### 1. Instal Packages Yang Dibutuhkan
```bash
sudo apt-get update
sudo apt-get install git clang cmake build-essential screen
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
ver="1.21.3" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile && \
source ~/.bash_profile && \
go version
```

### 4. Build Binary
```bash
git clone https://github.com/0glabs/0g-storage-node.git
cd 0g-storage-node
git submodule update --init
cargo build --release
cd run
```

### 5. Buat Miner ID
Buat `miner_id` dengan menjalankan command berikut ini. Silahkan rubah `your_name` dengan nama yang ingin anda gunakan.
```bash
echo -n your_name | sha256sum
```
Contoh output:
![CleanShot 2024-04-13 at 15 44 55@2x](https://github.com/BlockchainsHub/Testnet/assets/77204008/520bd6ff-5f62-4684-8d6e-d8f9bb9281a5)

> [!NOTE]
> Simpanlah `miner_id` yang sudah anda buat karena akan digunakan untuk konfigurasi miner.

### 6. Update Konfigurasi
```bash
nano $HOME/0g-storage-node/run/config.toml
```

Hapus tanda `#` yang ada pada baris `miner_id` dan `miner_key` kemudian isi data `miner_id` dan `miner_key`.

> [!NOTE]
> Data `miner_key` diisi dengan private key wallet anda. Anda dapat melihat private key wallet melalui Metamask.

Contoh setelan konfigurasi:
![CleanShot 2024-04-13 at 15 52 52@2x](https://github.com/BlockchainsHub/Testnet/assets/77204008/55272fec-d9e4-4151-a6cd-be619cc53023)

Setelah selesai mengatur file konfigurasi anda dapat menyimpannya dengan menekan tombol `ctrl+x`,`y`,`enter`

### 7. Buat Screen Baru
Buat screen baru untuk menjalankan storage node di background.
```bash
screen -S zgs
```

### 8. Jalankan Storage Node
```bash
../target/release/zgs_node --config config.toml
```

Setelah menjalankan command diatas anda bisa keluar dari screen dengan menekan tombol `ctrl+a+d`

### 9. Cek Log
Buka directory log.
```bash
cd $HOME/0g-storage-node/run/log
```

Lihat daftar log yang ada.
```bash
ls
```
![CleanShot 2024-04-13 at 16 34 05@2x](https://github.com/BlockchainsHub/Testnet/assets/77204008/6123290a-0ea9-4cc3-907c-3aaac9990961)

Buka file log yang ada. Silahkan rubah `nama_log` pada command dibawah dengan nama log yang muncul setelah menjalankan command `ls`
```bash
cat nama_log
```

Contoh command:
```bash
cat zgs.log.2024-04-13
```

Output dari file log akan muncul seperti digambar berikut ini.

Log output:
![CleanShot 2024-04-13 at 16 45 36@2x](https://github.com/BlockchainsHub/Testnet/assets/77204008/70870e65-2add-46fb-b24b-2865f168db09)
