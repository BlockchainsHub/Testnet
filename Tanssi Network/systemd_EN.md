# Tanssi Block Producer Guide
This guide will assist you in the installation process of a Tanssi Block Producer node, ensuring you meet the necessary specifications and follow correct installation steps for optimal performance.

-----------------------------------------------------------------

## Required Specification
Ensure your system meets or exceeds the following requirements to host a Tanssi Block Producer node effectively:
| Required | Specification |
|-|-
| CPU | Intel Xeon E-2386/2388 or Ryzen 9 5950x/5900x |
| Memory | 32 GB |
| Storage | 1 TB NVMe SSD |
| Bandwidth | 100mbps |
| OS | Linux |

-----------------------------------------------------------------

## Node Installation Process
### 1. Install Dependencies
Begin by updating your system and installing necessary dependencies. Open your terminal and execute:
```bash
apt update && apt upgrade -y
apt install curl iptables build-essential git wget jq make gcc nano tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev libgmp3-dev tar clang bsdmainutils ncdu unzip llvm libudev-dev make protobuf-compiler -y
```

### 2. Download the Latest Release
Download and prepare the Tanssi node software:
```bash
wget https://github.com/moondance-labs/tanssi/releases/download/v0.6.0/tanssi-node && \
chmod +x ./tanssi-node
```

### 3. Create New Wallet
Generate a new wallet with a 24-word seed phrase:
```bash
./tanssi-node key generate -w24
```

> [!IMPORTANT]
> After creating your wallet, you will receive a 24-word seed phrase. It is crucial to store this seed phrase securely and privately, as it provides access to your wallet and should never be shared.

### 4. Set Up Tanssi Data Directory
Configure the system service user and data directory:
```bash
adduser tanssi_service --system --no-create-home
mkdir /var/lib/tanssi-data
sudo chown -R tanssi_service /var/lib/tanssi-data
mv ./tanssi-node /var/lib/tanssi-data
```

### 5. Set Up the `$TANSSI_NODE_NAME` Environment Variable
To properly name your Tanssi node, you need to set the `$TANSSI_NODE_NAME` environment variable in your terminal session. This variable will hold the name of your node, which will be used in the systemd service file. Execute the following command in your terminal, replacing `MyTanssiNode` with the desired name for your Tanssi node:
```bash
export TANSSI_NODE_NAME="MyTanssiNode"
```

> [!NOTE]
> Replace `MyTanssiNode` with the desired name for your Tanssi node. This value will be used throughout the node setup process.

### 6. Create the systemd Service File 
With the `$TANSSI_NODE_NAME` variable set, you can now dynamically insert this name into your systemd service file. Use the following command to create and write your custom configuration into the systemd service file:
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

### 7. Enable and Start the Service
After creating the systemd file, you need to ensure the system recognizes and manages the new service correctly:
```bash
sudo systemctl daemon-reload
sudo systemctl enable tanssi.service
sudo systemctl start tanssi.service
```

### 8. Verify the Service Status
To check that your Tanssi node is running properly with the new configuration:
```bash
sudo systemctl status tanssi.service
```

You can also monitor the running logs of your service:
```bash
journalctl -f -u tanssi.service
```

-----------------------------------------------------------------

## Generate Session Keys
Generate new session keys by sending a JSON-RPC request:
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
To update the node, stop the service, download the latest release, and restart:
```bash
systemctl stop tanssi.service
wget https://github.com/moondance-labs/tanssi/releases/download/v0.6.0/tanssi-node && \
chmod +x ./tanssi-node
mv ./tanssi-node /var/lib/tanssi-data
systemctl restart tanssi.service && journalctl -f -u tanssi.service
```

Visit the [Tanssi GitHub Tags page](https://github.com/moondance-labs/tanssi/tags) to view the latest release version of the Tanssi node.

-----------------------------------------------------------------

<p align="center">
  &copy; 2024 BlockHub. All rights reserved.
</p>