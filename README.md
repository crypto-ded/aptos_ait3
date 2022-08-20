# Aptos AIT3 Guide

## Deploy the validator node
### Hardware requirements:
For running an Aptos node on incentivized testnet we recommend the following:
- CPU: 8 cores (Intel Xeon Skylake or newer)
- Memory: 32GB RAM
- Storage: 300GB 

## Follow the steps below

### 1. Setting up vars
Put your node name here
```
NODENAME=<your_nodename>
```

Save and import variables into system
```
echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
echo "export WORKSPACE=testnet" >> $HOME/.bash_profile
echo "export PUBLIC_IP=$(curl -s ifconfig.me)" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### 2. Update packages
```
sudo apt update && sudo apt upgrade -y
```

### 3. Install dependencies
```
sudo apt-get install jq unzip -y
```

### 4. Install docker
```
sudo apt-get install ca-certificates curl gnupg lsb-release -y
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io -y
```

### 5. Install docker compose
```
docker_compose_version=$(wget -qO- https://api.github.com/repos/docker/compose/releases/latest | jq -r ".tag_name")
sudo wget -O /usr/bin/docker-compose "https://github.com/docker/compose/releases/download/${docker_compose_version}/docker-compose-`uname -s`-`uname -m`"
sudo chmod +x /usr/bin/docker-compose
```

### 6. Download Aptos CLI
```
wget -qO aptos-cli.zip https://github.com/aptos-labs/aptos-core/releases/download/aptos-cli-v0.3.1/aptos-cli-0.3.1-Ubuntu-x86_64.zip
sudo unzip -o aptos-cli.zip -d /usr/local/bin
chmod +x /usr/local/bin/aptos
rm aptos-cli.zip
```

### 7. Install Validator node

#### Create directory
```
mkdir ~/$WORKSPACE && cd ~/$WORKSPACE
```

#### Download config files
```
wget -qO docker-compose.yaml https://raw.githubusercontent.com/aptos-labs/aptos-core/main/docker/compose/aptos-node/docker-compose.yaml
wget -qO validator.yaml https://raw.githubusercontent.com/aptos-labs/aptos-core/main/docker/compose/aptos-node/validator.yaml
```

#### Generate keys
This will create four key files under `~/$WORKSPACE/keys` directory: `public-keys.yaml`, `private-keys.yaml`, `validator-identity.yaml` and `validator-full-node-identity.yaml`
```
aptos genesis generate-keys --output-dir ~/$WORKSPACE/keys
```
**IMPORTANT**: Backup your key files somewhere safe. Never share those keys with anyone else. These key files are important for you to establish ownership of your node, 
and you will use this information to claim your rewards later if eligible. 

#### Configure validator
This will create two YAML files in the `~/$WORKSPACE/$USERNAME` directory: `owner.yaml` and `operator.yaml`
```
aptos genesis set-validator-configuration \
    --local-repository-dir ~/$WORKSPACE \
    --username $NODENAME \
    --owner-public-identity-file ~/$WORKSPACE/keys/public-keys.yaml \
    --validator-host $PUBLIC_IP:6180 \
    --stake-amount 100000000000000
```
> The --full-node-host flag is optional. Only use it if you plan or already have installed fullnode on a seperate server

#### Create layout file
Layout template file defines the node in the Aptos validatorSet.
```
sudo tee layout.yaml > /dev/null <<EOF
---
root_key: "D04470F43AB6AEAA4EB616B72128881EEF77346F2075FFE68E14BA7DEBD8095E"
users:
  - $NODENAME
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
EOF
```
> Please make sure you use the same root public key as shown in the example and same chain ID, those config will be used during registration to verify your node.

#### Download Aptos Framework
```
wget -qO framework.mrb https://github.com/aptos-labs/aptos-core/releases/download/aptos-framework-v0.3.0/framework.mrb -P ~/$WORKSPACE
```

#### Compile genesis blob and waypoint
This will create two files in your working directory: `genesis.blob` and `waypoint.txt`
```
aptos genesis generate-genesis --local-repository-dir ~/$WORKSPACE --output-dir ~/$WORKSPACE
```

#### Run docker compose
```
docker-compose up -d
```
## Post installation
When installation is finished please load variables into system
```
source $HOME/.bash_profile
```
### Check your node health
- Navigate to https://node.aptos.zvalid.com/
### Register your validator node

All the public key information you need is in the '~/$WORKSPACE/keys/public-keys.yaml' file.
```
cat ~/$WORKSPACE/keys/public-keys.yaml
```

## Useful commands
#### Check validator node logs
```
docker logs -f testnet-validator-1 --tail 50
```
#### Check sync status
```
curl 127.0.0.1:9101/metrics 2> /dev/null | grep aptos_state_sync_version | grep type
```
#### Restart docker container
```
docker restart testnet-validator-1
```
#### Clean up preveous installation
WARNING! Before this step make sure you have backed up your Aptos keys as this step will completely remove your Aptos working directory
```
cd ~/$WORKSPACE && docker-compose down; cd
rm ~/$WORKSPACE -rf
docker volume rm aptos-validator
unset NODENAME
```

# Deploy the fullnode
Recommended to install fullnode on a separate server
## For running an Aptos fullnode we recommend the following:

- CPU: 4 cores (Intel Xeon Skylake or newer)
- Memory: 8GB RAM

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
sudo apt-get install ca-certificates curl gnupg lsb-release -y
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io -y
```

### 4. Install docker compose
```
docker_compose_version=$(wget -qO- https://api.github.com/repos/docker/compose/releases/latest | jq -r ".tag_name")
sudo wget -O /usr/bin/docker-compose "https://github.com/docker/compose/releases/download/${docker_compose_version}/docker-compose-`uname -s`-`uname -m`"
sudo chmod +x /usr/bin/docker-compose
```

### 5. Install fullnode node
#### Create directory
```
mkdir ~/testnet && cd ~/testnet
```

#### Download config files
```
wget -qO docker-compose.yaml https://raw.githubusercontent.com/aptos-labs/aptos-core/main/docker/compose/aptos-node/docker-compose-fullnode.yaml
wget -qO fullnode.yaml https://raw.githubusercontent.com/aptos-labs/aptos-core/main/docker/compose/aptos-node/fullnode.yaml
```

#### Edit fullnode.yaml file to update the IP address for Validator node
Open fullnode.yaml file in nano editor
```
nano fullnode.yaml
```

Change `<Validator IP Address>` to your validator IP

Press `Ctrl + X` then press `Y` and `Enter` to save changes to file

### Copy the validator-full-node-identity.yaml, genesis.blob and waypoint.txt files from validator node into the same working directory on Fullnode machine


#### Run docker compose
```
docker-compose up -d
```

## 6. Connect to your validator node and update your validator config
Change `<YOUR_FULLNODE_IP>` to you fullnode public ip
```
aptos genesis set-validator-configuration \
    --keys-dir ~/$WORKSPACE --local-repository-dir ~/$WORKSPACE \
    --username $NODENAME \
    --validator-host $PUBLIC_IP:6180 \
    --full-node-host <YOUR_FULLNODE_IP>:6182
```

Restart docker compose
```
cd ~/$WORKSPACE
docker-compose restart
```

### Useful commands
#### Check fullnode node logs
```
docker logs -f testnet-fullnode-1 --tail 50
```

#### Check sync status
```
curl 127.0.0.1:9101/metrics 2> /dev/null | grep aptos_state_sync_version | grep type
```

#### Restart docker container
```
docker restart testnet-fullnode-1 
```
