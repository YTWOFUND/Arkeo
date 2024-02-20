# Arkeo

# Arkeo
Arkeo Node Installation Instructions </br>
### [Official documentation](https://docs.arkeo.network)

### System requirements: </br>
CPU: 4 Core </br>
RAM: 8 Gb </br>
SSD: 200 Gb </br>
OS: Ubuntu 20.04 LTS </br>

### Network configurations: </br>
Outbound - allow all traffic </br>
Inbound - open the following ports :</br>
1317 - REST </br>
26657 - TENDERMINT_RP </br>
26656 - Cosmos </br>
Running as a Provider? Add these specific ports: </br>
22221 - provider port </br>
22231 - provider port </br>
22241 - provider port </br>
    
# Manual configuration of the Lava node with Cosmovisor
Required packages installation </br>
```
sudo apt update
sudo apt upgrade -y
sudo apt install -y curl git jq lz4 build-essential
sudo apt install -y unzip logrotate git jq sed wget curl coreutils systemd
```

Go installation.
```
cd $HOME
sudo rm -rf /usr/local/go
curl -L https://go.dev/dl/go1.21.6.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile
source .bash_profile
go version
```

### Download and build binaries
```
cd $HOME
curl -s https://snapshots-testnet.nodejumper.io/arkeonetwork-testnet/arkeod > arkeod
chmod +x arkeod
mkdir -p $HOME/go/bin/
mv arkeod $HOME/go/bin/
```

# Set node CLI configuration
```
arkeod config chain-id arkeo
arkeod config keyring-backend test
arkeod config node tcp://localhost:26657
```

# Initialize the node
```
arkeod init "Your Node Name" --chain-id arkeo
```

# Download genesis and addrbook files
```
curl -L https://snapshots-testnet.nodejumper.io/arkeonetwork-testnet/genesis.json > $HOME/.arkeo/config/genesis.json
curl -L https://snapshots-testnet.nodejumper.io/arkeonetwork-testnet/addrbook.json > $HOME/.arkeo/config/addrbook.json
```

# Set seeds
```
sed -i -e 's|^seeds *=.*|seeds = "20e1000e88125698264454a884812746c2eb4807@seeds.lavenderfive.com:22856"|' $HOME/.arkeo/config/config.toml
```

# Set minimum gas price
```
sed -i -e 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.01uarkeo"|' $HOME/.arkeo/config/app.toml
```

# Set pruning
```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "17"|' \
  $HOME/.arkeo/config/app.toml
```

# Download latest chain data snapshot
```
curl "https://snapshots-testnet.nodejumper.io/arkeonetwork-testnet/arkeonetwork-testnet_latest.tar.lz4" | lz4 -dc - | tar -xf - -C "$HOME/.arkeo"
```

# Create a service
```
sudo tee /etc/systemd/system/arkeod.service > /dev/null << EOF
[Unit]
Description=Arkeo Network node service
After=network-online.target
[Service]
User=$USER
ExecStart=$(which arkeod) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable arkeod.service

# Start the service and check the logs
sudo systemctl start arkeod.service
sudo journalctl -u arkeod.service -f --no-hostname -o cat
```

### Becoming a Validator

# Create wallet key new
```
arkeod keys add wallet
```

(OPTIONAL) RECOVER EXISTING KEY
```
arkeod keys add wallet --recover
```

### We receive tokens from the tap in the discord(https://discord.com/invite/BfEHpm6uFc)

Go to the #faucet branch and specify your arkeo wallet $request tarkeo....

### Create the Validator

Before creating a validator, enter the command and check that you have false. This means that the Node has synchronized and you can create a validator:
```
arkeod status 2>&1 | jq .SyncInfo.catching_up
```

### Check the balance before creating for the presence of tokens
```
arkeod q bank balances $(arkeod keys show wallet -a)
```

Please enter your details below in quotation marks where required

```
arkeod tx staking create-validator \
--amount=1000000uarkeo \
--pubkey=$(arkeod tendermint show-validator) \
--moniker="Moniker" \
--identity=FFB0AA51A2DF5955 \
--details="I love YTWO" \
--chain-id=arkeo \
--commission-rate=0.10 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.01 \
--min-self-delegation=1 \
--from=wallet \
--gas-prices=0.01uarkeo \
--gas-adjustment=1.5 \
--gas=auto \
-y 
```

### Update
```
There have been no updates at the moment, as soon as they come out, we will immediately add them to this section.

Current network:arkeo
Current version:v1
```

### Useful commands

Check balance
```
arkeod q bank balances $(arkeod keys show wallet -a)
```

CHECK SERVICE LOGS
```
sudo journalctl -u arkeod -f --no-hostname -o cat
```

RESTART SERVICE
```
sudo systemctl restart arkeod
```

GET VALIDATOR INFO
```
arkeod status 2>&1 | jq .ValidatorInfo
```

DELEGATE TOKENS TO YOURSELF
```
arkeod tx staking delegate $(arkeod keys show wallet --bech val -a) 1000000uarkeo --from wallet --chain-id arkeo --gas-prices 0.01uarkeo --gas-adjustment 1.5 --gas auto -y
```

REMOVE NODE
Please, before proceeding with the next step! All chain data will be lost! Make sure you have backed up your priv_validator_key.json!
```
sudo systemctl stop arkeod && sudo systemctl disable arkeod && sudo rm /etc/systemd/system/arkeod.service && sudo systemctl daemon-reload && rm -rf $HOME/.arkeo && rm -rf arkeo && sudo rm -rf $(which arkeod)
```

