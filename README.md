# Testnet Setup Instructions

### Install dependencies

UPDATE SYSTEM AND INSTALL BUILD TOOLS
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget build-essential jq make lz4 gcc unzip -y
```

### Install GO
```
sudo rm -rf /usr/local/go && tar -C /usr/local -xzf go1.22.3.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin
```

### Install binary
```
cd $HOME
rm -rf wardenprotocol
git clone --depth 1 --branch v0.3.0 https://github.com/warden-protocol/wardenprotocol/
cd wardenprotocol
make install
```

### Init Moniker
```
wardend init "Your-moniker-name" --chain-id buenavista-1
```

### Set up configuration
Prepare the genesis file
```
cd $HOME/.warden/config
rm genesis.json
wget https://raw.githubusercontent.com/warden-protocol/networks/main/testnets/buenavista/genesis.json
```
