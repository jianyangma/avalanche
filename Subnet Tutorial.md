[![N|Solid](https://docs.avax.network/img/Avalanche_Horizontal_Red.svg)](https://www.avax.network/)

# Avalanche Tutorial
## Setting up development environment for local subnet development

In this tutorial, we will go through the steps for creating a local Avalanche test netowrk, and then deploy an EVM (Ethereum Virtural Machine) Subnet on to the local test network.

By the end of this tutorial, you will be able to complete the following:

- Create a local Avalanche test netowrk
- Create a custom Subnet-EVM through Gensis file
- Managing and deloying the local Subnet

## Introduction

For a quick definition of Subnets and their benefits, you can visit [It's Time: Scale with Subnets] and the [Subnet FAQ page].

Stated formally from the offical Avalanche documentation:

> A subnet, or subnetwork, is a dynamic set of validators working together to achieve consensus on the state of a set of blockchains. Each blockchain is validated by exactly one subnet. A subnet can validate many blockchains. A node may be a member of many subnets.

## Installation

The Avalanche Network Runner requires [Go] v1.17.9+ to run.
You can verify your Go installation by the following command:
```sh
go version
```

The Avalanche Network Runner will be installed at `$GOPATH/bin`, therefore we need to create the environment variable `$GOPATH` and add it to our `$PATH` by the following command:
```sh
echo 'export GOPATH=$HOME/go' >> ~/.zshenv
source ~/.zshenv
echo 'export PATH=$GOPATH/bin:$PATH' >> ~/.zshrc
source ~/.zshrc
```

Next we need to clone and build the main Avalanche Go reprository with:
```sh
git clone git@github.com:ava-labs/avalanchego.git
cd avalanchego
sudo ./scripts/build.sh
```
After the build is successful, the built binary will be located at `build/avalanchego`, let's set the `AVALANCHEGO_EXEC_PATH` variable so that it can be used by the Avalanche Network Runner
```sh
echo 'export AVALANCHEGO_EXEC_PATH=${HOME}/avalanchego/build/avalanchego' >> ~/.zshenv
source ~/.zshenv
```

Now we are ready to clone and build the Avalanche Network Runner reprository with:
```sh
cd $HOME
git clone https://github.com/ava-labs/avalanche-network-runner.git
cd avalanche-network-runner
sudo go install -v ./cmd/avalanche-network-runner
```

## Start the Local RPC Server
```sh
avalanche-network-runner server \
--log-level debug \
--port=":8080" \
--grpc-gateway-port=":8081"
```
To stop the RPC server, press `CTRL + C` on the terminal.
All local nodes will be shutdown, and ports will be unmapped.

## Start a new Avalanche Network with 5 Nodes
In another terminal session, run the following command:
```sh
avalanche-network-runner control start \
--log-level debug \
--endpoint="0.0.0.0:8080" \
--number-of-nodes=5 \
--avalanchego-path ${AVALANCHEGO_EXEC_PATH}
```

You can check the health of the server by the following command:
```sh
avalanche-network-runner control health \
--log-level debug \
--endpoint="0.0.0.0:8080"
```
At the very end of the response, you should get the text saying `health:true`
The local RPC server will also log the 5 Node IDs, as well as the URI

## Fund an address
First create a new user in the local keystore by running the following command:
> Replace the port number with one of your Node URI from the previous RPC server response. Here `15026` is the Node URI we are using

```sh
curl -X POST --data '{
    "jsonrpc":"2.0",
    "id"     :1,
    "method" :"keystore.createUser",
    "params" :{
        "username":"myUsername",
        "password":"AvalancheTutorialPass1314!"
    }
}' -H 'content-type:application/json;' 127.0.0.1:15026/ext/keystore
```
NOTE: minimum password length is 8 characters, and should contain lower case, upper case letters as well as numbers and symbols. If the password is too weak, you will get an error reponse `password is too weak`.

Then we can import the private key to the AVAX C-Chain
> The same private key, PrivateKey-ewoqjP7PxY4yr3iLTpLisriqt94hdyDFNgchSxGGztUrTXtNN, can be used to sign txs locally using AvalancheJS. You don't need to import the key into the local keystore in order to access those funds. 
```sh
curl -X POST --data '{
    "jsonrpc":"2.0",
    "id"     :1,
    "method" :"avax.importKey",
    "params" :{
        "username":"myUsername",
        "password":"AvalancheTutorialPass1314!",
        "privateKey":"PrivateKey-ewoqjP7PxY4yr3iLTpLisriqt94hdyDFNgchSxGGztUrTXtNN"
    }
}' -H 'content-type:application/json;' 127.0.0.1:15026/ext/bc/C/avax
```
Finally we check the balance of 50m (0x295be96e64066972000000 in hex)
```sh
curl -X POST --data '{
    "jsonrpc":"2.0",
    "id"     :1,
    "method" :"eth_getBalance",
    "params" : [
        "0x8db97C7cEcE249c2b98bDC0226Cc4C2A57BF52FC",
        "latest"
    ]
}' -H 'content-type:application/json;' 127.0.0.1:15026/ext/bc/C/rpc
```

## Local Subnet
Congratulations 👏 now you have a local Avalanche network running!
Now, let's go ahead and create a local subnet where we can customize our chains even further and host Solidity smart contracts.

First, let's clone the Avalanche [CLI repo]:
```sh
curl -sSfL https://raw.githubusercontent.com/ava-labs/avalanche-cli/main/scripts/install.sh | sh -s
cd bin
export PATH=$PWD:$PATH
```

Then, we need to create a Genesis file where we can define the behavior of our subnet:
```sh
touch genesis.json
```
And copy the following example parameters to the Genesis file. For an explanation of each field, please see [Official docs]:
>Note: Every EVM-based chain has a parameter called the chainId. When choosing a chainId for your network, you should choose a unique value. Check chainlist.org to see if the value you'd like is already in use.

```sh
{
  "config": {
    "chainId": 32,
    "homesteadBlock": 0,
    "eip150Block": 0,
    "eip150Hash": "0x2086799aeebeae135c246c65021c82b4e15a2c451340993aacfd2751886514f0",
    "eip155Block": 0,
    "eip158Block": 0,
    "byzantiumBlock": 0,
    "constantinopleBlock": 0,
    "petersburgBlock": 0,
    "istanbulBlock": 0,
    "muirGlacierBlock": 0,
    "subnetEVMTimestamp": 0,
    "feeConfig": {
      "gasLimit": 8000000,
      "minBaseFee": 25000000000,
      "targetGas": 15000000,
      "baseFeeChangeDenominator": 36,
      "minBlockGasCost": 0,
      "maxBlockGasCost": 1000000,
      "targetBlockRate": 2,
      "blockGasCostStep": 200000
    },
    "allowFeeRecipients": false
  },
  "alloc": {
    "8db97C7cEcE249c2b98bDC0226Cc4C2A57BF52FC": {
      "balance": "0x295BE96E64066972000000"
    }
  },
  "nonce": "0x0",
  "timestamp": "0x0",
  "extraData": "0x00",
  "gasLimit": "0x7A1200",
  "difficulty": "0x0",
  "mixHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "coinbase": "0x0000000000000000000000000000000000000000",
  "number": "0x0",
  "gasUsed": "0x0",
  "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000"
}
```

Now we are ready to create and deploy the subet to your local network:
```sh
avalanche subnet create firstsubnet --file genesis.json
avalanche subnet deploy firstsubnet
``` 

To verify the deployment, run:
```sh
avalanche subnet list
+-------------+-------------+-----------+
|   SUBNET    |    CHAIN    |   TYPE    |
+-------------+-------------+-----------+
| firstsubnet | firstsubnet | SubnetEVM |
+-------------+-------------+-----------+
```

## Managing the Local Subnet
To stop a running local network, run:
```sh
avalanche network stop
```

To start/restart a local network, run:
```sh
avalanche network start
```

To stop your local network and clear its state, run:
```sh
avalanche network clean
```

## Conclusion
By going through this tutorial, you have successfully created your very own custom Avalanche subnet! 🥂 With the Ethereum Virtural Machine running, you can now [Create your own Blockchains] and start [Minting Native Coins]

[Go]: <https://go.dev/doc/install>
[It's Time: Scale with Subnets]: <https://medium.com/avalancheavax/its-time-infinitely-scale-with-subnets-ab7cc91efa7f>
[Subnet FAQ page]: <https://support.avax.network/en/articles/6158840-subnet-faq>
[CLI repo]: <https://github.com/ava-labs/avalanche-cli>
[Official docs]: <https://docs.avax.network/subnets/customize-a-subnet>
[Create your own Blockchains]: <https://docs.avax.network/apis/avalanchego/apis/p-chain#platformcreateblockchain>
[Minting Native Coins]: <https://docs.avax.network/subnets/customize-a-subnet#minting-native-coins>
