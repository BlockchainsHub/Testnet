![Eclipse Github Banner](https://github.com/BlockchainsHub/Testnet/assets/77204008/fc25892b-e8ab-4f3e-8b19-430f94420bd8)

# Panduan Testnet Eclipse
Panduan ini akan memandu Anda untuk mendeploy smart contract sederhana ke Eclipse Testnet.

## Instal Dependensi
Anda harus melakukan beberapa hal sebelum dapat mendeploy smart contract Anda ke Eclipse Testnet.

### Instal Rust
Instal Rust, dan package managernya Cargo.
```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

. "$HOME/.cargo/env"
```

Anda dapat memeriksa apakah instalasi berhasil dengan menjalankan command berikut:
```bash
rustc --version
cargo --version
```

### Instal Node.js
Instal NVM (Node Version Manager)
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"
```

Download dan instal node.js
```bash
nvm install 21
```

Verifikasi versi Node.js yang benar telah terinstal. Seharusnya menampilkan versi `v21.7.3`.
```bash
node -v
```

Verifikasi versi NPM yang benar telah terinstal. Seharusnya menampilkan versi `10.5.0`.
```bash
npm -v
```

### Instal Solana CLI
Solana CLI memungkinkan Anda berinteraksi dengan cluster Solana (Eclipse Testnet dalam kasus ini).
```bash
sh -c "$(curl -sSfL https://release.solana.com/stable/install)"
```

Atur Solana CLI Anda ke Eclipse Testnet dengan command berikut:
```bash
solana config set --url https://testnet.dev2.eclipsenetwork.xyz
```

### Instal Yarn
```bash
npm install -g yarn
```

## Deposit Sepolia ETH ke Eclipse Testnet
### Dapatkan Private Key Ethereum Anda
Anda bisa mendapatkan private key Anda dari dompet Metamask dengan membuka menu `Account details` dan klik `Show private key`.

> [!NOTE]
> Pastikan Anda memiliki setidaknya 0,1 Sepolia ETH di dompet Anda.

### Dapatkan Public Address Eclipse Anda
Buat dompet baru:
```bash
solana-keygen new
```
Ini akan menghasilkan dompet baru dan menyimpannya ke mesin lokal Anda. Anda memerlukan dompet ini untuk menandatangani transaksi untuk mendeploy smart contract Anda ke Eclipse Testnet.

> [!CAUTION]
> **PASTIKAN UNTUK MENYIMPAN PRIVATE KEY ANDA!**

Anda dapat melihat alamat Solana yang Anda buat sebelumnya menggunakan Solana CLI:
```bash
solana address
```

### Download Skrip Deposit.js
```bash
git clone https://github.com/Eclipse-Laboratories-Inc/testnet-deposit
cd testnet-deposit
```

Instal dependensi dengan yarn.
```bash
yarn install
```

### Lakukan Deposit
Pastikan untuk mengubah variabel ini pada perintah di bawah ini:
- **[ADDRESS_SOLANA_ANDA]** : Ganti dengan alamat Solana yang Anda buat sebelumnya
- **[PRIVATE_KEY_ETHEREUM_ANDA]** : Ganti dengan private key yang Anda dapatkan dari dompet Metamask Anda.

```bash
node deposit.js [ADDRESS_SOLANA_ANDA] 0x7C9e161ebe55000a3220F972058Fb83273653a6e 1500000 100 [PRIVATE_KEY_ETHEREUM_ANDA] https://rpc.sepolia.org
```

Contoh command:
```bash
node deposit.js yyiecymjvwYE6Wxg4ZSt4ibdfFRo3JUNhQ99AKnaRqu 0x7C9e161ebe55000a3220F972058Fb83273653a6e 15000000 100 3e1bf180e4778c7944f509b422711101186d26ac15337934f12088623755c0b7 https://rpc.sepolia.org/
```

Jika transaksi Anda berhasil, Anda akan mendapatkan hasil transaksi seperti ini.
```bash
Transaction successful: 0xb05a37f4e4b420f651fdffb0b169ba96cb8c8e201b32f3d8d0c94705d7dc6d5f
```

Anda dapat memeriksa transaksi Anda di [Sepolia Testnet Explorer](https://sepolia.etherscan.io/).

Anda dapat memverifikasi saldo akun testnet Eclipse Anda dengan menjalankan command berikut:
```bash
solana balance
```

## Mendeploy Smart Contract
Clone repositori Solana Hello World dan instal dependensinya.
```bash
cd $HOME
git clone https://github.com/solana-labs/example-helloworld
cd example-helloworld
npm install
```

Build smart contract:
```bash
npm run build:program-rust
```

Deploy smart contract ke Eclipse Testnet:
```bash
solana program deploy dist/program/helloworld.so
```

Jalankan JavaScript client dan konfirmasi apakah smart contract berhasil dideploy:
```bash
npm run start
```

Outputnya harus seperti ini:
```bash
Let's say hello to a Solana account...
Connection to cluster established: http://127.0.0.1:8899 { 'feature-set': 2045430982, 'solana-core': '1.7.8' }
Using account AiT1QgeYaK86Lf9kudqKthQPCWwpG8vFA1bAAioBoF4X containing 0.00141872 SOL to pay for fees
Using program Dro9uk45fxMcKWGb1eWALujbTssh6DW8mb4x8x3Eq5h6
Creating account 8MBmHtJvxpKdYhdw6yPpedp6X6y2U9dCpdYaZJdmwV3A to say hello to
Saying hello to 8MBmHtJvxpKdYhdw6yPpedp6X6y2U9dCpdYaZJdmwV3A
8MBmHtJvxpKdYhdw6yPpedp6X6y2U9dCpdYaZJdmwV3A has been greeted 1 times
Success
```

Salin program addressnya `8MBmHtJvxpKdYhdw6yPpedp6X6y2U9dCpdYaZJdmwV3A` dan kirimkan ke [formulir](https://forms.gle/yJfFABQDPmpvgzAf7) ini.

Selamat! Anda telah berhasil mendeploy smart contract ke Eclipse Testnet.