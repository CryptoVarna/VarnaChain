# VarnaChain
Permissioned City Level Blockchain

## Abstract
VarnaChain will be a permissioned blockchain and infrastructure for it built in the city of Varna. The blockchain will be used for free public services, tests and experimental projects. First version will be built on Ethereum PoA. The control of the network will be done by Smart Contracts defining a DAO.  

## Infrastructure
![Infrastructure](media/infrastructure.svg)

### Ethereum Proof of Authority and Clique Protocol
https://github.com/ethereum/EIPs/issues/225

### Docker Registry
Will store and deploy the containers needed for all participants to operate. Updates will be managed also by this registry. It is a private registry as it is not needed the public to have access to that part. Though all the containers will be open sourced to be validated.

Setting up the provate registry:
https://docs.docker.com/registry/deploying/

## Starting the Blockchain
Step by step tutorial to start the permissioned blockchain

### Download Go-Ethereum
https://geth.ethereum.org/downloads/

### Create folder on the sealer device
```
mkdir data
```

### Create new account (key store)
```
$ geth --datadir data/ account new
Your new account is locked with a password. Please give a password. Do not forget this password.
Passphrase: ******
Repeat passphrase: ******
Address: {87366ef81db496edd0ea2055ca605e8686eec1e6}
```
On this step a new private/public key pair are generated and stored locally in the data folder. 
You need to keep that information private and secure.

_IMPORTANT: Copy the public address. Your node will be added to the list of allowed sealers using it_

### Copy your password 
_IMPORTANT: This is not the most secure way and will be changed in the future_
Create a file password.txt with your password in it so the node can be ran without the need to type the password manually.
```
echo "******" > password.txt
```

### Send your address to the authoritiy
At this step you need to send the public address such as {87366ef81db496edd0ea2055ca605e8686eec1e6} to the VarnaChain authority to add you as a sealer when creating the genesis block.

### Creation of the genesis block
Here the genesis will be created. It could be done in different ways.
One is to use puppeth
```
puppeth --network=varnachain
````
It is a wizard for blockchain setup. The genesis file must be defined and exported.

Other option is to edit it directly. 
[varnachain.json](https://github.com/CryptoVarna/VarnaChain/blob/master/varnachain.json)

The significant properties are:
* chainId - the id of the chain. It is good to be bigger than 10 since there are popular public chains under that.
* extraData - here all the allowed sealer's addresses are listed
* ...

### Initialize node
```
$ geth --datadir data/ init varnachain.json
```

### Initialize bootnode
Boot node is a rendezvous-like server for node discovery.
```
$ bootnode -genkey boot.key
```

### Start bootnode
The first time a node connects to the network it uses one of the predefined bootnodes. Through these bootnodes a node can join the network and find other nodes. In the case of a private cluster these predefined bootnodes are not of much use. Therefore go-ethereum offers a bootnode implementation that can be configured and run in your private network.
```
$ bootnode -nodekey boot.key -verbosity 9 -addr :10666
INFO [02-07|22:44:09] UDP listener up                          self=enode://3ec4fef2d726c2c01f16f0a0030f15dd5a81e274067af2b2157cafbf76aa79fa9c0be52c6664e80cc5b08162ede53279bd70ee10d024fe86613b0b09e1106c40@[::]:10666
```
_IMPORTANT: Copy the enode address_

### Start node
```
$ geth --datadir data/ --syncmode "full" --port 10667 --rpc --rpcaddr "localhost" --rpcport 8545 --rpcapi "personal,db,eth,net,web3,txpool,miner" --bootnodes "enode://3ec4fef2d726c2c01f16f0a0030f15dd5a81e274067af2b2157cafbf76aa79fa9c0be52c6664e80cc5b08162ede53279bd70ee10d024fe86613b0b09e1106c40@10.10.10.10:10666" --networkid 27382 --gasprice "1" -unlock "0x87366ef81db496edd0ea2055ca605e8686eec1e6" --password password.txt --mine
```
Additionally few options can be added for:
* --ipcdisable - to disable IPC for example when running more than one node on same computer
* --rpccorsdomain "*" - to allow external connection to the RPC (for all domains)

### Add exception to the firewall
Allow connections to the ports for the bootnode and the node. If required also for the RPC 
