![Github Banner](https://github.com/Kamus-Crypto/Testnet/assets/77204008/0a15a020-e7e0-4366-8030-4d4302b49428)

# Madara App Chains Guide
This document provides an introduction to joining the Avail Clash of Nodes campaign for Madara App chains. Before you decide to run your node, **make sure your Gitcoin Passport have enough score. Avail required 20 Gitcoin Passport score for request faucet**.

## Hardware recommended requirements:
It is recommended to use **ubuntu** operating system.
* 4 CPU Cores
* 16GB of Memory
* 50GB SSD

## Dependencies
There are a few dependencies that need to be installed to smoothly use `madara-cli`.

### Installing Git
```
sudo apt install git-all
```

### Installing Rust
```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

### Installing Docker
1. Install Gnome Terminal
```
sudo apt install gnome-terminal
```

2. Set up Doocker's apt repository.

Add Docker's official GPG key:
```
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

Add the repository to Apt sources:
```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
```

3. Install the Docker packages.

To install the latest version, run:
```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

4. Verify that the Docker Engine installation is successful by running the `hello-world` image.
```
sudo docker run hello-world
```

### Other Dependencies
```
sudo apt update
sudo apt upgrade
sudo apt install build-essential

sudo apt install pkg-config
sudo apt install libssl-dev
sudo apt install clang
sudo apt install protobuf-compiler
sudo apt install screen
```

## Quick Start
Clone repo:
```
git clone https://github.com/karnotxyz/madara-cli
```

Build repo:
```
cd madara-cli
cargo build --release
```

Initialize a new app chain and follow the instruction. Please fund the DA account (if applicable):
```
./target/release/madara init
```

Change your WS Provider to `wss://goldberg.avail.tools:443/ws`
```
cd $HOME
nano /root/.madara/app-chains/<your_app_chains_name>/da-config.json
```

If you forget to save your address, you can see it on `da-config.json`.
```
cat /root/.madara/app-chains/<your_app_chains_name>/da-config.json
```
Example Output:
![CleanShot 2024-02-12 at 01 27 34](https://github.com/Kamus-Crypto/Testnet/assets/77204008/60da855e-cd76-4e50-88cb-66fb15c51a4b)

Request faucet. Join the [Avail Discord](https://discord.gg/availproject) and verify yourself. Go to **#faucet-access** channel and follow the instruction on how to get the role. After you have granted the role you can request the faucet on **#goldberg-faucet** channel.
```
/deposit-rollup address <your_address>
```

Example output:
![CleanShot 2024-02-12 at 01 47 33](https://github.com/Kamus-Crypto/Testnet/assets/77204008/5186f866-72db-4f9f-90d2-29487bc57830)


Create new screen named madara:
```
Screen -S madara
```

Run your App Chain:
```
cd madara-cli
./target/release/madara run
```

Exit from your Madara screen:
```
Control + A + D
```

Create new screen named explorer:
```
Screen -S explorer
```

Run your Explorer:
```
cd madara-cli
./target/release/madara explorer
```

Exit from your Explorer screen:
```
Control + A + D
```

