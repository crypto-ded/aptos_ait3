# Aptos AIT3 Guide

### Steps in AIT3 [here](https://aptos.dev/nodes/ait/steps-in-ait3)
### Fullnode guide [here](https://github.com/mmyevyn/aptos-ait3/blob/main/FULLNODE-SETUP.md)

## Deploy the validator node
### Hardware requirements:
For running an Aptos node on incentivized testnet i recommend the following:
- CPU: 8 cores, 16 threads. 2.8GHz, or faster (Intel Xeon Skylake or newer)
- Memory: 32GB RAM
- Storage: 300GB SSD

## Follow the steps below

### 1. Setting up vars
Type your node name here
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
. <(wget -qO- https://raw.githubusercontent.com/letsnode/Utils/main/installers/docker.sh)
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

### Create directory
```
mkdir ~/$WORKSPACE && cd ~/$WORKSPACE
```

### Download config files
```
wget -qO docker-compose.yaml https://raw.githubusercontent.com/aptos-labs/aptos-core/main/docker/compose/aptos-node/docker-compose.yaml
wget -qO validator.yaml https://raw.githubusercontent.com/aptos-labs/aptos-core/main/docker/compose/aptos-node/validator.yaml
```

### Generate keys
This will create four key files under `~/$WORKSPACE/keys` directory: `public-keys.yaml`, `private-keys.yaml`, `validator-identity.yaml` and `validator-full-node-identity.yaml`
```
aptos genesis generate-keys --output-dir ~/$WORKSPACE/keys
```
> **Backup your key files in a safe place and never share those to anyone else.** 

### Configure validator
This will create two YAML files in the `~/$WORKSPACE/$USERNAME` directory: `owner.yaml` and `operator.yaml`
```
aptos genesis set-validator-configuration \
    --local-repository-dir ~/$WORKSPACE \
    --username $NODENAME \
    --owner-public-identity-file ~/$WORKSPACE/keys/public-keys.yaml \
    --validator-host $PUBLIC_IP:6180 \
    --stake-amount 100000000000000
```
> **The --full-node-host flag is optional. Use it only if you plan to or have already installed fullnode.**

### Create layout file
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
> **Please make sure you use the same root public key as shown in the example and same chain ID, those config will be used during registration to verify your node.**

### Download Aptos Framework
```
wget -qO framework.mrb https://github.com/aptos-labs/aptos-core/releases/download/aptos-framework-v0.3.0/framework.mrb -P ~/$WORKSPACE
```

### Compile genesis blob and waypoint
This will create two files in your working directory: `genesis.blob` and `waypoint.txt`
```
aptos genesis generate-genesis --local-repository-dir ~/$WORKSPACE --output-dir ~/$WORKSPACE
```

### Run docker compose
```
docker-compose up -d
```
## Post installation
When installation is finished please load variables into system
```
source $HOME/.bash_profile
```
### Public key

All the public key information you need is in the '~/$WORKSPACE/keys/public-keys.yaml' file.
```
cat ~/$WORKSPACE/keys/public-keys.yaml
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
