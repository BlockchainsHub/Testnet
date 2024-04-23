![RISC Zero Banner](https://github.com/BlockchainsHub/Testnet/assets/77204008/ee01db53-478b-4eb6-85ce-1bc532d46825)

# Panduan Ceremony Contribution RISC Zero
Panduan ini akan membantu Anda untuk berpartisipasi dalam Ceremony Contribution RISC Zero.

-----------------------------------------------------------

## System Requirements
| System | Requirements |
|-|-
| OS | Linux |
| Memory | 8 GB |
| Bandwidth | At least 25 Mbps |

-----------------------------------------------------------

## Github Requirements
Sebelum melanjutkan, pastikan akun GitHub Anda memenuhi persyaratan ini.
- Mengikuti 5 orang
- Diikuti oleh minimal 1 orang
- Memiliki 2 public repository
- Akun Anda setidaknya berumur satu bulan

-----------------------------------------------------------

## Instal Dependencies
### Instal Node.js
Instal NVM (Node Version Manager)
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"
```

Download dan instal Node.js
```bash
nvm install 20
```

Verifikasi versi Node.js yang benar ada di environment. Seharusnya menampilkan versi `v20.12.2`.
```bash
node -v
```

Verifikasi versi NPM yang tepat ada di environment. Seharusnya menampilkan versi `10.5.0`.
```bash
npm -v
```

### Instal Tmux
```bash
sudo apt install tmux
```

-----------------------------------------------------------

## Contributing To The Ceremony
### 1. Buat Direktori Sementara
```bash
mkdir ~/p0tion-tmp
cd ~/p0tion-tmp
```

### 2. Beralih Ke Node v16.20
```bash
nvm install 16.20
nvm use 16.20
```

### 3. Buat Sesi Tmux Baru
```bash
tmux
```

### 4. Instal phase2cli
```bash
npm i @p0tion/phase2cli
```

### 5. Otentikasi dengan GitHub
```bash
npx phase2cli auth
```
Ini akan menyalin kode autentikasi ke clipboard Anda, membuka browser web Anda, dan membawa Anda ke situs GitHub yang meminta izin untuk menggunakan fitur "Gists" GitHub Anda. Paste konten clipboard Anda ke situs web ini dan klik tombol untuk memberi otorisasi. GitHub akan memberi tahu Anda “Congratulations, you’re all set!” dan Anda dapat kembali ke terminal.

### 6. Contribute
```bash
npx phase2cli contribute
```

Anda akan ditanya ceremony mana yang harus disumbangkan.

Ceremony `RISC Zero STARK-to_SNARK Prover` seharusnya sudah disorot, jadi tekan `enter` untuk memilihnya.

Sekarang, Anda akan diminta untuk memilih cara menambahkan entropi — ini adalah data rahasia yang harus Anda hancurkan dan lupakan (atau idealnya tidak pernah tahu) untuk mengamankan ceremony. Anda dapat memilih `randomly` atau `manually`. Jika Anda tidak tahu harus memilih apa, pilih saja `randomly`.

Setelah Anda menambahkan entropi, Anda akan ditempatkan di antrean kontribusi. Kontribusi memerlukan waktu lama, jadi biarkan proses ini berjalan di latar belakang atau lakukan hal lain. (Tetapi jangan biarkan komputer Anda tertidur, komputer yang tertidur tidak dapat berkontribusi!)

Sekarang Anda dapat melepaskan diri dari sesi Tmux. Untuk melepaskan diri dari sesi saat ini (membiarkannya berjalan di latar belakang), tekan `Ctrl + b`, lalu lepaskan kedua tombol dan tekan `d`.

Untuk menyambungkan kembali ke sesi terpisah, ketik `tmux attach` di terminal Anda.

> [!NOTE]
> ‍Jika Anda mengalami error atau kehilangan koneksi, Anda dapat mencoba memulai ulang kontribusi Anda dengan menjalankan kembali perintah kontribusi `npx phase2cli`. Kontribusi Anda seharusnya dilanjutkan di bagian terakhirnya.

**Setelah kontribusi Anda selesai, Anda akan diminta untuk membagikan pesan di Twitter/X atau di platform media sosial apa pun yang Anda sukai! 🎉**

### 7. Pembersihan
Setelah kontribusi ceremony Anda selesai, mungkin ada baiknya Anda membersihkan file dan otorisasi GitHub.
```bash
npx phase2cli clean
```
```bash
npx phase2cli logout
```

Buka `authorized app list di GitHub` dan hapus izin untuk `pse-p0tion-production`. Anda juga dapat menghapus folder `~/p0tion-tmp` yang Anda buat jika Anda mau.

-----------------------------------------------------------------

<p align="center">
  &copy; 2024 BlockHub. All rights reserved.
</p>