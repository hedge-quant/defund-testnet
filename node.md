# **DeFund Privet Testnet**

At the time of writing, the project has launched only Private Testnet, perhaps everyone will be admitted further.
UPD: 04/22/2022 - You can still join the Private Testnet
Fill out the [FORM](https://docs.google.com/forms/d/e/1FAIpQLSfqw1F6cDCsEt1Qmn70oIp3tdvcPFvRyk9Nd4mFacelhzQFYA/viewform) and join [DISCORD](https://discord.gg/5D4Mdetf)

# **Node Setup**

**Hardware Requirements: 4/8/200**

First, we need setup **GO**

```bash
cd $HOME

wget -O go1.17.1.linux-amd64.tar.gz https://golang.org/dl/go1.17.linux-amd64.tar.gz

rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go1.17.1.linux-amd64.tar.gz && rm go1.17.1.linux-amd64.tar.gz

echo 'export GOROOT=/usr/local/go' >> $HOME/.bash_profile

echo 'export GOPATH=$HOME/go' >> $HOME/.bash_profile

echo 'export GO111MODULE=on' >> $HOME/.bash_profile

echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile && . $HOME/.bash_profile
```


Let's check the version
```bash
go version
```

As always, we update the server and install the necessary packages
```bash
sudo apt update && sudo apt upgrade -y

sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```

Download and install node
```bash
git clone https://github.com/defund-labs/defund

cd $HOME/defund

make install
```

Edit and fill your personal _moniker-name_ and _wallet_name_
```bash
DEFUND_CHAIN="defund-private-1"

DEFUND_MONIKER="YOUR_MONIKER_NAME"

DEFUND_WALLET="YOUR_WALLET_NAME"
```

Writing all to the bash profile
```bash
echo 'export DEFUND_CHAIN='${DEFUND_CHAIN} >> $HOME/.bash_profile

echo 'export DEFUND_MONIKER='${DEFUND_MONIKER} >> $HOME/.bash_profile

echo 'export DEFUND_WALLET='${DEFUND_WALLET} >> $HOME/.bash_profile

source $HOME/.bash_profile
```

Initialise node
```bash
defundd init $DEFUND_MONIKER --chain-id=$DEFUND_CHAIN
```

Add seeds and peers
```bash
seeds="1b3e596531dd8f36363b13339beed2364900e4c6@104.131.41.157:26656" peers="111ba4e5ae97d5f294294ea6ca03c17506465ec5@208.68.39.221:26656,26c42b6c3e8940c5433a5601464c4b370ab32cb4@139.162.146.250:26656"

sed -i "s/^seeds *=.*/seeds = \"$seeds\"/;" $HOME/.defund/config/config.toml

sed -i "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/;" $HOME/.defund/config/config.toml
```

Download genesis.json file
```bash
wget -O $HOME/.defund/config/genesis.json https://raw.githubusercontent.com/schnetzlerjoe/defund/main/testnet/private/genesis.json

defundd tendermint unsafe-reset-all --home $HOME/.defund
```

Create a service
```bash
tee $HOME/defund.service > /dev/null <<EOF
[Unit]
Description=Defund
After=network.target
[Service]
Type=simple
User=$USER
ExecStart=$(which defundd) start
Restart=on-failure
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF

sudo mv $HOME/defund.service /etc/systemd/system/
```

Reload system service and start node
```bash
sudo systemctl restart systemd-journald
sudo systemctl daemon-reload
sudo systemctl enable defund
sudo systemctl restart defund
```

Check logs
```bash
journalctl -u defund -f -o cat
```

Chech status and wait until synced
```bash
curl -s localhost:26657/status
```

Once synced `"catching_up": false`

Start to create new wallet, edit and fill your personal _wallet name_ instaed _$DEFUND_WALLET_
```bash
defundd keys add $DEFUND_WALLET

DEFUND_ADDR=$(defundd keys show $DEFUND_WALLET -a)

echo 'export DEFUND_ADDR='${DEFUND_ADDR} >> $HOME/.bash_profile

source $HOME/.bash_profile
```

Check balance
```bash
defundd query bank balances $DEFUND_ADDR
```

Create validator address
```bash
defundd tx staking create-validator \
  --amount=25000000ufetf \
  --pubkey=$(defundd tendermint show-validator) \
  --moniker=$DEFUND_MONIKER \
  --chain-id=defund-private-1 \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1" \
  --from=$DEFUND_WALLET
```

Then adding validator address to variable and adding to bash profile
```bash
DEFUND_VALOPER=$(defundd  keys show $DEFUND_WALLET --bech val -a)

echo 'export DEFUND_VALOPER='${DEFUND_VALOPER} >> $HOME/.bash_profile
source $HOME/.bash_profile
```

Chech validator status
```bash
defundd  query staking validator $DEFUND_VALOPER
```

Now you can start to delegate to your validator address
```bash
defundd  tx staking delegate [VALOPER_ADDRESS] [STAKE_AMOUNT]ufetf --from [your-key-name] --chain-id defund-private-1
```

In case if you got jail, this command will help to unjail
```bash
defundd   tx slashing unjail --chain-id defund-private-1 --from [your-key-name]
```


Hope it helped you to tun your node

Nick



