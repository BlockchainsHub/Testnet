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

## Terhubung dengan Tanssi

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>SVG Theme Mode</title>
  <style>
    :root {
      --light-color: black;
      --dark-color: white;
    }

    /* Dark Mode */
    @media (prefers-color-scheme: dark) {
      :root {
        --light-color: white;
        --dark-color: black;
      }
    }

    /* Center the SVG */
    .center {
      display: flex;
      justify-content: space-between;
      align-items: center;
      max-width: 500px; /* Set a maximum width to limit the space */
      margin: 0 auto; /* Center the container */
    }

    /* SVG colors */
    .svg-path {
      fill: var(--light-color); /* Default color */
      transition: fill 0.3s ease; /* Smooth transition */
    }

    /* Dark mode SVG color */
    @media (prefers-color-scheme: dark) {
      .svg-path {
        fill: var(--dark-color); /* Dark mode color */
      }
    }
  </style>
</head>
<body>
  <div class="center">
    <div>
      <!-- Website Logo -->
      <a href="https://www.tanssi.network/">
        <svg class="svg-path" xmlns="http://www.w3.org/2000/svg" id="Outline" version="1.0" viewBox="0 0 24 24" width="16" height="16">
          <g>
            <path d="M12,0A12,12,0,1,0,24,12,12.013,12.013,0,0,0,12,0Zm8.647,7H17.426a19.676,19.676,0,0,0-2.821-4.644A10.031,10.031,0,0,1,20.647,7ZM16.5,12a10.211,10.211,0,0,1-.476,3H7.976A10.211,10.211,0,0,1,7.5,12a10.211,10.211,0,0,1,.476-3h8.048A10.211,10.211,0,0,1,16.5,12ZM8.778,17h6.444A19.614,19.614,0,0,1,12,21.588,19.57,19.57,0,0,1,8.778,17Zm0-10A19.614,19.614,0,0,1,12,2.412,19.57,19.57,0,0,1,15.222,7ZM9.4,2.356A19.676,19.676,0,0,0,6.574,7H3.353A10.031,10.031,0,0,1,9.4,2.356ZM2.461,9H5.9a12.016,12.016,0,0,0-.4,3,12.016,12.016,0,0,0,.4,3H2.461a9.992,9.992,0,0,1,0-6Zm.892,8H6.574A19.676,19.676,0,0,0,9.4,21.644,10.031,10.031,0,0,1,3.353,17Zm11.252,4.644A19.676,19.676,0,0,0,17.426,17h3.221A10.031,10.031,0,0,1,14.605,21.644ZM21.539,15H18.1a12.016,12.016,0,0,0,.4-3,12.016,12.016,0,0,0-.4-3h3.437a9.992,9.992,0,0,1,0,6Z"/>
          </g>
        </svg>
      </a>
    </div>
    <div>
      <!-- Twitter Logo -->
      <a href="https://twitter.com/TanssiNetwork">
        <svg class="svg-path" xmlns="http://www.w3.org/2000/svg" id="Capa_1" data-name="Capa 1" version="1.0" viewBox="0 0 24 24" width="16" height="16">
          <g>
            <path d="m18.9,1.153h3.682l-8.042,9.189,9.46,12.506h-7.405l-5.804-7.583-6.634,7.583H.469l8.6-9.831L0,1.153h7.593l5.241,6.931,6.065-6.931Zm-1.293,19.494h2.039L6.482,3.239h-2.19l13.314,17.408Z"/>
          </g>
        </svg>
      </a>
    </div> 
    <div>  
      <!-- Discord Logo -->
      <a href="https://discord.com/invite/kuyPhew2KB">
        <svg class="svg-path" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1" id="Capa_1" viewBox="0 0 24 24" width="16" height="16">
          <g>
            <path d="M20.317,4.37c-1.53-0.702-3.17-1.219-4.885-1.515c-0.031-0.006-0.062,0.009-0.079,0.037   c-0.211,0.375-0.445,0.865-0.608,1.249c-1.845-0.276-3.68-0.276-5.487,0C9.095,3.748,8.852,3.267,8.641,2.892   C8.624,2.864,8.593,2.85,8.562,2.855C6.848,3.15,5.208,3.667,3.677,4.37C3.664,4.375,3.652,4.385,3.645,4.397   c-3.111,4.648-3.964,9.182-3.546,13.66c0.002,0.022,0.014,0.043,0.031,0.056c2.053,1.508,4.041,2.423,5.993,3.029   c0.031,0.01,0.064-0.002,0.084-0.028c0.462-0.63,0.873-1.295,1.226-1.994c0.021-0.041,0.001-0.09-0.042-0.106   c-0.653-0.248-1.274-0.55-1.872-0.892c-0.047-0.028-0.051-0.095-0.008-0.128c0.126-0.094,0.252-0.192,0.372-0.291   c0.022-0.018,0.052-0.022,0.078-0.01c3.928,1.793,8.18,1.793,12.061,0c0.026-0.012,0.056-0.009,0.079,0.01   c0.12,0.099,0.246,0.198,0.373,0.292c0.044,0.032,0.041,0.1-0.007,0.128c-0.598,0.349-1.219,0.645-1.873,0.891   c-0.043,0.016-0.061,0.066-0.041,0.107c0.36,0.698,0.772,1.363,1.225,1.993c0.019,0.027,0.053,0.038,0.084,0.029   c1.961-0.607,3.95-1.522,6.002-3.029c0.018-0.013,0.029-0.033,0.031-0.055c0.5-5.177-0.838-9.674-3.548-13.66   C20.342,4.385,20.33,4.375,20.317,4.37z M8.02,15.331c-1.183,0-2.157-1.086-2.157-2.419s0.955-2.419,2.157-2.419   c1.211,0,2.176,1.095,2.157,2.419C10.177,14.246,9.221,15.331,8.02,15.331z M15.995,15.331c-1.182,0-2.157-1.086-2.157-2.419   s0.955-2.419,2.157-2.419c1.211,0,2.176,1.095,2.157,2.419C18.152,14.246,17.206,15.331,15.995,15.331z"/>
          </g>
        </svg>
      </a>
    </div>
    <div>
      <!-- Telegram Logo -->
      <a href="https://t.me/tanssiofficial">
        <svg class="svg-path" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1" id="Capa_1" x="0px" y="0px" viewBox="0 0 24 24" style="enable-background:new 0 0 24 24;" xml:space="preserve" width="16" height="16">
          <g id="Artboard">
            <path style="fill-rule:evenodd;clip-rule:evenodd;" d="M12,0C5.373,0,0,5.373,0,12s5.373,12,12,12s12-5.373,12-12S18.627,0,12,0z    M17.562,8.161c-0.18,1.897-0.962,6.502-1.359,8.627c-0.168,0.9-0.5,1.201-0.82,1.23c-0.697,0.064-1.226-0.461-1.901-0.903   c-1.056-0.692-1.653-1.123-2.678-1.799c-1.185-0.781-0.417-1.21,0.258-1.911c0.177-0.184,3.247-2.977,3.307-3.23   c0.007-0.032,0.015-0.15-0.056-0.212s-0.174-0.041-0.248-0.024c-0.106,0.024-1.793,1.139-5.062,3.345   c-0.479,0.329-0.913,0.489-1.302,0.481c-0.428-0.009-1.252-0.242-1.865-0.442c-0.751-0.244-1.349-0.374-1.297-0.788   c0.027-0.216,0.324-0.437,0.892-0.663c3.498-1.524,5.831-2.529,6.998-3.015c3.333-1.386,4.025-1.627,4.477-1.635   C17.472,7.214,17.608,7.681,17.562,8.161z"/>
          </g>
        </svg>
      </a>
    </div>
    <div>
      <!-- Reddit Logo -->
      <a href="https://www.reddit.com/r/TanssiNetwork/">
        <svg class="svg-path" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1" id="Capa_1" x="0px" y="0px" viewBox="0 0 24 24" style="enable-background:new 0 0 24 24;" xml:space="preserve" width="16" height="16">
          <g>
            <path d="M9.25,14.5C8.561,14.5,8,13.939,8,13.25C8,12.561,8.561,12,9.25,12c0.689,0,1.25,0.561,1.25,1.25   C10.5,13.939,9.939,14.5,9.25,14.5z"/>
	          <path d="M14.97,16.095c0.127,0.127,0.126,0.332,0,0.458c-0.853,0.852-2.488,0.918-2.969,0.918c-0.481,0-2.116-0.066-2.968-0.918   c-0.127-0.127-0.127-0.331,0-0.458c0.127-0.127,0.331-0.127,0.458,0c0.538,0.538,1.688,0.728,2.51,0.728   c0.822,0,1.972-0.191,2.511-0.729C14.639,15.968,14.844,15.968,14.97,16.095z"/>
	          <path d="M16,13.25c0,0.69-0.561,1.25-1.25,1.25c-0.69,0-1.25-0.561-1.25-1.25c0-0.689,0.561-1.25,1.25-1.25   C15.439,12,16,12.561,16,13.25z"/>
	          <path d="M12,0C5.373,0,0,5.373,0,12s5.373,12,12,12s12-5.373,12-12S18.627,0,12,0z M18.957,13.599   c0.027,0.173,0.041,0.348,0.041,0.526c0,2.692-3.134,4.875-7,4.875c-3.866,0-7-2.183-7-4.875c0-0.179,0.015-0.355,0.042-0.529   C4.431,13.322,4.006,12.711,4.006,12c0-0.967,0.783-1.75,1.75-1.75c0.47,0,0.896,0.186,1.21,0.488   c1.212-0.873,2.886-1.431,4.749-1.483l0.89-4.185c0.017-0.081,0.066-0.152,0.135-0.197c0.069-0.045,0.154-0.061,0.235-0.044   l2.908,0.618C16.088,5.036,16.509,4.75,17,4.75c0.69,0,1.25,0.56,1.25,1.25S17.69,7.25,17,7.25c-0.67,0-1.213-0.529-1.244-1.191   l-2.604-0.554l-0.797,3.75c1.836,0.064,3.484,0.622,4.68,1.485c0.315-0.303,0.742-0.491,1.214-0.491c0.967,0,1.75,0.783,1.75,1.75   C19.998,12.714,19.57,13.327,18.957,13.599z"/>
          </g>
        </svg>
      </a>
    </div>
    <div>
      <!-- Github Logo -->
      <a href="https://github.com/moondance-labs/tanssi">
        <svg class="svg-path" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1" id="Capa_1" x="0px" y="0px" viewBox="0 0 24 24" style="enable-background:new 0 0 24 24;" xml:space="preserve" width="16" height="16">
          <g>
            <path style="fill-rule:evenodd;clip-rule:evenodd;" d="M12,0.296c-6.627,0-12,5.372-12,12c0,5.302,3.438,9.8,8.206,11.387   c0.6,0.111,0.82-0.26,0.82-0.577c0-0.286-0.011-1.231-0.016-2.234c-3.338,0.726-4.043-1.416-4.043-1.416   C4.421,18.069,3.635,17.7,3.635,17.7c-1.089-0.745,0.082-0.729,0.082-0.729c1.205,0.085,1.839,1.237,1.839,1.237   c1.07,1.834,2.807,1.304,3.492,0.997C9.156,18.429,9.467,17.9,9.81,17.6c-2.665-0.303-5.467-1.332-5.467-5.93   c0-1.31,0.469-2.381,1.237-3.221C5.455,8.146,5.044,6.926,5.696,5.273c0,0,1.008-0.322,3.301,1.23   C9.954,6.237,10.98,6.104,12,6.099c1.02,0.005,2.047,0.138,3.006,0.404c2.29-1.553,3.297-1.23,3.297-1.23   c0.653,1.653,0.242,2.873,0.118,3.176c0.769,0.84,1.235,1.911,1.235,3.221c0,4.609-2.807,5.624-5.479,5.921   c0.43,0.372,0.814,1.103,0.814,2.222c0,1.606-0.014,2.898-0.014,3.293c0,0.319,0.216,0.694,0.824,0.576   c4.766-1.589,8.2-6.085,8.2-11.385C24,5.669,18.627,0.296,12,0.296z"/>
	          <path d="M4.545,17.526c-0.026,0.06-0.12,0.078-0.206,0.037c-0.087-0.039-0.136-0.121-0.108-0.18   c0.026-0.061,0.12-0.078,0.207-0.037C4.525,17.384,4.575,17.466,4.545,17.526L4.545,17.526z"/>
	          <path d="M5.031,18.068c-0.057,0.053-0.169,0.028-0.245-0.055c-0.079-0.084-0.093-0.196-0.035-0.249   c0.059-0.053,0.167-0.028,0.246,0.056C5.076,17.903,5.091,18.014,5.031,18.068L5.031,18.068z"/>
	          <path d="M5.504,18.759c-0.074,0.051-0.194,0.003-0.268-0.103c-0.074-0.107-0.074-0.235,0.002-0.286   c0.074-0.051,0.193-0.005,0.268,0.101C5.579,18.579,5.579,18.707,5.504,18.759L5.504,18.759z"/>
	          <path d="M6.152,19.427c-0.066,0.073-0.206,0.053-0.308-0.046c-0.105-0.097-0.134-0.234-0.068-0.307   c0.067-0.073,0.208-0.052,0.311,0.046C6.191,19.217,6.222,19.355,6.152,19.427L6.152,19.427z"/>
	          <path d="M7.047,19.814c-0.029,0.094-0.164,0.137-0.3,0.097C6.611,19.87,6.522,19.76,6.55,19.665   c0.028-0.095,0.164-0.139,0.301-0.096C6.986,19.609,7.075,19.719,7.047,19.814L7.047,19.814z"/>
	          <path d="M8.029,19.886c0.003,0.099-0.112,0.181-0.255,0.183c-0.143,0.003-0.26-0.077-0.261-0.174c0-0.1,0.113-0.181,0.256-0.184   C7.912,19.708,8.029,19.788,8.029,19.886L8.029,19.886z"/>
	          <path d="M8.943,19.731c0.017,0.096-0.082,0.196-0.224,0.222c-0.139,0.026-0.268-0.034-0.286-0.13   c-0.017-0.099,0.084-0.198,0.223-0.224C8.797,19.574,8.925,19.632,8.943,19.731L8.943,19.731z"/>
          </g>
        </svg>
      </a>
    </div>
    <!-- Youtube Logo -->
      <a href="https://www.youtube.com/@TanssiNetwork?sub_confirmation=1">
        <svg class="svg-path" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1" id="Capa_1" x="0px" y="0px" viewBox="0 0 24 24" style="enable-background:new 0 0 24 24;" xml:space="preserve" width="16" height="16">
          <g id="XMLID_184_">
            <path d="M23.498,6.186c-0.276-1.039-1.089-1.858-2.122-2.136C19.505,3.546,12,3.546,12,3.546s-7.505,0-9.377,0.504   C1.591,4.328,0.778,5.146,0.502,6.186C0,8.07,0,12,0,12s0,3.93,0.502,5.814c0.276,1.039,1.089,1.858,2.122,2.136   C4.495,20.454,12,20.454,12,20.454s7.505,0,9.377-0.504c1.032-0.278,1.845-1.096,2.122-2.136C24,15.93,24,12,24,12   S24,8.07,23.498,6.186z M9.546,15.569V8.431L15.818,12L9.546,15.569z"/>
          </g>
        </svg>
      </a>
    </div>
  </div>
  <script>
    const prefersDarkScheme = window.matchMedia("(prefers-color-scheme: dark)");

    // Function to toggle color based on user's preference
    function setColorScheme() {
      if (prefersDarkScheme.matches) {
        document.documentElement.style.setProperty('--light-color', 'white');
        document.documentElement.style.setProperty('--dark-color', 'black');
      } else {
        document.documentElement.style.setProperty('--light-color', 'black');
        document.documentElement.style.setProperty('--dark-color', 'white');
      }
    }

    // Initialize color scheme
    setColorScheme();

    // Listen for changes in color scheme
    prefersDarkScheme.addListener(setColorScheme);
  </script>
</body>
</html>

-----------------------------------------------------------------

<p align="center">
  &copy; 2024 BlockHub. All rights reserved.
</p>