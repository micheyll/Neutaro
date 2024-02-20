# Why you should care about Neutaro
Neutaro is closely working with Timpi to help them create the first truly decentralized search engine! On Neutaro you can claim your rewards for contributing to Timpi and you can vote on different proposals affecting Timpi. These proposals will for example be about the ethical standpoint of the Timpi search engine.

# Introduction
You can be a part of Neutaro by having tokens and delegating those, running a node or by becoming a validator.

# Delegating
By delegating your tokens to a validator you increase the amount of staked tokens they have and by that their voting power. In return you get a cut of their rewards based on the amount you have delegated. You can delegate to a Validator using the command. 1.000.000 uneutaro is 1 NTMPI. if you want to stake 100 NTMPI tokens you put **100000000uneutaro** into the command. The example is 1 token.

Neutaro tx staking delegate ValidatorAddress 1000000uneutaro --from YOURWALLET --chain-id Neutaro-1 --node https://rpc1.neutaro.tech:443

# Running a node
Running a node means that you run the chains binary. Follow these steps to create a Validator that runs as a service on linux.

### First we update linux.
we suggest using Ubuntu 22.04.03.

sudo apt update && sudo apt upgrade -y && sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y

### Installing Go v1.20.4
ver="1.20.4" <br>
cd $HOME <br>
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" <br>
sudo rm -rf /usr/local/go <br>
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" <br>
rm "go$ver.linux-amd64.tar.gz" <br>
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile <br>
source $HOME/.bash_profile

### Check the go version

go version

### Then we install the Neutaro Binary

cd $HOME <br> 
git clone https://github.com/Neutaro/Neutaro <br>
cd Neutaro/cmd/Neutaro/ <br>
go build <br>

### installing cosmovisor

mkdir -p $HOME/.Neutaro/cosmovisor/genesis/bin <br>
mv $HOME/Neutaro/cmd/Neutaro/Neutaro $HOME/.Neutaro/cosmovisor/genesis/bin/ <br>
sudo ln -s $HOME/.Neutaro/cosmovisor/genesis $HOME/.Neutaro/cosmovisor/current <br>
sudo ln -s $HOME/.Neutaro/cosmovisor/current/bin/Neutaro /usr/local/bin/Neutaro <br>
cd  $HOME/Neutaro/ <br>
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.4.0

### Now we configure the node

MONIKER=YOURMONIKER <br>
Neutaro init $MONIKER --chain-id Neutaro-1 <br>
Neutaro config chain-id Neutaro-1 <br>
Neutaro config keyring-backend os <br>

url -Ls curl http://154.26.153.186/genesis.json > \$HOME/.Neutaro/config/genesis.json\ <br>
PEERS="$(curl -sS https://rpc1.neutaro.tech/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print \$1":"\$(NF)}' | sed -z 's|\n|,|g;s|.$||')" <br>
sed -i -e "s|^persistent_peers *=.*|persistent_peers = \"$PEERS\"|" $HOME/.Neutaro/config/config.toml <br>
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\" /" $HOME/.Neutaro/config/config.toml <br>

### Here you can adjust the minimum-gas-price. Currently most use 0
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0uneutaro\"|" $HOME/.Neutaro/config/app.toml
### Configuring pruning
You could add whatever you like. These options mainly decide how much storage the Node will use.

PRUNING="custom" <br>
PRUNING_KEEP_RECENT="100" <br>
PRUNING_INTERVAL="19" <br>
sed -i -e "s/^pruning *=.*/pruning = \"$PRUNING\"/" $HOME/.Neutaro/config/app.toml <br>
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \ <br>
\"$PRUNING_KEEP_RECENT\"/" $HOME/.Neutaro/config/app.toml <br>
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \ <br>
\"$PRUNING_INTERVAL\"/" $HOME/.Neutaro/config/app.toml


### Create Neutaro service. Copy/paste everything from sudo to EOF.

sudo tee /etc/systemd/system/Neutaro.service > /dev/null << EOF
[Unit]
Description=Neutaro Node Service
After=network-online.target

[Service]
User=$USER <br>
ExecStart=$(which cosmovisor) run start <br>
Restart=on-failure <br>
RestartSec=10 <br>
LimitNOFILE=65535 <br>
Environment="DAEMON_HOME=$HOME/.Neutaro" <br>
Environment="DAEMON_NAME=Neutaro" <br>
Environment="UNSAFE_SKIP_BACKUP=true" <br>

[Install] <br>
WantedBy=multi-user.target <br>
EOF 

### Enabling the Service
sudo systemctl daemon-reload <br>
sudo systemctl enable Neutaro

### Starting the Service/ the Node
sudo systemctl restart Neutaro

### To view the service use
sudo journalctl -fu Neutaro -o cat
<br>
<br>
use ctrl + c to exit the log

### Check the sync status
Neutaro status 2>&1 | jq .SyncInfo

### **Proceed once it's synced**
you will be asked for your memonic on this step. You can also remove the --recover flag and create a new wallet and send funds to this new wallet from your main wallet
<br>
<br>
Neutaro keys add WALLET --keyring-backend os --recover 

### Sending the becomming a validator transaction
once you have a funded wallet on the node send this, **__but make sure to check all the parameters to see if they are fine for you!__** <br>
<br>
Neutaro tx staking create-validator --amount=1500000uneutaro --pubkey=$(Neutaro tendermint show-validator) --moniker=$MONIKER --identity="79E14FB4E5BE4F30" --chain-id=Neutaro-1 --from WALLET --keyring-backend os --commission-rate="0.10" --commission-max-rate="0.20" --commission-max-change-rate="0.01" --min-self-delegation="1000000" --gas="auto" --gas-prices="0.0025uneutaro" --gas-adjustment="1.5"
