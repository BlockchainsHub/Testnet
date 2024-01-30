![Github Banner Masa](https://github.com/Kamus-Crypto/Testnet/assets/77204008/665af79e-bf5b-4bfa-944e-ead70dbce5fa)

# Masa Oracle Node Guide

## Hardware recommended requirements:
It is recommended to use **ubuntu** operating system.
* 4 CPU Cores
* 1GB of Memory
* 20GB SSD

## Getting Started

### Prerequisites

#### Install Go
You can skip this if you already installed Go. Run command ```go version``` on your terminal to verify it.
```
ver="1.21.6"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```

#### Install Docker
