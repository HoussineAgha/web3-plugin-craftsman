# web3.js Plugin: web3-plugin-craftsman

This package is a plugin for web4.js version 4. And it allows instantiating contracts directly from Solidity source code or a solidity file. So, instead of requiring an ABI and bytecode, you can pass the source code to ExtendedContract and it will:
  - Compile the solidity code.
  - Generate the ABI and bytecode (you can read them and save them)
  - Set bytecode internally, so it will be used when deploying the Smart Contract.

## Installation

```sh
yarn add web3-plugin-craftsman
``` 
or
```sh
npm install web3-plugin-craftsman
``` 
## Usage

Basically, you just need to create an object of `ExtendedContract` by passing the solidity source code. 

After that, you have to call `await contract.compilationResult` to be able to wait for the compilation to finish.

### Creating an instance of the `ExtendedContract` object

However, there is 3 ways to create an object of `ExtendedContract`:
- First alternative is to instantiate from the `ExtendedContract` class
  ```ts
  import { ExtendedContract } from 'web3-plugin-craftsman';

  // Using ExtendedContract directly
  const contract = new ExtendedContract(sourceCodeOrPath);

  // You need to set the provider only if you need to deploy or execute methods
  // You do not need it incase you only need to compile the source code and save the ABI and bytecode for later usage. 
  contract.provider = 'http://localhost:8545';
  ```

- Alternatively, you can create the contract using an instance of `ExtendedWeb3` that is basically the same as if you created an object of `Web3`. But it has `.eth.ExtendedContract` that is equivalent to `.eth.Contract` but with the ability to accept a smart contract source code.

  ```ts
  import { ExtendedWeb3 } from 'web3-plugin-craftsman';

  // Using ExtendedWeb3
  const web3 = new ExtendedWeb3('http://localhost:8545');

  const contract = new web3.eth.ExtendedContract(sourceCodeOrPath);
  ```

- As a third alternative, you can use the functionality as a plugin to Web3. This is helpful if you like to using this functionality on an existing `Web3` object.

  ```ts
  import { ExtendedWeb3 } from 'web3-plugin-craftsman';


  const web3 = new Web3('http://localhost:8545'); // your Web3 object

  // Using ExtendedWeb3 as a plugin
  web3.registerPlugin(new ExtendedWeb3());

  const contract = new web3.craftsman.ExtendedContract(sourceCodeOrPath);
  ```


### Deploying and interacting with your Contract

After you have your `contract` object created, the rest is the same regardless of the alternative you chose:

```ts
// ... after you initialized the contract using one of the 3 alternatives above ... the rest is the same:

// Wait for the contract compilation and handle compilation errors if any
try {
  const compilationResult = await contract.compilationResult;
  // the compilationResult will consists of:
  // {
  //     abi: ContractAbi,
  //     bytecodeString: string,
  // }
  // Note: The contract object will not recognize the ABI. To make it do so, save the ABI to a file and assign it to a variable and add `as const` to it.
} catch (e) {
  console.log(e);
}


// Deploy contract   
const deployed = await contract
  .deploy({ arguments: [1000] })
  .send({ from: accounts[0] });
  
// Call a method     
const myNumber = await deployed.methods.myNumber().call();

// Send a transaction
await (deployed.methods.setMyNumber(100) as any).send({ from: accounts[0] });

// Call a method  
const myNumberUpdated = await deployed.methods.myNumber().call();

```

### The constructor options

Note that, you can pass one of the following, as the first argument, to the extended contract contractor. (This is what you can enter to replace the variable `sourceCodeOrPath` mentioned above):
- A string containing a Smart Contract source code
- A string containing a Smart Contract file path
- An object containing the source code and the Smart Contract name:
  ```ts
  {
    sourceCode: string;
    // The name must be specified if there are multiple contracts
    contractName: string;
  }
  ```
- An object containing the path of the file (path/file.sol), or the paths, and the Smart Contract name:
  ```ts
  {
    // Pass an array, if the contract inherits from other contracts that exist at other files
    path: string | string[];   
    // The name must be specified if there are multiple contracts 
    contractName: string;
  }
  ```

#### Examples for the constructor options

And here is an example for passing the file path (the recommended way):
```ts
const contract = new ExtendedContract('./test/smart_contracts/simple-contract.sol');
```

Or as the following when you need to pass multiple files:
```ts
const contract = new ExtendedContract({
    path: [
      './test/smart_contracts/simple-contract.sol',
      './test/smart_contracts/child-contract.sol'
    ],
    contractName: 'ChildContract',
  }
  );
```

And here is an example for passing the solidity code in-line (not recommended):
```ts
const contract = new ExtendedContract('
  // SPDX-License-Identifier: MIT
  pragma solidity ^0.8.0;

  contract SimpleContract {
      uint256 public myNumber;

      constructor(uint256 _myNumber) {
          myNumber = _myNumber;
      }

      function setMyNumber(uint256 _myNumber) public {
          myNumber = _myNumber;
      }
  }
'
);
```



## Features

- Compile Solidity source code.
- The bytecode and ABI are available to read (and possibly saved for faster runtime later).
- When deploying the Contract, the generated bytecode is used internally automatically (no need to provide it).
- The rest is exactly as you would interact with `Contract` instance. 

Note: The intellisense would not be available according to the contract ABI. To have it you just need to save the ABI and the bytecode to a file, and assign the ABI to a variable and add `as const` to it. Then you can pass those variables next time instead of the source code.

## Contributing

Please include relevant test cases to cover any new functionality.

## Compatibility

This plugin is compatible with:

- web3.js version 4.x
- Node.js 16+
- Browsers (via Webpack/Rollup/Browserify)

## Troubleshooting

Just try to run your code again, if you faced the following error:

> EACCES: permission denied, open '/home/maltabba/repos/experiments/web3-plugin-craftsman/node_modules/solc-typed-ast/.compiler_cache/wasm/soljson-v0.8.20+commit.a1b79de6.js'

And if the error persist, you may open an issue at: https://github.com/ConsenSys/solc-typed-ast. 

---

If you like to contribute to the development of this package, you may check the [CONTRIBUTION.md](CONTRIBUTION.md) file.