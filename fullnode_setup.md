# Deploy the fullnode
Recommended to install fullnode on a separate server
## For running an fullnode i recommend the following:

- CPU: 4 cores (Intel Xeon Skylake or newer)
- Memory: 8GB RAM

### 1. Update packages
```
sudo apt update && sudo apt upgrade -y
sudo apt install build-essential pkg-config openssl libssl-dev libclang-dev -y
```

### 2. Install docker
```
. <(wget -qO- https://raw.githubusercontent.com/letsnode/Utils/main/installers/docker.sh)
```

### 3. Install docker compose
```
docker_compose_version=$(wget -qO- https://api.github.com/repos/docker/compose/releases/latest | jq -r ".tag_name")
sudo wget -O /usr/bin/docker-compose "https://github.com/docker/compose/releases/download/${docker_compose_version}/docker-compose-`uname -s`-`uname -m`"
sudo chmod +x /usr/bin/docker-compose
```

### 4. Install fullnode node
#### Create directory
```
mkdir ~/testnet && cd ~/testnet
```


### Download config files
```
wget -qO docker-compose.yaml https://raw.githubusercontent.com/aptos-labs/aptos-core/main/docker/compose/aptos-node/docker-compose-fullnode.yaml
wget -qO fullnode.yaml https://raw.githubusercontent.com/aptos-labs/aptos-core/main/docker/compose/aptos-node/fullnode.yaml
```

### Edit `fullnode.yaml` file to update the IP address for Validator node
Open `fullnode.yaml` file in nano editor
```
nano fullnode.yaml
```

Change `<Validator IP Address>` to your validator IP
![image](https://user-images.githubusercontent.com/95987354/185751546-2dea9366-b901-4206-8f42-846a975fc653.png)
Press `Ctrl + X` then press `Y` and `Enter` to save changes to file

### Copy the `validator-full-node-identity.yaml`, `genesis.blob` and `waypoint.txt` files from validator node into the same working directory on Fullnode machine


### Run docker compose
```
docker-compose up -d
```

## 5. Connect to your validator node and update your validator config
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
### Check fullnode node logs
```
docker logs -f testnet-fullnode-1 --tail 50
```

### Check sync status
```
curl 127.0.0.1:9101/metrics 2> /dev/null | grep aptos_state_sync_version | grep type
```

### Restart docker container
```
docker restart testnet-fullnode-1 
```

