# Run Bots - Dutch X
This is just a simple project that shows how to run the Dutch X bots to ensure 
liquidity for any ERC20 token pair.

To make it easier, we provide a `Docker` image with all the **bots** and a 
**CLI**.

Follow through this document to run your own bots and learn how to operate on
the Dutch Exchange.

# Run the bots
**Create the running script**:
Create a script `run-bots.sh` with the following content:

```bash
#!/usr/bin/env bash

docker run \
  -p 8081:8081 \
  -e MNEMONIC="super secret thing that nobody should know" \
  -e NODE_ENV=dev \
  -e NETWORK=rinkeby \
  -e MARKETS=WETH-RDN \
  -e WETH_TOKEN_ADDRESS=0xc58b96a0278bd2c77bc93e01b148282fb8e753a5 \
  -e RDN_TOKEN_ADDRESS=0xde6efd396e18a950b45e24d6225505f48d0c627b \
  gnosispm/dx-services:develop \
  npm run bots


#   -e MARKETS=WETH-RDN,WETH-OMG \
#   -e WETH_TOKEN_ADDRESS=0xc58b96a0278bd2c77bc93e01b148282fb8e753a5 \
#   -e RDN_TOKEN_ADDRESS=0xde6efd396e18a950b45e24d6225505f48d0c627b \
#   -e OMG_TOKEN_ADDRESS=0x8bdd53a53c53fbc40d38231d6219f8cb7b59a704
```

Fill the environment variables with your own configuration:

* `MNEMONIC`: 
  * Use your secret BIP39 mnemomic. 
  * The bot address will be the first 
account generated by that mnemonic.
* `NODE_ENV`: 
  * Optional, `local` is the default
  * Can be one of the following: `local`, `dev`, `pre` or `pro`
* `NETWORK` (optional):
  * Not mandatory, if you provide the URL for the ehtereum node 
    `ETHEREUM_RPC_URL` (recomended).
  * Can be `local`, `dev`, `pre` or `pro`
* `MARKETS`: List of the ERC20 token pairs you want the bots to watch.
  * Format: `<token1>-<token2>[,<tokenN>-<tokenM>]*`
    * Example: `WETH-RDN,WETH-OMG`
  * It's important that for every distinct token provided, you also provide the
    addresse, in the case of the previous example:
    * **WETH_TOKEN_ADDRESS**: `0xc58b96a0278bd2c77bc93e01b148282fb8e753a5`
    * **RDN_TOKEN_ADDRESS**: `0xde6efd396e18a950b45e24d6225505f48d0c627b`
    * **OMG_TOKEN_ADDRESS**: `0x8bdd53a53c53fbc40d38231d6219f8cb7b59a704`

When you run it for the first time, you should see something similar to:

![alt text](./docs/img/run-docker.png "Run the bots with docker")

Don't worry for now about the **WARN** message shown at the bottom, we'll deal
with it in the **Fund the bots** section.

This script will:

* Start **3** bots that will make sure the liquidity: `SellLiquidityBot`, 
`BuyLiquidityBot` and `BalanceBot` (more info about these bots in
[Dutch X Bots](./docs/bots.md)
)
* Runs a simple API server that exposes basic information: 
[http://localhost:8081]()

# Fund the bots
The bots automatically participate in the auctions performing bids and asks when
the time is right.

In order to do this bids and asks, they need to have a balance in the `Dutch X` 
smart contract.

For founding the bots, we need to know their Ethereum address, this is 
determined by the secret mnemonic you've used to run the bots.

An easy way to know the address is just to visit the about endpoint:

* [http://localhost:8081]()

You should see among other information, the accounts used by the bots:

![alt text](./docs/img/bot-account.png "Get the account of the bors")

Once you have the **bot account**, your **secret mnemonic** and the 
**bots running**, you are all set for the funding.

> The easiest way is to use the **Dutch X CLI**.
>
> Check out the **Funding** section 
in [Dutch X Cli page](./docs/cli.md)

## Specify the Ethereum RPC Node
It's best, if you configure an ethereum node you consider reliable.

You can specify the url to the node by using the `ETHEREUM_RPC_URL` environment 
variable. Example:

* **ETHEREUM_RPC_URL**: `http://localhost:8545`

# Dutch X CLI (Command Line Interface)
In the docker image, it's also avaliable a CLI, with
some basic operatios for using the Dutch X.

## State of a Dutch X Auction
There's a basic command in the CLI that is very helpful to get the state of the
auctions.

It's a good idea if we create `state.sh` script simar to:

```
TOKEN_PAIR=${1:-WETH-RDN}

docker run \
  -e NETWORK=rinkeby \
  -e MARKETS=WETH-RDN \
  -e WETH_TOKEN_ADDRESS=0xc58b96a0278bd2c77bc93e01b148282fb8e753a5 \
  -e RDN_TOKEN_ADDRESS=0xde6efd396e18a950b45e24d6225505f48d0c627b \
  gnosispm/dx-services:develop \
  npm run cli -- state $TOKEN_PAIR
```

Give execution permisions to the script:
```
chmod +x ./state.sh
```

This way, we can ask the CLI for the state of any known token pair:

```
./state.sh WETH-RDN
```

We would get something similar to:

![alt text](./docs/img/state-of-auction.png "State of an auction")

## More CLI commands
The CLI offers more commands, read more about the CLI in: 
* [Dutch X Cli page](./docs/cli.md)

# Dutchx Bots
There are 3 bots:

* `SellLiquidityBot`
* `BuyLiquidityBot`
* `CheckBakanceBot`

Find out more about them in:
* [Dutch X Bots page](./docs/bots.md)

# Debug
To increase the debug level, you can change the bot script to run with 
`run bots-dev` instead of `run bots`.

> Don't forget to change it back for the production script.