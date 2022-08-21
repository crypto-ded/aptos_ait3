<p style="font-size:20px" align="left">
<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/e/ef/Telegram_X_2019_Logo.svg/1024px-Telegram_X_2019_Logo.svg.png" width="35"/></a>
<a href="https://t.me/dedlutaet" target="_blank">Join to my telegram</a>

# Aptos AIT3 Guide

### Steps in AIT3 [here](https://aptos.dev/nodes/ait/steps-in-ait3)
### Fullnode guide [here](https://github.com/mmyevyn/aptos_ait3/blob/main/fullnode_setup.md)

## Deploy the validator node
### Hardware requirements:
For running an Aptos node on incentivized testnet i recommend the following:
- CPU: 8 cores, 16 threads
- Memory: 32GB RAM
- Storage: 300GB SSD

## Follow the steps below

### 1. Update packages
```
sudo apt update && sudo apt upgrade -y
```

### 2. Install dependencies
```
sudo apt-get install jq unzip -y
```

### 3. Install docker
```
sudo apt-get install ca-certificates curl gnupg lsb-release wget -y
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io -y
```

### 4. Install docker compose
```
mkdir -p ~/.docker/cli-plugins/
curl -SL https://github.com/docker/compose/releases/download/v2.6.1/docker-compose-linux-x86_64 -o ~/.docker/cli-plugins/docker-compose
chmod +x ~/.docker/cli-plugins/docker-compose
sudo chown $USER /var/run/docker.sock
```

### 5. Download Aptos CLI
```
wget -qO aptos-cli.zip https://github.com/aptos-labs/aptos-core/releases/download/aptos-cli-v0.3.1/aptos-cli-0.3.1-Ubuntu-x86_64.zip
unzip aptos-cli.zip -d /usr/local/bin
chmod +x /usr/local/bin/aptos
rm aptos-cli.zip
```

### 6. Create directories and change the name of your validator
```
export WORKSPACE=testnet
export USERNAME=validator_name
mkdir ~/$WORKSPACE
cd ~/$WORKSPACE
```

### 7. Install Validator node

### Create directory
```
mkdir ~/$WORKSPACE && cd ~/$WORKSPACE
```

### Download config files
```
wget https://raw.githubusercontent.com/aptos-labs/aptos-core/main/docker/compose/aptos-node/docker-compose.yaml
wget https://raw.githubusercontent.com/aptos-labs/aptos-core/main/docker/compose/aptos-node/validator.yaml
```

### Generate keys
This will create four key files under `~/$WORKSPACE/keys` directory: `public-keys.yaml`, `private-keys.yaml`, `validator-identity.yaml` and `validator-full-node-identity.yaml`
```
aptos genesis generate-keys --output-dir ~/$WORKSPACE/keys
```
> **Backup your key files in a safe place and never share those to anyone else.** 

### Open ports
```
apt install ufw -y 
ufw allow ssh 
ufw allow https 
ufw allow http 
ufw allow 6180 
ufw allow 80 
ufw allow 9101 
ufw allow 181 
ufw allow 182 
ufw allow 8080 
ufw allow 9103 
ufw enable
```

### Configure validator
This will create two YAML files in the `~/$WORKSPACE/$USERNAME` directory: `owner.yaml` and `operator.yaml`
```
cd ~/$WORKSPACE
aptos genesis set-validator-configuration \
    --local-repository-dir ~/$WORKSPACE \
    --username <$USERNAME> \
    --owner-public-identity-file ~/$WORKSPACE/keys/public-keys.yaml \
    --validator-host IP:6180 \
    --full-node-host IP:6182 \
    --stake-amount 100000000000000
```
> **The --full-node-host flag is optional. Use it only if you plan to or have already installed fullnode.**

### Create layout file
Layout template file defines the node in the Aptos validatorSet.
```
aptos genesis generate-layout-template --output-file ~/$WORKSPACE/layout.yaml
```

### Replace your user name with the one you set before.

root_key: "D04470F43AB6AEAA4EB616B72128881EEF77346F2075FFE68E14BA7DEBD8095E"
users: ["username you specified from previous step"]
chain_id: 43
allow_new_validators: false
epoch_duration_secs: 7200
is_test: true
min_stake: 100000000000000
min_voting_threshold: 100000000000000
max_stake: 100000000000000000
recurring_lockup_duration_secs: 86400
required_proposer_stake: 100000000000000
rewards_apy_percentage: 10
voting_duration_secs: 43200
voting_power_increase_limit: 20

> **Please make sure you use the same root public key as shown in the example and same chain ID, those config will be used during registration to verify your node.**

### Download Aptos Framework
```
wget https://github.com/aptos-labs/aptos-core/releases/download/aptos-framework-v0.3.0/framework.mrb
```

### Compile genesis blob and waypoint
This will create two files in your working directory: `genesis.blob` and `waypoint.txt`
```
aptos genesis generate-genesis --local-repository-dir ~/$WORKSPACE --output-dir ~/$WORKSPACE
```

### Run docker compose
```
docker compose up
```

### Public key

All the public key information you need is in the '~/$WORKSPACE/keys/public-keys.yaml' file.
```
cat /root/testnet/keys/public-keys.yaml
```

# Other commands
### Check validator node logs
```
docker logs -f testnet-validator-1 --tail 50
```
### Check status
```
curl 127.0.0.1:9101/metrics 2> /dev/null | grep aptos_state_sync_version | grep type
```
### Restart docker container
```
docker restart testnet-validator-1
```
