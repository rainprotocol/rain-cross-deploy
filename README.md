# Rain Cross Network Deploy

Utility to deploy Rain Protocol Contracts to EVM based Networks.
The dependency provides you with a way to get the `transaction data` for contract deployment of a particular `Rain Contract` which you can submit via a signer .

## Installation

```sh
npm install rain-x-deploy
```

## Usage

import (ESM, TypeScript):

```ts
import { getDeployTxData, RainNetworks, type DISpair } from "rain-x-deploy"
```

### Get Transaction Data for Contract Deployment

You can then encode this data inside a transaction object and submit the transaction .
If you intend to use it with frontend library like `React` or `Svelte`

### Deploy Rain Contracts

To deploy rain contracts use `getDeployTxData` :

```ts
import { getDeployTxData, RainNetworks, type DISpair } from "rain-x-deploy"
import { ethers } from "ethers"; 

const deployContract = async () => {
  if(!window.ethereum) alert("Provider not injected")  
    
  //Initialize provider
  const provider = new ethers.providers.Web3Provider(window.ethereum) 
    
  // Get the signer
  const signer = await provider.getSigner()  
    
  // DISpair contracts of the originating network
  const fromDIS: DISpair = {
    interpreter:'0x24035b15e908551a2e1b4f435384d9485766d296',
    store:'0xd28543743f017045c9448ec6d82f5568a0f26918',
    deployer:'0x32ba42606ca2a2b91f28ebb0472a683b346e7cc7'
  }  
     
  // DISpair contracts of the target network
  const toDIS: DISpair = {
    interpreter : '0xeBEA638926F7BA49c0a1808e0Ff3B6d78789b153' ,
    store : '0xB92fd23b5a9CBE5047257a0300d161D449296C03' , 
    deployer : '0x34a81e8bc3e2420efc8ae84d35045c0326b00bdc'
  } 

  // Get contract deployment tx data for target network
  const txData = await getDeployTxData(
    RainNetworks.Mumbai, // originating network
    "0xacf6069f4a6a9c66db8b9a087592278bbccde5c3", //Origin contract address to x-deploy
    {
      DIS: { // The DISpair instances provided as options
        from: fromDIS,
        to: toDIS,
      }
    }
  )
    
  // Submit the transaction
  await signer.sendTransaction({
    data :txData
  })
}
```

In case the originating and target networks are same(meaning you are simply trying to clone a contract on the network) you can pass the same DISpair as `fromDIS` and `toDIS`.

Eg:

```ts
const DISpair = {
  interpreter : '0xeBEA638926F7BA49c0a1808e0Ff3B6d78789b153' ,
  store : '0xB92fd23b5a9CBE5047257a0300d161D449296C03' , 
  deployer : '0x34a81e8bc3e2420efc8ae84d35045c0326b00bdc'
} 

// Pass the same DIS as origin and target to clone the contract.
const txData = await getDeployTxData(
  RainNetworks.Ethereum, // originating network
  "0xce0a4f3e60178668c89f86d918a0585ca80e0f6d", //Origin contract address to x-deploy
  {
    DIS: {
      from: DISpair,
      to: DISpair,
    }
  }
)
```

#### Deploy RainterpreterExpressionDeployer

To deploy `RainterpreterExpressionDeployer`, now it is possible to use the same `getDeployTxData` function, using DISpair objects but without providing the `deployer` field.

```ts
const fromDIS = {
  interpreter: '0x24035b15e908551a2e1b4f435384d9485766d296',
  store: '0xd28543743f017045c9448ec6d82f5568a0f26918'
}   

const toDIS = {
  interpreter : '0xeBEA638926F7BA49c0a1808e0Ff3B6d78789b153' ,
  store : '0xB92fd23b5a9CBE5047257a0300d161D449296C03'
}

const txData = await getDeployTxData(
  RainNetworks.Mumbai, // Originating network
  "0x32ba42606ca2a2b91f28ebb0472a683b346e7cc7", // Deployer address from originating network
  {
    DIS: {
      from: fromDIS, // Originating network DIS
      to: toDIS, // Target network DIS
    }
  }
) 
```

#### Provide transaction hash

If have doubts about if the package can obtain the transaction data of an address, there is a exposed function called `checkObtainTxHash` that check it. Generally it will work to inform a consumer about it and request for the transaction hash manually. Then, you can pass the transaction hash as an option when calling `getDeployTxData`.

```ts
const network = RainNetworks.Mumbai;
const contractAddress = '0x9278fe76aac69745a6af85ac6d1c456a2921fdf5';

const checkTxHash = await checkObtainTxHash(network, contractAddress); // false

const txHashProivded = '0x2cdef2c3521553e677b49916bfc6427387a97bb21a5781a212059ffd4df89fe9';

const txData = await getDeployTxData(network, contractAddress, {
  txHash: txHashProivded
})

```

### Supported Networks

You can deploy contracts to any of the following networks by importing `RainNetworks` enum.

```ts
export enum RainNetworks{
    Mumbai = 'mumbai' ,
    Polygon = 'polygon',
    Ethereum = 'ethereum'
} 
```

#### Get network from chain ID

This allow you to get the correct network to be use when calling the function

```ts
const networkA = getRainNetworkForChainId(1) // For Ethereum 
const networkB = getRainNetworkForChainId(137) // For Polygon 
const networkC = getRainNetworkForChainId(80001) // For Mumbai 
```

## Scenarios when consuming

Please keep in mind that in certain cases, using this function without understanding the type of contract may not yield any result or may produce incorrect or unknown outcomes. Let's explore some scenarios and their possible outputs:

- If it's a rain contract using DISpair instances on the supported chains, it will generate the correct transaction data.
- If it's a rain contract that does not use DISpair instances and lacks a constructor, it will still generate the correct transaction data by utilizing the provider.
- If it's a rain contract with a constructor that is not chain-dependent, it will generate the correct transaction data without modifying the transaction data.

Please note that understanding the contract type is crucial to ensure accurate and expected results when consuming this function.

## Limitations

As mentioned previously some scenearios, also is worth to mention the limitations of the package that maybe serve to the consumer as basic understanding about the previous scenarios and potentially other news. The rain tooling used on the package mainly track the rain contracts that use DISpair instances and ExpressionDeployers on their networks. The deploy transaction hash is possible to obtain. With that said, there are other rain contracts that don't use DISpair instances, which means that they are not present on the Rain subgraphs. On those cases, the package will try to obtain their deploy transaction by using other methods, like the BlockScanner APIs (etherscan, polygonscan, etc). Sadly, even these APIS have their limitations on test networks (like sepolia or mumbai), that does not support the endpoints to return the required deploy transacton. This is the reason why the package allow to provide the `txHash` as option, so the code could retrieve the transaction and check that match with the address.

## Ask for network support

If need other network to be supported on this package, please open a new issue
with the proposal and the basic required information to add the network like the
the block scanner of the desired network and their API documention.
