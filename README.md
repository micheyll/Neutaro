# Why Neutaro

Neutaro is closely working with Timpi to help create the first truly decentralized search engine! On Neutaro, you can claim rewards for contributing to Timpi and vote on different proposals affecting Timpi, such as those regarding the ethical standpoint of the Timpi search engine.<br>

## Neutaro Validator Security Guide
For more details on security, please check the [Security Guide](https://github.com/Neutaro/Neutaro/blob/main/Security%20Guide.md).


## Introduction
Running a node means that you run the chains binary. Follow these steps to create a Validator that runs as a service on linux. <br>
You can be a part of Neutaro by holding tokens and delegating them, running a node, or becoming a validator.

## Delegating Tokens

By delegating your tokens to a validator, you increase the amount of staked tokens they have, thereby increasing their voting power. In return, you receive a share of their rewards based on the amount you have delegated.

To delegate your tokens to a Validator, use the command below. **1,000,000 uneutaro** is equal to **1 NTMPI**. If you want to stake 100 NTMPI tokens, you input `100000000uneutaro` into the command.

**Example Command:**

```shell
Neutaro tx staking delegate ValidatorAddress 1000000uneutaro --from YOURWALLET --chain-id Neutaro-1
```

# Before Running a Node

### **Open Port 26656 for Better Connectivity**

To improve network connectivity and allow your node to share seeds with others, you need to open port `26656` on both your Linux firewall and router.

#### **1. Open Port 26656 on Linux Firewall (UFW):**

Run the following commands to allow traffic on port `26656`:

```shell
sudo ufw allow 26656/tcp
sudo ufw reload
```

#### **2. Open Port 26656 on Your Router:**

Ensure port `26656` is open for **TCP** traffic in your router's Port Forwarding settings.

Opening port `26656` helps with better network performance and faster synchronization.


# Running a node
# **Prefer a Semi-Automated Installation?**
We suggest using **Ubuntu 22.04.03**, **4 cores, 8gb RAM** and **250-500gb of free storage**. The storage will increase overtime, but with the suggested pruning and current state of the chain it's fine and it will be fine for a few more months. <br>
<br>


If you'd like to skip the manual installation and opt for a semi-automated process, follow the steps below:

### **Semi-Automated Installation**

Run the following command in your terminal to perform a semi-automated installation:

```shell
bash <(wget -qO- https://raw.githubusercontent.com/Neutaro/Neutaro/main/neutaro_setup.sh)
```

#### **Automated Removal of the Installation**

To automatically remove the Neutaro installation, use this command:

```shell
bash <(wget -qO- https://raw.githubusercontent.com/Neutaro/Neutaro/main/neutaro_remove.sh)
```

## Validator's Best Friend: The Neutaro Help Command
Before diving into specific commands, remember that the most valuable tool for any validator is the Neutaro help command. This command provides a comprehensive list of all available commands and their descriptions, making it easier for validators to find exactly what they need. Use it regularly to stay updated:

```shell
Neutaro help
```
This will display all the commands, their descriptions, and available flags for customization.

# Step 1: Manual Setup - Update System and Install Dependencies
We suggest using **Ubuntu 22.04.03**, **4 cores, 8gb RAM** and **250-500gb of free storage**. The storage will increase overtime, but with the suggested pruning and current state of the chain it's fine and it will be fine for a few more months. <br>
<br>
Ensure your system is up-to-date and install all required dependencies: <br>
```shell
sudo apt update && sudo apt upgrade -y && sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```

### Step 2: Install Go (Golang) Version 1.22.2
Install Go, which is required for building the Neutaro binary:
<br>
```shell
GO_VERSION="1.22.2"
cd $HOME
wget "https://golang.org/dl/go$GO_VERSION.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$GO_VERSION.linux-amd64.tar.gz"
rm "go$GO_VERSION.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

Make sure version 1.22.2 is installed (if not, the make command will stop you):
<br>
Verify the installation:
<br>
```shell
go version
```

### Step 3: Clone the Neutaro Repository and Install Cosmovisor
Clone the repository and install cosmovisor, a utility to manage upgrades:
<br>
```shell
cd $HOME
git clone https://github.com/Neutaro/Neutaro
cd Neutaro
make build
go install
```

### Step 4: Set Up Cosmovisor Directories and Install Neutaro Binary
Set up cosmovisor directories and move the Neutaro binary:
<br>
```shell
cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.4.0
mkdir -p $HOME/.Neutaro/cosmovisor/genesis/bin
cp $HOME/Neutaro/build/Neutaro $HOME/.Neutaro/cosmovisor/genesis/bin/
ln -sfn $HOME/.Neutaro/cosmovisor/genesis $HOME/.Neutaro/cosmovisor/current
sudo ln -sfn $HOME/.Neutaro/cosmovisor/current/bin/Neutaro /usr/local/bin/Neutaro
```

Set up cosmovisor directories and move the Neutaro binary:
<br>
```shell
sudo apt install tree -y
tree $HOME/.Neutaro/cosmovisor/
```

Expected output
<br>
```shell
/root/.Neutaro/cosmovisor/
├── current -> /root/.Neutaro/cosmovisor/genesis
└── genesis
    └── bin
        └── Neutaro
```

### Step 5: Initialize and Configure the Neutaro Node
Initialize the node:
<br>
Replace `YourMonikerName` with your desired moniker name.
<br>
```shell
MONIKER="YourMonikerName"
Neutaro init $MONIKER --chain-id Neutaro-1
```
Configure the chain ID and keyring:
<br>

```shell
Neutaro config chain-id Neutaro-1
Neutaro config keyring-backend os
```
Download the genesis file:
<br>
```shell
curl http://154.26.153.186/genesis.json > $HOME/.Neutaro/config/genesis.json
```
### Step 6: Edit Configuration Files for Node Optimization
To optimize your node's performance, you'll need to edit configuration files using `vim`. If you're not familiar with `vim`, here’s a quick guide:

- To **enter "Insert" mode** for editing, press `i`.
- To **exit "Insert" mode**, press `ESC`.
- To **save and exit** the file, type `:x` and press `Enter`.

If you don't have `vim` installed, you can install it using the following command:

```shell
sudo apt install vim -y
```


Edit `app.toml`:
<br>
```shell
sudo vim $HOME/.Neutaro/config/app.toml
```
<br>
For pruning you could add whatever you like. These options mainly decide how much storage the Node will use. <br>


An example would be using: <br>

`minimum-gas-prices = "0uneutaro"` <br>
`pruning = "custom"` <br>
`pruning-keep-recent = "100"` <br>
`pruning-interval = "19" ` <br> 

Edit `config.toml`: <br>
```shell
sudo vim $HOME/.Neutaro/config/config.toml
```

Update the seeds line: <br>
```shell
seeds = "0e24a596dc34e7063ec2938baf05d09b374709e6@109.199.106.233:26656,84ae242b0c4c14af59a61438ba2eca4573b91c95@seed0.neutaro.tech:26656"
```

### Step 7: Download and Apply the Latest Snapshot
Using a snapshot significantly speeds up the sync process:
<br>
```shell
cd $HOME
mv $HOME/.Neutaro/data $HOME/.Neutaro/data-old
mv $HOME/.Neutaro/wasm $HOME/.Neutaro/wasm-old
wget https://snapshot.neutaro.tech/latest.tar.lz4
lz4 -d latest.tar.lz4 -c | tar xvf -
rm latest.tar.lz4
rm -r data-old wasm-old
```

### Step 8: Create and Configure a Systemd Service for Neutaro
Create a systemd service file for Neutaro: <br>
```shell
sudo vim /etc/systemd/system/Neutaro.service
```
Add the following configuration **(if you already have information set in the /Neutaro.services skip this step:**
```shell
[Unit]
Description=Neutaro Node Service
After=network-online.target

[Service]
User=root
ExecStart=/root/go/bin/cosmovisor run start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=/root/.Neutaro"
Environment="DAEMON_NAME=Neutaro"
Environment="DAEMON_DATA_BACKUP_DIR=/root/.Neutaro/data-backup"
Environment="UNSAFE_SKIP_BACKUP=true"

[Install]
WantedBy=multi-user.target
```
Enable the service but do not start it yet:<br>
```shell
sudo systemctl daemon-reload
sudo systemctl enable Neutaro
```

### Step 9: Upgrade the Neutaro Node
Since the sync status relies on the upgraded version, perform the upgrade before starting the service:
<br>
Navigate to the Neutaro Directory and Checkout the Latest Version: <br>
```shell
cd $HOME/Neutaro
git fetch --all --tags
git checkout v2.0.0
```

Rebuild the Neutaro binary:
<br>
```shell
make build
```

Prepare the Cosmovisor Upgrade Directory:
<br>
```shell
mkdir -p $HOME/.Neutaro/cosmovisor/upgrades/v2/bin
cp build/Neutaro $HOME/.Neutaro/cosmovisor/upgrades/v2/bin
```

Verify the Upgrade Setup:
<br>
Ensure the binary is in the correct location:
<br>
```shell
ls -lart $HOME/.Neutaro/cosmovisor/upgrades/v2/bin
```

Check the New Binary Version:
<br>
```shell
$HOME/.Neutaro/cosmovisor/upgrades/v2/bin/Neutaro version
```

### Step 10: Start the Neutaro Service and Monitor the Logs
Now that the upgrade is prepared, start the service:
<br>
```shell
sudo systemctl restart Neutaro
```
Monitor the logs to ensure everything is running smoothly:
<br>
```shell
sudo journalctl -fu Neutaro -o cat
```

### Step 11: Check the Number of Unique Peers
To check the number of unique peers, run the following command, which works for both root and non-root users:
<br>
```shell
jq -r '.addrs[].addr.ip' $HOME/.Neutaro/config/addrbook.json | sort | uniq | wc -l
```
### Step 12: Verify the Node Sync Status
Once the upgraded binary is running, you can check the sync status:
<br>
```shell
Neutaro status 2>&1 | jq .SyncInfo
```
<br>
use `ctrl + c` to exit the log

### **Proceed once it's synced**
you will be asked for your memonic on this step. You can also remove the --recover flag and create a new wallet and send funds to this new wallet from your main wallet. You can now delete the files using the commands from the snapshot section. <br>
<br>
```shell
Neutaro keys add WALLET --keyring-backend os --recover
```

### Step 13 Sending the becomming a validator transaction
once you have a funded wallet on the node send this, **__but make sure to check all the parameters to see if they are fine for you!__** <br>
<br>
```shell
Neutaro tx staking create-validator --amount=1000000uneutaro --pubkey=$(Neutaro tendermint show-validator) --moniker=$MONIKER --chain-id=Neutaro-1 --from WALLET --keyring-backend os --commission-rate="0.10" --commission-max-rate="0.20" --commission-max-change-rate="0.01" --min-self-delegation="1000000" --gas="auto" --gas-prices="0.0025uneutaro" --gas-adjustment="1.5" --details "About_Your_Validator"
```

## Validator's Best Friend: The Neutaro Help Command
Before diving into specific commands, remember that the most valuable tool for any validator is the Neutaro help command. This command provides a comprehensive list of all available commands and their descriptions, making it easier for validators to find exactly what they need. Use it regularly to stay updated:

```shell
Neutaro help
```
This will display all the commands, their descriptions, and available flags for customization.
