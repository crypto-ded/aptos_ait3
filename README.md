# Aptos AIT3 Registration
To participate in the AIT-3 program, follow the below steps. Use these steps as a checklist to keep track of your progress. Click on the links in each step for a detailed documentation.
# Sign-in, connect Wallet and complete survey
1. Navigate to the Aptos Community page and follow the steps, starting with registering or signing in to your Discord account.

2. Before you click on Step 2 CONNECT WALLET:

- Delete any previous versions of Aptos Wallet you have installed on Chrome
- Install the Petra (Aptos Wallet) extension using Step 3 instructions, and
- Create the first wallet using Step 4 instructions.
3. Install the Petra (Aptos Wallet) extension on your Chrome browser by following the instructions:
- Download the latest Petra Wallet release and unzip.
- Open a Chrome window and navigate to the Extensions using any of the below methods:
  -  At the top right corner of the browser window, click the three vertical dots and then More tools and then Extensions, or
  -  On a new tab or a window type chrome://extensions in the URL field and press return.
- Enable Developer mode at the top right of the Extensions page.
- Click on Load unpacked at the top left, and point it to the folder where you just unzipped the downloaded Wallet release.

Now you will see Wallet in your Chrome extensions.

4. Create the first wallet using Petra (Aptos Wallet). This first wallet will always be the owner wallet
- Open the Aptos Wallet extension from the Extensions section of the Chrome browser, or by clicking on the puzzle piece on top right of the browser and selecting Aptos   Wallet.
- Click Create a new wallet.
- Make sure to store your seed phrase somewhere safe. This account will be used in the future.
5. Navigate to AIT3 Registration Page and click on Step 2 'CONNECT WALLET' to register the owner wallet address to your Aptos Community account. The Aptos team will airdrop coins to this owner wallet address.

6. Click on the Step 3 COMPLETE SURVEY to complete the survey.

7. Next, proceed to install and deploy the validator node.

# Deploy the validator node and register the node
# Hardware requirements:
For running an Aptos node on incentivized testnet we recommend the following:
- CPU: 8 cores (Intel Xeon Skylake or newer)
- Memory: 32GB RAM
- Storage: 300GB 
# Set up your aptos validator
Follow the steps below

# 1. Setting up vars
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

# 2. Update packages
```
sudo apt update && sudo apt upgrade -y
```

# 3. Install dependencies
```
sudo apt-get install jq unzip -y
```

# 4. Install docker
```
sudo apt-get install ca-certificates curl gnupg lsb-release -y
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io -y
```

# 5. Install docker compose
```
docker_compose_version=$(wget -qO- https://api.github.com/repos/docker/compose/releases/latest | jq -r ".tag_name")
sudo wget -O /usr/bin/docker-compose "https://github.com/docker/compose/releases/download/${docker_compose_version}/docker-compose-`uname -s`-`uname -m`"
sudo chmod +x /usr/bin/docker-compose
```

# 6. Download Aptos CLI
```
wget -qO aptos-cli.zip https://github.com/aptos-labs/aptos-core/releases/download/aptos-cli-v0.3.1/aptos-cli-0.3.1-Ubuntu-x86_64.zip
sudo unzip -o aptos-cli.zip -d /usr/local/bin
chmod +x /usr/local/bin/aptos
rm aptos-cli.zip
```

# 7. Install Validator node

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
**IMPORTANT**: Backup your key files somewhere safe. Never share those keys with anyone else. These key files are important for you to establish ownership of your node, 
and you will use this information to claim your rewards later if eligible. 

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
> The --full-node-host flag is optional. Only use it if you plan or already have installed fullnode on a seperate server

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
> Please make sure you use the same root public key as shown in the example and same chain ID, those config will be used during registration to verify your node.

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
# Post installation
When installation is finished please load variables into system
```
source $HOME/.bash_profile
```
### Check your node health
- Navigate to https://node.aptos.zvalid.com/
- Enter your node public IP address
- You should see data like in example below:
### Register your validator node
1.Come back to the Aptos Community page and register your node by clicking on Step 4: 'NODE REGISTRATION' button.

2.Provide the details of your validator node on this node registration screen, all the public key information you need is in the '~/$WORKSPACE/keys/public-keys.yaml' file (please don't enter anything from private keys).
```
cat ~/$WORKSPACE/keys/public-keys.yaml
```
- OWNER KEY: the first wallet public key. From Settings -> Credentials
- CONSENSUS KEY: consensus_public_key from public-keys.yaml
- CONSENSUS POP: consensus_proof_of_possession from public-keys.yaml
- ACCOUNT KEY: account_public_key from public-keys.yaml
- VALIDATOR NETWORK KEY: validator_network_public_key from public-keys.yaml
4. Next, click on VALIDATE NODE. If your node passes healthcheck, you will be prompted to complete the identity verification process.
- The Aptos team will perform a node health check on your validator, using the Node Health Checker. When Aptos confirms that your node is healthy, you will be asked to complete the KYC process.
5. Wait for the selection announcement. If you are selected, the Aptos team will airdrop coins into your owner wallet address. If you do not see airdropped coins in your owner wallet, you were not selected.
# Useful commands
### Check validator node logs
```
docker logs -f testnet-validator-1 --tail 50
```
### Check sync status
```
curl 127.0.0.1:9101/metrics 2> /dev/null | grep aptos_state_sync_version | grep type
```
### Restart docker container
```
docker restart testnet-validator-1
```
### Clean up preveous installation
WARNING! Before this step make sure you have backed up your Aptos keys as this step will completely remove your Aptos working directory
```
cd ~/$WORKSPACE && docker-compose down; cd
rm ~/$WORKSPACE -rf
docker volume rm aptos-validator
unset NODENAME
```

