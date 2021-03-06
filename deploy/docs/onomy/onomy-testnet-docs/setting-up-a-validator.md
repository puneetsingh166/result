# How to become a validator on the Onomy testnet!

## What do I need?

A Linux server with any modern Linux distribution, 4cores, 8gb of ram and at least 20gb of SSD storage.

In theory Onomy chain can be run on Windows and Mac. Binaries are provided on the releases page. But instructions are not provided.

I also suggest an open notepad or other document to keep track of the keys you will be generating.

## Bootstrapping steps and commands

Start by logging into your Linux server using ssh. The following commands are intended to be run on that machine

### Download/install Onomy chain binaries
```
To download binary follow these commands
mkdir binaries
cd binaries
wget https://github.com/sunnyk56/market/raw/ONET-65/release/download/v0.0.1/onomyd
wget https://github.com/sunnyk56/market/raw/ONET-65/release/download/v0.0.1/gbt
wget https://github.com/sunnyk56/market/raw/ONET-65/release/download/v0.0.1/geth
cd ..
chmod -R +x binaries
export PATH=$PATH:$HOME/binaries/


or If you have Fedora (Fedora 34) or Redhat (Red Hat Enterprise Linux 8.4 (Ootpa))
 and you want to make binaries yourself, then follow these steps

sudo yum install -y git
git clone -b ONET-65 https://github.com/sunnyk56/market.git
cd market/deploy/onomy-chain
bash bin.sh
```

### Initiate chain

```
git clone -b ONET-65 https://github.com/sunnyk56/market.git
cd market/deploy/onomy-chain
```

### Run the first time bootstrapping playbook and script

This script will print a lot of instructions, follow them carefully and be sure to copy
down the keys you generate. You will need them later.

```
bash peer-validator/init.sh

Note: 1. Script will ask for enter validator name(Type any name for example validator1)
      2. Script will ask for enter node-id of any validator that is running in chain to add seed(Please enter 1302d0ed290d74d6f061fb8506e0e34f3f67f7ff)
      3. Script will ask for enter ip of validator for which you have added node-id(Please enter 147.182.128.38)
```
Now it's finally up and start the sycing block

### Check the status of the Onomy chain

You should be good to go! You can check the status of the
Onomy chain by running.
```
curl http://localhost:26657/status
```
if catching_up is false means your node is fully synced

### Run script to make it as validator node but before that node should fully sync

```
bash peer-validator/makeValidator.sh

Note: 1. Script will ask for enter faucet url get faucet token(Please enter http://147.182.128.38:8000/)
```
You can check the validators of the
Onomy chain by running.
```
curl http//localhost:26657/validators
```
validator count will be increased by 1 count

### Setup Gravity bridge

You are now validating on the Onomy blockchain. But as a validator you also need to run the Gravity bridge components or you will be slashed and removed from the validator set after about 16 hours.

### Register your delegate keys

Delegate keys allow the for the validator private keys to be kept in secure storage while the Orchestrator can use it's own delegated keys for Gravity functions. The delegate keys registration tool will generate Ethereum and Cosmos keys for you if you don't provide any. These will be saved in your local config for later use.

\*\*If you have set a minimum fee value in your `~/onomy/onomy/config/app.toml` modify the `--fees` parameter to match that value!

```

gbt -a onomy init

gbt -a onomy keys register-orchestrator-address --validator-phrase "the phrase you saved earlier" --fees=1nom

```

### Fund your delegate keys

Both your Ethereum delegate key and your Cosmos delegate key will need some tokens to pay gas. On the Onomy chain side you where sent some 'footoken' along with your nom. We're essentially using nom as a gas token for this testnet.

In a production network only relayers would need Ethereum to fund relaying, but for this testnet all validators run relayers by default, allowing us to more easily simulate a lively economy of many relayers.

You should have received 1000000000 Onomy Governance Token in nom and the same amount of footoken. We're going to send half of the footoken to the delegate address

To get the address for your validator key you can run the below, where 'myvalidatorkeyname' is whatever you named your key in the 'generate your key' step.

```

onomyd --home $HOME/onomy/onomy keys show myvalidatorkeyname --keyring-backend test

```

```

onomyd --home $HOME/onomy/onomy tx bank send <your validator address> <your delegate cosmos address> 1000000nom --chain-id=onomy --keyring-backend test
or using facuet
curl -X POST http://147.182.128.38:8000/ -H  "accept: application/json" -H  "Content-Type: application/json" -d "{  \"address\": \"<your delegate cosmos address>\",  \"coins\": [    \"100000000nom\"  ]}"

```

With the Onomy side faucet funded, now we need some Rinkeby Eth in the Ethereum delegate key

```
https://www.rinkeby.io/#faucet
```

### Setup Geth on the Rinkeby testnet

#### Light client instructions

```
geth --rinkeby --syncmode "light"  --rpc --rpcport "8545"
```

#### Fullnode instructions

```
geth --rinkeby --syncmode "full"  --rpc --rpcport "8545"
```

You'll see this url, please note your ip and share both this node url and your ip in chat to add to the light client nodes list

```
INFO [06-10|14:11:03.104] Started P2P networking self=enode://71b8bb569dad23b16822a249582501aef5ed51adf384f424a060aec4151b7b5c4d8a1503c7f3113ef69e24e1944640fc2b422764cf25dbf9db91f34e94bf4571@127.0.0.1:30303
```

Finally you'll need to wait for several hours until your node is synced, you can not continue with the instructions until your node is synced.

### Deployment of the Gravity contract

Once 66% of the validator set has registered their delegate Ethereum key it is possible to deploy the Gravity Ethereum contract. Once deployed the Gravity contract address on G??rli will be posted here

Here is the contract address! Move forward!

```

0xD6F7ab117DF5172264221FA33299Dd6C290cE26f

```

### Start your Orchestrator

Now that the setup is complete you can start your Orchestrator. Use the Cosmos mnemonic generated in the 'register delegate keys' step and the Ethereum private key also generated in that step. You should setup your Orchestrator in systemd or elsewhere to keep it running and restart it when it crashes.

If your Orchestrator goes down for more than 16 hours during the testnet you will be slashed and booted from the active validator set.

Since you'll be running this a lot I suggest putting the command into a script, like so. The next version of the orchestrator will use a config file for these values and have encrypted key storage.

\*\*If you have set a minimum fee value in your `$HOME/onomy/onomy/config/app.toml` modify the `--fees` parameter to match that value!

```
nano start-orchestrator.sh
```

```
#!/bin/bash
gbt --address-prefix="onomy" orchestrator \
        --cosmos-phrase="<registered delegate consmos phrase>" \
        --cosmos-grpc="http://0.0.0.0:9090" \
        --ethereum-key="<registered delegate ethereum private key>" \
        --ethereum-rpc="http://0.0.0.0:8545" \
        --fees="1nom" \
        --gravity-contract-address="0xD6F7ab117DF5172264221FA33299Dd6C290cE26f"
```

```
bash start-orchestrator.sh
```