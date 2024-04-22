![Eclipse Github Banner](https://github.com/BlockchainsHub/Testnet/assets/77204008/fc25892b-e8ab-4f3e-8b19-430f94420bd8)

# Eclipse Testnet Guide
This guide walks you through deploying a simple smart contract to Eclipse Testnet.

## Install Dependencies
You'll have to do a few things before you can deploy your smart contract to Eclipse Testnet.

### Install Rust
Install Rust, and its package manager Cargo.
```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

. "$HOME/.cargo/env"
```

You can check if the installation was successful by running the following commands:
```bash
rustc --version
cargo --version
```

### Install Node.js
Install NVM (Node Version Manager)
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"
```

Download and install node.js
```bash
nvm install 21
```

Verifies the right Node.js version is in the environment. It should show `v21.7.3`
```bash
node -v
```

Verifies the right NPM version is in the environment. It should show `10.5.0`
```bash
npm -v
```

### Install Solana CLI
This allows you to interact with Solana clusters (Eclipse Testnet in this case).
```bash
sh -c "$(curl -sSfL https://release.solana.com/stable/install)"
```

Set your Solana CLI to Eclipse Testnet with the following command:
```bash
solana config set --url https://testnet.dev2.eclipsenetwork.xyz
```

### Install Yarn
```bash
npm install -g yarn
```

## Deposit Sepolia ETH to Eclipse Testnet
### Getting Your Ethereum Private Key
You can get your private key from Metamask wallet by navigating to `Account details` and clicking `Show private key`.

> [!NOTE]
> Make sure you have at least 0.1 Sepolia ETH in your wallet.

### Getting Your Eclipse Public Address
Generate a new keypair:
```bash
solana-keygen new
```
This will generate a new key pair and save it to your local machine. You will need this key pair to sign transactions to deploy your smart contract to Eclipse Testnet.

> [!CAUTION]
> **MAKE SURE TO SAVE YOUR PRIVATE KEY!**

You can see your previously generated Solana address using the Solana CLI:
```bash
solana address
```

### Download Deposit.js Script
```bash
git clone https://github.com/Eclipse-Laboratories-Inc/testnet-deposit
cd testnet-deposit
```

Install the dependencies with yarn.
```bash
yarn install
```

### Make a Deposit
Make sure to change this variable on the command below:
- **[YOUR_SOLANA_ADDRESS]** : Change with your previously generated Solana address
- **[ETHEREUM_PRIVATE_KEY]** : Change with the private key that you get from your Metamask wallet.

```bash
node deposit.js [YOUR_SOLANA_ADDRESS] 0x7C9e161ebe55000a3220F972058Fb83273653a6e 1500000 100 [ETHEREUM_PRIVATE_KEY] https://rpc.sepolia.org
```

Command example:
```bash
node deposit.js yyiecymjvwYE6Wxg4ZSt4ibdfFRo3JUNhQ99AKnaRqu 0x7C9e161ebe55000a3220F972058Fb83273653a6e 1500000 100 3e1bf180e4778c7944f509b422711101186d26ac15337934f12088623755c0b7 https://rpc.sepolia.org/
```

If your transaction is success, you will get a transaction has like this.
```bash
Transaction successful: 0xb05a37f4e4b420f651fdffb0b169ba96cb8c8e201b32f3d8d0c94705d7dc6d5f
```

You can check your transaction has on [Sepolia Testnet Explorer](https://sepolia.etherscan.io/).

You can verify your Eclipse testnet account balance by running the following commands:
```bash
solana balance
```

## Deploying the Smart Contract
Clone the Solana Hello World repository and install the dependencies.
```bash
cd $HOME
git clone https://github.com/solana-labs/example-helloworld
cd example-helloworld
npm install
```

Build the smart contract:
```bash
npm run build:program-rust
```

Deploy the smart contract to Eclipse Testnet:
```bash
solana program deploy dist/program/helloworld.so
```

Run the JavaScript client and confirm whether the smart contract was deployed successfully:
```bash
npm run start
```

The output should be something like this:
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

Copy the program address `8MBmHtJvxpKdYhdw6yPpedp6X6y2U9dCpdYaZJdmwV3A` and submit to this [form](https://forms.gle/yJfFABQDPmpvgzAf7).

Congratulations! You've successfully deployed a smart contract to the Eclipse Testnet.