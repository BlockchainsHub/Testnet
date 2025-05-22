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
sudo apt-get install git cargo clang cmake build-essential libssl-dev
sudo apt install pkg-config
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
git clone -b v0.4.5 https://github.com/0glabs/0g-storage-node.git
cd 0g-storage-node
git submodule update --init
cargo build --release
sudo mv "$HOME/0g-storage-node/target/release/zgs_node" /usr/local/bin
```

### 5. Set Up Environment Variables
Store your Miner Key.
```bash
read -p "Enter your private key for miner_key configuration: " PRIVATE_KEY && echo
```

Store your validator json-rpc. You can use Node3's RPC `http://18.166.164.232:8545/` to sync and change to yours after full sync.
```bash
read -p "Enter your validator json-rpc for blockchain_rpc_endpoint configuration: " JSONRPC && echo
```

Store your ENR Address.
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

### 6. Create Network & DB Directory
```bash
mkdir -p "$HOME/0g-storage-node/network" "$HOME/0g-storage-node/db"
```

### 7. Create config.toml file
```bash
cp $HOME/0g-storage-node/run/config-testnet-turbo.toml $HOME/0g-storage-node/run/config.toml
```

### 8. Update Config File
```bash
sed -i 's|^\s*#\?\s*network_dir\s*=.*|network_dir = "/root/0g-storage-node/network"|' "$ZGS_CONFIG_FILE"

sed -i "s|^\s*#\?\s*network_enr_address\s*=.*|network_enr_address = \"$ENR_ADDRESS\"|" "$ZGS_CONFIG_FILE"

sed -i "s|^\s*#\?\s*blockchain_rpc_endpoint\s*=.*|blockchain_rpc_endpoint = \"$JSONRPC\"|" "$ZGS_CONFIG_FILE"

sed -i 's|^\s*#\?\s*db_dir\s*=.*|db_dir = "/root/0g-storage-node/db"|' "$ZGS_CONFIG_FILE"

sed -i 's|^\s*#\?\s*log_config_file\s*=.*|log_config_file = "/root/0g-storage-node/run/log_config"|' "$ZGS_CONFIG_FILE"

sed -i 's|^\s*#\?\s*log_directory\s*=.*|log_directory = "/root/0g-storage-node/run/log"|' "$ZGS_CONFIG_FILE"

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

## Storage Node Upgrade
## 1. Stop The Node
```bash
sudo systemctl stop zgs
```

## 2. Upgrade The Node
```bash
cd $HOME/0g-storage-node
git stash
git fetch --all --tags
git checkout v0.4.5
git submodule update --init
cargo build --release
sudo mv "$HOME/0g-storage-node/target/release/zgs_node" /usr/local/bin
```

## 3. Setup Environment Variables
Store your miner key
```bash
read -p "Enter your private key for miner_key configuration: " PRIVATE_KEY && echo
```

Store your validator json-rpc. You can use Node3's RPC `http://18.166.164.232:8545/` to sync and change to yours after full sync.
```bash
read -p "Enter your validator json-rpc for blockchain_rpc_endpoint configuration: " JSONRPC && echo
```

Store your ENR Address
```bash
ENR_ADDRESS=$(wget -qO- eth0.me)
echo "export ENR_ADDRESS=${ENR_ADDRESS}"
```

```
cat <<EOF >> ~/.bash_profile
export ENR_ADDRESS=${ENR_ADDRESS}
export ZGS_CONFIG_FILE="$HOME/0g-storage-node/run/config.toml"
export ZGS_LOG_DIR="$HOME/0g-storage-node/run/log"
export ZGS_LOG_CONFIG_FILE="$HOME/0g-storage-node/run/log_config"
EOF

source ~/.bash_profile
```

## 4. Create config.toml file
```bash
cp $HOME/0g-storage-node/run/config-testnet-turbo.toml $HOME/0g-storage-node/run/config.toml
```

## 5. Update config.toml file
```bash
sed -i 's|^\s*#\?\s*network_dir\s*=.*|network_dir = "/root/0g-storage-node/network"|' "$ZGS_CONFIG_FILE"

sed -i "s|^\s*#\?\s*network_enr_address\s*=.*|network_enr_address = \"$ENR_ADDRESS\"|" "$ZGS_CONFIG_FILE"

sed -i "s|^\s*#\?\s*blockchain_rpc_endpoint\s*=.*|blockchain_rpc_endpoint = \"$JSONRPC\"|" "$ZGS_CONFIG_FILE"

sed -i 's|^\s*#\?\s*db_dir\s*=.*|db_dir = "/root/0g-storage-node/db"|' "$ZGS_CONFIG_FILE"

sed -i 's|^\s*#\?\s*log_config_file\s*=.*|log_config_file = "/root/0g-storage-node/run/log_config"|' "$ZGS_CONFIG_FILE"

sed -i 's|^\s*#\?\s*log_directory\s*=.*|log_directory = "/root/0g-storage-node/run/log"|' "$ZGS_CONFIG_FILE"

sed -i "s|^\s*#\?\s*miner_key\s*=.*|miner_key = \"$PRIVATE_KEY\"|" "$ZGS_CONFIG_FILE"
```

## 6. Start the Node
```bash
systemctl start zgs
```

## 7. Check Log
```
tail -n 100 "$ZGS_LOG_DIR/$(ls -Art $ZGS_LOG_DIR | tail -n 1)"
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
--contract "0xbD2C3F0E65eDF5582141C35969d66e34629cC768" \
--key $PRIVATE_KEY \
--node http://YOURSTORAGEIP:5678 \
--file test.txt
```

### Download file
Please change `YOURSTORAGEIP:5678` with your actual storage node IP address and port and `$ROOTH_HASH` with the root hash that you get when generating a test file.
```bash
0g-storage-client download \
--node http://YOURSTORAGEIP:5678 \
--root $ROOTH_HASH \
--file test_downloaded.txt \
--proof
```

-----------------------------------------------------------------

<p align="center">
  &copy; 2024 BlockHub. All rights reserved.
</p>
