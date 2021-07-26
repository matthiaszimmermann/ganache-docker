# ganache docker

use [ganache](https://www.trufflesuite.com/ganache) for a ethereum sandbox installation.

the sandbox ethereum chain can be used for interaction with the browser extension [metamask](https://metamask.io/) 
and/or via commonly used javascript api like 
[web3](https://web3js.readthedocs.io) or 
[ethers](https://docs.ethers.io/).

instead of actually installing software you may use the dockerized setup provided by this repository.

## create image

the docker image produced by the dockerfile creates a simple ganache network for testing and exploration.

```bash
docker build -t ganache .
```

the chosen setup deterministically creates the addresses (and private keys) provided below.

### accounts

0. 0x627306090abaB3A6e1400e9345bC60c78a8BEf57 (10000 ETH)
1. 0xf17f52151EbEF6C7334FAD080c5704D77216b732 (10000 ETH)
2. 0xC5fdf4076b8F3A5357c5E395ab970B5B54098Fef (10000 ETH)
3. 0x821aEa9a577a9b44299B9c15c88cf3087F3b5544 (10000 ETH)
4. 0x0d1d4e623D10F9FBA5Db95830F7d3839406C6AF2 (10000 ETH)
5. 0x2932b7A2355D6fecc4b5c0B6BD44cC31df247a2e (10000 ETH)
6. 0x2191eF87E392377ec08E7c08Eb105Ef5448eCED5 (10000 ETH)
7. 0x0F4F2Ac550A1b4e2280d04c21cEa7EBD822934b5 (10000 ETH)
8. 0x6330A553Fc93768F612722BB8c2eC78aC90B3bbc (10000 ETH)
9. 0x5AEDA56215b167893e80B4fE645BA6d5Bab767DE (10000 ETH)

### private keys

0. 0xc87509a1c067bbde78beb793e6fa76530b6382a4c0241e5e4a9ec0a0f44dc0d3
1. 0xae6ae8e5ccbfb04590405997ee2d52d2b330726137b875053c36d94e974d162f
2. 0x0dbbe8e4ae425a6d2687f1a7e3ba17bc98c673636790f1b8ad91193c05875ef1
3. 0xc88b703fb08cbea894b6aeff5a544fb92e78a18e19814cd85da83b71f772aa6c
4. 0x388c684f0ba1ef5017716adb5d21a053ea8e90277d0868337519f97bede61418
5. 0x659cbb0e2411a44db63778987b1e22153c086a95eb6b18bdf89de078917abc63
6. 0x82d052c865f5763aad42add438569276c00d3d88a2d062d36b2bae914d58b8c8
7. 0xaa3680d5d48a8283413f7a108367c7299ca73f553735860a87b08f39395618b7
8. 0x0f62d96d6675f32685bbdb8ac13cda7c23436f63efbb9d07700d8669ff12b7c4
9. 0x8d5366123cb560bb606379f90a0bfd4769eecc0557f1b362dcae9012b548b1e5

### mnemonic

for the account creation above a hd wallet with the mnemonic `candy maple cake sugar pudding cream honey rich smooth crumble sweet treat` is used. for the creation of the individual accounts the hd path `m/44'/60'/0'/0/{account_index}` is used.

## run container

```bash
docker run -p 7545:8545 -d --name ganache ganache
```

port 7545 is chosen to avoid conflicts with any productive local ethereum client (that would run on default port 8545).

## interaction with metamask

you may interact with ganache via [metamask](https://metamask.io/) via a custom network. 

to add such a custom network use "Settings" and "Networks" and "Add Network".
in the form, use the following custom network settings.

* network name: ganache
* new rpc url: http://localhost:7545
* chain id: 1234

you can now add the first ganache account as follows

1. via account selector use "Import Account"
2. for select type "Private Key" enter `0xc87509a1c067bbde78beb793e6fa76530b6382a4c0241e5e4a9ec0a0f44dc0d3`
3. click on "Import"
4. the new account is selected
5. you may edit the account name via "Account details" to `ganache[0]`

using this account you can transfer funds to some other account address.

1. click "Send"
2. enter `0xf17f52151EbEF6C7334FAD080c5704D77216b732` into field "Add Recipient"
3. set the amount field to `1` ether and click "Next"
4. create and send the transaction to ganache with "Confirm" 

should the chain data diverge from the state in metamask (eg via repeated stopping/starting with new chain data) you may need to reset the account state in metamask.

1. make sure that you have selected the account you want to reset (eg `ganache[0]`)
2. use the account selector to enter "Settings"
3. use "Advanced" and click on "Reset Account"

alternatively, you may remove and reinstall the metamask extension.

## persisting chain data

in cases starting/stopping the container should keep the state of the chain data the data needs to be persisted outside the container. the simplest way to do this is to mount a local directory.

```bash
docker run -p 7545:8545 -d -v $PWD/data:/var/lib/ganache/data --name ganache ganache_local
```

alternatively use a docker volume to store the chain data

```bash
docker volume create ganache_data
docker volume inspect ganache_data
docker run -p 7545:8545 -d -v ganache_data:/var/lib/ganache/data --name ganache ganache_local
```

to reset the chain data, delete and recreate the volumen.
to delete the volume you may use the command below.

```bash
docker volume rm ganache_data
```

## api interaction

you may connect to the running container to interact with the ganache chain with the following command

```bash
docker exec -it ganache sh
```

inside the docker container, check the installed node packages. then, start node and use the web3 and or the ethers api to do what you need. 

```sh
/app # npm ls -g --depth 0
/usr/local/lib
+-- ethers@5.4.2
+-- ganache-cli@6.12.2
+-- npm@7.19.1
`-- web3@1.4.0
/app # node
Welcome to Node.js v16.5.0.
Type ".help" for more information.
>
```

to exit the online node session use `.exit`. then, to exit the container use `exit`.

### interaction via web3

sample commands for [web3](https://web3js.readthedocs.io) inside an interactive node session (see above).

```js
const Web3 = require('web3')
const web3 = new Web3(new Web3.providers.HttpProvider('http://localhost:8545'))
web3.eth.getCoinbase(function (error, result) { if (!error) console.log('coinbase', result); else console.log('fail:', error); })

const addr = ('0x627306090abaB3A6e1400e9345bC60c78a8BEf57')
web3.eth.getBalance(addr, function (error, result) { if (!error) console.log(addr, '(', web3.utils.fromWei(result,'ether'), 'ETH)'); else console.log('fail:', error); })
```

### interaction via ethers

sample commands for [ethers](https://docs.ethers.io/) inside an interactive node session (see above).

```js
const { ethers } = require("ethers")
const provider = new ethers.providers.JsonRpcProvider('http://localhost:8545')

const addr = ('0x627306090abaB3A6e1400e9345bC60c78a8BEf57')
provider.getBalance(addr).then((result) => { console.log(addr, '(', ethers.utils.formatEther(result), 'ETH)') })

const signer = provider.getSigner()
const tx = signer.sendTransaction({ to: "0xf17f52151EbEF6C7334FAD080c5704D77216b732", value: ethers.utils.parseEther("1.0") })
```
