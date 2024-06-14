![0G Github Banner](https://github.com/BlockchainsHub/Testnet/assets/77204008/34a32724-b411-41e4-8696-e390dfa01cab)

# 0G Storage KV Guide
This guide will help you in the 0G storage kv node installation process.

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
git clone https://github.com/0glabs/0g-storage-kv.git
cd 0g-storage-kv
git submodule update --init
cargo build --release
sudo mv "$HOME/0g-storage-kv/target/release/zgs_kv" /usr/local/bin
```

### 5. Create DB & KV-DB Directory
```bash
mkdir -p "$HOME/0g-storage-kv/db" "$HOME/0g-storage-kv/kv-db"
```

### 6. Set Up Environment Variables
Run the command below and input your IP and port in this format `http://x.x.x.x:5678`. If you installed Storage Node and Storage KV on the same server, you can use `http://127.0.0.1:5678`.
```bash
read -p "Enter your storage node IP and Port for zgs_node_urls configuration: " ZGS_NODE_URLS
```

Run the command below and input your storage node `log_sync_start_block_number`. You can run this `grep "log_sync_start_block_number" $HOME/0g-storage-node/run/config.toml | cut -d'=' -f2` command on your storage node to see the `log_sync_start_block_number`.
```bash
read -p "Enter your storage node log_sync_start_block_number configuration: " BLOCK_NUMBER
```

```bash
grep -qxF 'export DB_DIR="$HOME/0g-storage-kv/db"' ~/.bash_profile || echo 'export DB_DIR="$HOME/0g-storage-kv/db"' >> ~/.bash_profile

grep -qxF 'export ZGSKV_DB_DIR="$HOME/0g-storage-kv/kv-db"' ~/.bash_profile || echo 'export ZGSKV_DB_DIR="$HOME/0g-storage-kv/kv-db"' >> ~/.bash_profile

grep -qxF 'export ZGS_NODE_URLS="'$ZGS_NODE_URLS'"' ~/.bash_profile || echo 'export ZGS_NODE_URLS="'$ZGS_NODE_URLS'"' >> ~/.bash_profile

grep -qxF 'export ZGSKV_CONFIG_FILE="$HOME/0g-storage-kv/run/config.toml"' ~/.bash_profile || echo 'export ZGSKV_CONFIG_FILE="$HOME/0g-storage-kv/run/config.toml"' >> ~/.bash_profile

grep -qxF 'export ZGSKV_LOG_CONFIG_FILE="$HOME/0g-storage-kv/run/log_config"' ~/.bash_profile || echo 'export ZGSKV_LOG_CONFIG_FILE="$HOME/0g-storage-kv/run/log_config"' >> ~/.bash_profile

grep -qxF 'export BLOCK_NUMBER="'$BLOCK_NUMBER'"' ~/.bash_profile || echo 'export BLOCK_NUMBER="'$BLOCK_NUMBER'"' >> ~/.bash_profile

source ~/.bash_profile
```

### 7. Update Config File
If you run a validator node, you can replace `https://og-testnet-jsonrpc.blockhub.id` with your validator IP and port on the `blockchain_rpc_endpoint` section below.
```bash
sed -i "s|^\s*#\?\s*db_dir\s*=.*|db_dir = \"$DB_DIR\"|" "$ZGSKV_CONFIG_FILE"

sed -i "s|^\s*#\?\s*kv_db_dir\s*=.*|kv_db_dir = \"$ZGSKV_DB_DIR\"|" "$ZGSKV_CONFIG_FILE"

sed -i 's|^\s*#\?\s*rpc_listen_address\s*=.*|rpc_listen_address = "0.0.0.0:6789"|' "$ZGSKV_CONFIG_FILE"

sed -i "s|^\s*#\?\s*zgs_node_urls\s*=.*|zgs_node_urls = \"$ZGS_NODE_URLS\"|" "$ZGSKV_CONFIG_FILE"

sed -i "s|^\s*#\?\s*log_config_file\s*=.*|log_config_file = \"$ZGSKV_LOG_CONFIG_FILE\"|" "$ZGSKV_CONFIG_FILE"

sed -i 's|^\s*#\?\s*blockchain_rpc_endpoint\s*=.*|blockchain_rpc_endpoint = "https://og-testnet-jsonrpc.blockhub.id"|' "$ZGSKV_CONFIG_FILE"

sed -i 's|^\s*#\?\s*log_contract_address\s*=.*|log_contract_address = "0xb8F03061969da6Ad38f0a4a9f8a86bE71dA3c8E7"|' "$ZGSKV_CONFIG_FILE"

ed -i "s|^\s*#\?\s*log_sync_start_block_number\s*=.*|log_sync_start_block_number = $BLOCK_NUMBER|" "$ZGSKV_CONFIG_FILE"
```

### 8. Create Service File
Create a service file to run the storage kv node in the background.
```bash
sudo tee /etc/systemd/system/zgskv.service > /dev/null <<EOF
[Unit]
Description=0G Storage KV Node
After=network.target

[Service]
User=$USER
Type=simple
ExecStart=/usr/local/bin/zgs_kv --config $HOME/0g-storage-kv/run/config.toml
Restart=on-failure
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

### 9. Start Storage Node
```bash
sudo systemctl daemon-reload && \
sudo systemctl enable zgskv && \
sudo systemctl start zgskv && \
sudo systemctl status zgskv
```

-----------------------------------------------------------------

## Useful Commands
### Check Log
```bash
sudo journalctl -u zgskv -f -o cat
```

### Restart the Node
```bash
sudo systemctl restart zgskv
```

### Stop the Node
```bash
sudo systemctl stop zgskv
```

### Delete the Node
```bash
sudo systemctl stop zgskv
sudo systemctl disable zgskv
sudo rm /etc/systemd/system/zgskv.service
sudo rm /usr/local/bin/zgs_kv
rm -rf $HOME/0g-storage-kv
```

-----------------------------------------------------------------

<p align="center">
  &copy; 2024 BlockHub. All rights reserved.
</p>