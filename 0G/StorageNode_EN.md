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
curl -o zgs_EN.sh https://gist.githubusercontent.com/botxx15/47d79d8b52bd0d156cc61f2aa58bddcd/raw/28dc5aa35f053fe1a5dbdce640d6e8a5c0158ab3/zgs_EN.sh && bash zgs_EN.sh
```

After the installation is completed, you can continue to the [Check Logs](#9-Check-Logs) step to check your logs.

-----------------------------------------------------------------

## Storage Node Installation
### 1. Install Dependencies
```bash
sudo apt-get update
sudo apt-get install git clang cmake build-essential screen
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

### 5. Create Miner ID
Create `miner_id` by running the following command. Please change `your_name` to the name you want to use.
```bash
echo -n your_name | sha256sum
```
Example output:
![CleanShot 2024-04-13 at 15 44 55@2x](https://github.com/BlockchainsHub/Testnet/assets/77204008/520bd6ff-5f62-4684-8d6e-d8f9bb9281a5)

> [!NOTE]
> Save the `miner_id` that you have created because it will be used to configure the miner.

### 6. Update The Config File
```bash
nano $HOME/0g-storage-node/run/config.toml
```

Delete the `#` sign in the `miner_id` and `miner_key` lines then fill in the `miner_id` and `miner_key` data.

> [!NOTE]
> The `miner_key` data is filled with your wallet's private key. You can view the wallet private key using Metamask.

Example configuration settings:
![CleanShot 2024-04-13 at 15 52 52@2x](https://github.com/BlockchainsHub/Testnet/assets/77204008/55272fec-d9e4-4151-a6cd-be619cc53023)

Once you have finished setting up the configuration file, you can save it by pressing `ctrl x`, `y`, `enter`.

### 7. Create a New Screen
Create a new screen to run the storage node in the background.
```bash
screen -S zgs
```

### 8. Start Storage Node
```bash
../target/release/zgs_node --config config.toml
```

After running the command above, you can immediately exit the screen by pressing the `ctrl a d` button.

### 9. Check Logs
Open the log directory.
```bash
cd $HOME/0g-storage-node/run/log
```

View the list of existing logs.
```bash
ls
```
![CleanShot 2024-04-13 at 16 34 05@2x](https://github.com/BlockchainsHub/Testnet/assets/77204008/6123290a-0ea9-4cc3-907c-3aaac9990961)

Open the existing log file. Please change the `log_name` in the command below with the log name that appears after running the `ls` command.
```bash
cat log_name
```

Example command:
```bash
cat zgs.log.2024-04-13
```

The output from the log file will appear as shown below.

Log output:
![CleanShot 2024-04-13 at 16 45 36@2x](https://github.com/BlockchainsHub/Testnet/assets/77204008/70870e65-2add-46fb-b24b-2865f168db09)
