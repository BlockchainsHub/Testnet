![0G Github Banner](https://github.com/BlockchainsHub/Testnet/assets/77204008/34a32724-b411-41e4-8696-e390dfa01cab)

# 0G Storage Node Guide
This guide will help you in the 0G storage node installation process.

-----------------------------------------------------------------

## Required Hardware Specifications
| Required | Specification |
|-|-
| CPU | 4 Cores |
| Memory | 16 GB |
| Storage | 500 GB NVMe SSD |
| Bandwidth | 100mbps |
| OS | Linux |

-----------------------------------------------------------------

## Storage Node Installation
### 1. Install Dependencies
```bash
sudo apt-get update
sudo apt-get install git cargo clang cmake build-essential
```

### 2. Install Rustup
```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

You will be asked for Rust installation options, just use the default one. Press `1` then `enter` to continue the installation process.
![CleanShot 2024-04-13 at 15 07 52@2x](https://github.com/BlockchainsHub/Testnet/assets/77204008/bcb81284-8235-4cf2-a4f1-50821044cc21)

After the installation process is complete, run the following command, to restart the shell.
```bash
. "$HOME/.cargo/env"
```
![CleanShot 2024-04-13 at 15 13 17@2x](https://github.com/BlockchainsHub/Testnet/assets/77204008/f8f94656-0f1f-4d27-b347-3842b2b77a6f)

### 3. Install GO
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
git clone -b v0.3.4 https://github.com/0glabs/0g-storage-node.git
cd 0g-storage-node
git submodule update --init
cargo build --release
sudo mv "$HOME/0g-storage-node/target/release/zgs_node" /usr/local/bin
```

### 5. Set Up Environment Variables
```bash
ENR_ADDRESS=$(wget -qO- eth0.me)
echo "export ENR_ADDRESS=${ENR_ADDRESS}"
```

```bash
cat <<EOF >> ~/.bash_profile
export ENR_ADDRESS=${ENR_ADDRESS}
export ZGS_CONFIG_FILE="$HOME/0g-storage-node/run/config.toml"
export ZGS_LOG_DIR="$HOME/0g-storage-node/run/log"
export ZGS_LOG_CONFIG_FILE="$HOME/0g-storage-node/run/log_config"
EOF

source ~/.bash_profile
```

### 6. Store Miner Key
```bash
read -p "Enter your private key for miner_key configuration: " PRIVATE_KEY && echo
```

### 7. Create Network & DB Directory
```bash
mkdir -p "$HOME/0g-storage-node/network" "$HOME/0g-storage-node/db"
```

### 8. Update Config File
```bash
sed -i 's|^\s*#\?\s*network_dir\s*=.*|network_dir = "/root/0g-storage-node/network"|' "$ZGS_CONFIG_FILE"

sed -i "s|^\s*#\?\s*network_enr_address\s*=.*|network_enr_address = \"$ENR_ADDRESS\"|" "$ZGS_CONFIG_FILE"

sed -i 's|^\s*#\?\s*network_enr_tcp_port\s*=.*|network_enr_tcp_port = 1234|' "$ZGS_CONFIG_FILE"

sed -i 's|^\s*#\?\s*network_enr_udp_port\s*=.*|network_enr_udp_port = 1234|' "$ZGS_CONFIG_FILE"

sed -i 's|^\s*#\?\s*network_libp2p_port\s*=.*|network_libp2p_port = 1234|' "$ZGS_CONFIG_FILE"

sed -i 's|^\s*#\?\s*network_discovery_port\s*=.*|network_discovery_port = 1234|' "$ZGS_CONFIG_FILE"

sed -i 's|^\s*#\?\s*network_target_peers\s*=.*|network_target_peers = 50|' "$ZGS_CONFIG_FILE"

sed -i 's|^\s*#\?\s*blockchain_rpc_endpoint\s*=.*|blockchain_rpc_endpoint = "https://og-testnet-jsonrpc.blockhub.id"|' "$ZGS_CONFIG_FILE"

sed -i 's|^\s*#\?\s*log_contract_address\s*=.*|log_contract_address = "0x8873cc79c5b3b5666535C825205C9a128B1D75F1"|' "$ZGS_CONFIG_FILE"

sed -i 's|^\s*#\?\s*log_sync_start_block_number\s*=.*|log_sync_start_block_number = 802|' "$ZGS_CONFIG_FILE"

sed -i 's|^\s*#\?\s*rpc_enabled\s*=\s*true|rpc_enabled = true|' "$ZGS_CONFIG_FILE"

sed -i 's|^\s*#\?\s*rpc_listen_address\s*=\s*"0.0.0.0:5678"|rpc_listen_address = "0.0.0.0:5678"|' "$ZGS_CONFIG_FILE"

sed -i 's|^\s*#\?\s*db_dir\s*=.*|db_dir = "/root/0g-storage-node/db"|' "$ZGS_CONFIG_FILE"

sed -i 's|^\s*#\?\s*log_config_file\s*=.*|log_config_file = "/root/0g-storage-node/run/log_config"|' "$ZGS_CONFIG_FILE"

sed -i 's|^\s*#\?\s*log_directory\s*=.*|log_directory = "/root/0g-storage-node/run/log"|' "$ZGS_CONFIG_FILE"

sed -i 's|^\s*#\?\s*mine_contract_address\s*=.*|mine_contract_address = "0x85F6722319538A805ED5733c5F4882d96F1C7384"|' "$ZGS_CONFIG_FILE"

sed -i "s|^\s*#\?\s*miner_key\s*=.*|miner_key = \"$PRIVATE_KEY\"|" "$ZGS_CONFIG_FILE"
```

### 9. Create Service File
Create a service file to run the storage node in the background.
```bash
sudo tee /etc/systemd/system/zgs.service > /dev/null <<EOF
[Unit]
Description=0G Storage Node
After=network.target

[Service]
User=$USER
Type=simple
ExecStart=/usr/local/bin/zgs_node --config $HOME/0g-storage-node/run/config.toml
Restart=on-failure
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

### 10. Start Storage Node
```bash
sudo systemctl daemon-reload && \
sudo systemctl enable zgs && \
sudo systemctl start zgs && \
sudo systemctl status zgs
```

After checking the your node logs and make sure it's up and running, you can continue to [Storage Node CLI](#storage-node-cli) to upload a test file.

-----------------------------------------------------------------

## Useful Commands
### Check Latest Log
```bash
tail -n 100 "$ZGS_LOG_DIR/$(ls -Art $ZGS_LOG_DIR | tail -n 1)"
```

### Check Peers Log
```bash
grep 'peers' "$ZGS_LOG_DIR/$(ls -Art $ZGS_LOG_DIR | tail -n 1)"
```

### Check Tx Sequence Log
```bash
grep 'tx_seq' "$ZGS_LOG_DIR/$(ls -Art $ZGS_LOG_DIR | tail -n 1)"
```

### Look for Errors
```bash
grep "Error" $ZGS_LOG_DIR/zgs.log.*
```

### List Logs by Date
```bash
ls -lt $ZGS_LOG_DIR
```

### View Specific Date Logs
```bash
cat $ZGS_LOG_DIR/zgs.log.2024-04-15
```

### Restart the Node
```bash
sudo systemctl restart zgs
```

### Stop the Node
```bash
sudo systemctl stop zgs
```

### Backup Miner Key
You can backup your `miner_key` by saving the output value of the command below.
```bash
grep 'miner_key' $ZGS_CONFIG_FILE
```

### Delete the Node
> [!CAUTION]
> **Make sure to backup your `miner_key` before deleting the node!**
```bash
sudo systemctl stop zgs
sudo systemctl disable zgs
sudo rm /etc/systemd/system/zgs.service
sudo rm /usr/local/bin/zgs_node
rm -rf $HOME/0g-storage-node
```

-----------------------------------------------------------------

## Storage Node CLI
### Build Binary
```bash
cd $HOME
git clone https://github.com/0glabs/0g-storage-client.git
cd 0g-storage-client
go build
sudo mv "$HOME/0g-storage-client/0g-storage-client" /usr/local/bin
```

### Generate a test file
```bash
0g-storage-client gen --file test.txt
```

### Upload file
Please change `YOURVALIDATORIP:8545` with your actual validator IP address and port and `YOURSTORAGEIP:5678` with your actual storage node IP address and port.
```bash
0g-storage-client upload \
--url http://YOURVALIDATORIP:8545 \
--contract "0x8873cc79c5b3b5666535C825205C9a128B1D75F1" \
--key $PRIVATE_KEY \
--node http://YOURSTORAGEIP:5678 \
--file test.txt
```

-----------------------------------------------------------------

<p align="center">
  &copy; 2024 BlockHub. All rights reserved.
</p>
