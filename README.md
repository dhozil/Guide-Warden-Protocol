# Guide Warden Protocol Buenavista-1

## Testnet Setup Instructions

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
Prepare the genesis file And set some mandatory configuration options
```
cd $HOME/.warden/config
rm genesis.json
wget https://raw.githubusercontent.com/warden-protocol/networks/main/testnets/buenavista/genesis.json
```

### Set minimum gas price & peers
```
sed -i 's/minimum-gas-prices = ""/minimum-gas-prices = "0.0025uward"/' app.toml
sed -i 's/persistent_peers = ""/persistent_peers = "ddb4d92ab6eba8363bab2f3a0d7fa7a970ae437f@sentry-1.buenavista.wardenprotocol.org:26656,c717995fd56dcf0056ed835e489788af4ffd8fe8@sentry-2.buenavista.wardenprotocol.org:26656,e1c61de5d437f35a715ac94b88ec62c482edc166@sentry-3.buenavista.wardenprotocol.org:26656"/' config.toml
```

### Set pruning
```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "19"|' \
  $HOME/.side/config/app.toml
```

### Create service file
```
sudo tee /etc/systemd/system/wardend.service > /dev/null <<EOF
[Unit]
Description=Warden node
After=network-online.target
[Service]
User=$USER
WorkingDirectory=$HOME/.warden
ExecStart=$(which wardend) start --home $HOME/.warden
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```

### Enable and start service
```
sudo systemctl daemon-reload
sudo systemctl enable wardend
sudo systemctl restart wardend && sudo journalctl -u wardend -f -o cat
```

## Key management

### Add new wallet
```
wardend keys add wallet
```

### Restore wallet
```
wardend keys add wallet --recover
```

### Check wallet balance 
```
wardend q bank balances $(wardend keys show wallet -a)
```

## Creating a validator
To create a validator and initialize it with a self-delegation, you need to create a validator.json file and submit a create-validator transaction

### Get pubkey
```
wardend comet show-validator
```

### Create file validator.json, then edit with your pubkey and your identity
```
cd $HOME/.warden/config
nano validator.json
```

### Edit this with your identity and paste to validator.json then save it
```
{    
    "pubkey": {"@type":"/cosmos.crypto.ed25519.PubKey","key":"lR1d7YBVK5jYijOfWVKRFoWCsS4dg3kagT7LB9GnG8I="},
    "amount": "1000000uward",
    "moniker": "your-node-moniker",
    "identity": "eqlab testnet validator",
    "website": "optional website for your validator",
    "security": "optional security contact for your validator",
    "details": "optional details for your validator",
    "commission-rate": "0.1",
    "commission-max-rate": "0.2",
    "commission-max-change-rate": "0.01",
    "min-self-delegation": "1"
}
```

### Create validator
```
wardend tx staking create-validator $HOME/.warden/config/validator.json \
  --chain-id buenavista-1 \
  --from wallet \
  --fees 650uward
```

## Additional commands

### Validator info
```
wardend q staking validator $(wardend keys show $WALLET --bech val -a)
```

### Node info
```
wardend status 2>&1 | jq
```

### Delegate to your validator
```
wardend tx staking delegate $(wardend keys show $WALLET --bech val -a) 1000000uward --from $WALLET --chain-id buenavista-1 --gas auto --gas-adjustment 1.5 --fees 650uward -y
```

### Unjail Validator 
```
wardend tx slashing unjail --from $WALLET --chain-id buenavista-1 --gas auto --gas-adjustment 1.5 --fees 650uward -y
```

### Stop Warden
```
sudo systemctl stop wardend
```

### Restart warden
```
sudo systemctl restart wardend
```

### Check Log
```
sudo journalctl -u wardend -f
```

### View Proposal
```
wardend query gov proposal 1
```

### Vote Proposal
```
wardend tx gov vote 1 yes --from wallet --chain-id buenavista-1  --gas auto --gas-adjustment 1.5 --fees 650uward -y
```
