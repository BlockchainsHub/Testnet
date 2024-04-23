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

## Storage Node Auto-Installation
```bash
curl -o zgs_EN.sh https://gist.githubusercontent.com/botxx15/47d79d8b52bd0d156cc61f2aa58bddcd/raw/4b2576c6f404f7108fae0ce0cb187edd0b75f562/zgs_EN.sh && bash zgs_EN.sh
```

After the installation is completed, you can continue to the [Check Latest Log](#Check-Latest-Log) step to check your logs.

-----------------------------------------------------------------

## Storage Node Installation
### 1. Install Dependencies
```bash
sudo apt-get update
sudo apt-get install git wget curl clang cmake build-essential
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
sudo mv $HOME/0g-storage-node/target/release/zgs_node /usr/local/bin
```

### 5. Set Up Environment Variables
```bash
echo 'export ZGS_CONFIG_FILE="$HOME/0g-storage-node/run/config.toml"' >> ~/.bash_profile
echo 'export ZGS_LOG_DIR="$HOME/0g-storage-node/run/log"' >> ~/.bash_profile
echo 'export ZGS_LOG_CONFIG_FILE="$HOME/0g-storage-node/run/log_config"' >> ~/.bash_profile
source ~/.bash_profile
```

### 6. Store Miner ID and Key
```bash
read -p "Enter your name for miner_id config: " MINER_NAME && echo
```
```bash
read -sp "Enter your private key for miner_key config: " PRIVATE_KEY && echo
```

### 7. Update Config File
```bash
if grep -q 'miner_id' "$ZGS_CONFIG_FILE" || grep -q '# miner_id' "$ZGS_CONFIG_FILE"; then
    MINER_ID=$(echo -n "$MINER_NAME" | sha256sum | cut -d ' ' -f1)
    sed -i "/#*miner_id/c\miner_id = \"$MINER_ID\"" "$ZGS_CONFIG_FILE"
fi

if grep -q 'miner_key' "$ZGS_CONFIG_FILE" || grep -q '# miner_key' "$ZGS_CONFIG_FILE"; then
    sed -i "/#*miner_key/c\miner_key = \"$PRIVATE_KEY\"" "$ZGS_CONFIG_FILE"
fi

if ! grep -q "^# log_config_file" "$ZGS_CONFIG_FILE"; then
    sed -i "/^# log_config_file/c\log_config_file = \"$ZGS_LOG_CONFIG_FILE\"" $ZGS_CONFIG_FILE
elif ! grep -q "^log_config_file" "$ZGS_CONFIG_FILE"; then
    sed -i "/^# log_config_file/c\log_config_file = \"$ZGS_LOG_CONFIG_FILE\"" $ZGS_CONFIG_FILE
fi

if ! grep -q "^# log_directory" "$ZGS_CONFIG_FILE"; then
    sed -i "/^# log_directory/c\log_directory = \"$ZGS_LOG_DIR\"" $ZGS_CONFIG_FILE
elif ! grep -q "^log_directory" "$ZGS_CONFIG_FILE"; then
    sed -i "/^# log_directory/c\log_directory = \"$ZGS_LOG_DIR\"" $ZGS_CONFIG_FILE
fi
```

### 8. Create Service File
Create a service file to run the storage node in the background.
```bash
sudo tee /etc/systemd/system/zgs.service > /dev/null <<EOF
[Unit]
Description=0G Storage Node
After=network.target

[Service]
User=$USER
Type=simple
ExecStart=zgs_node --config $ZGS_CONFIG_FILE
Restart=on-failure
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

### 9. Start Storage Node
```bash
sudo systemctl daemon-reload && \
sudo systemctl enable zgs && \
sudo systemctl restart zgs && \
sudo systemctl status zgs
```

-----------------------------------------------------------------

## Useful Commands
### Check Latest Log
```bash
tail -n 100 "$ZGS_LOG_DIR/$(ls -Art $ZGS_LOG_DIR | tail -n 1)"
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

### Upgrade the Node
Please change the `<version>` to the latest version.
```bash
cd $HOME/0g-storage-node
git fetch
git checkout tags/<version>
git submodule update --init
cargo build --release
sudo mv $HOME/0g-storage-node/target/release/zgs_node /usr/local/bin
```

### Backup Miner ID and Key
You can backup your `miner_id` and `miner_key` by saving the output value of the command below.
```bash
grep 'miner_id' $ZGS_CONFIG_FILE
grep 'miner_key' $ZGS_CONFIG_FILE
```

### Delete The Node
> [!CAUTION]
> **Make sure to backup your `miner_id` and `miner_key` before deleting the node!**
```bash
sudo systemctl stop zgs
sudo systemctl disable zgs
sudo rm /etc/systemd/system/zgs.service
rm -rf $HOME/0g-storage-node
```

-----------------------------------------------------------------

<p align="center">
  &copy; 2024 BlockHub. All rights reserved.
</p>