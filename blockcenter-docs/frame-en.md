#  Bytom DAPP development process

From the current release of `DAPP`, the `DAPP` architecture can be roughly divided into three types: plug-in wallet mode, full-node wallet mode and compatibility mode.
- Plug-in wallet mode communicates with the blockchain node via the `RPC` protocol by means of a browser plug-in that encapsulates the wallet. The plug-in will inject the `Web3` framework into the `DAPP` front-end page at runtime, and then `DApp` pass. Web3` communicates with the blockchain node.
- The full-node wallet mode requires the project side to synchronize and hold a blockchain node and provide a browser environment to interact with the user.
- Compatibility mode can be used simultaneously under plug-in wallet and full-node wallet, that is, the above two methods can be switched freely, and the security performance is relatively high.

The architecture pattern of the original chain `DAPP` is similar to the plug-in wallet mode of the account model `DAPP`, which is composed of the `DAPP` front-end, the plug-in wallet and the contract program. The plug-in wallet needs to be connected and decentralized. The blockchain server `blockcenter`, which is mainly used to manage plugin wallet related information. In addition, the blockchain system with the original chain is the `UTXO` model, the contract program exists in the stateless `UTXO`. If you want to implement such a specific `DAPP`, you need to do more logic on the front end and the back end. Processing.

# 1. Write, compile, and instantiate smart contracts

## Writing smart contracts


The virtual machine than the original chain is Turing-complete, and theoretically it can realize the operation that any Turing computer can achieve. And `Equity` as a smart contract language than the original chain, using the `Equity` language can implement many typical financial model cases, but in order to solve the downtime problem, the upper limit of the fee is also set than the original chain, so the user is designing the contract. Time to make a trade-off.


The contract template structure is as follows:

```
contract contract_name(...) locks valueAmount of valueAsset {
  clause clause_name(...) {
    ...
    lock/unlock ...
  }
  ...
}
```

`Equity` has a simple grammatical structure, and the statement has clear meaning. The children's shoes with development experience can basically understand the meaning of the contract. For writing smart contracts, you can refer to the [`Equity` contract introduction] (https://docs.bytom.io/mydoc_smart_contract_overview.cn.html). The syntax and compilation methods of the `Equity` language are described in detail in the documentation. In addition, the document also introduces some typical [template contracts] (https://docs.bytom.io/mydoc_contract_template.cn.html), developers can refer to their own needs.

##  Compile and instantiate the contract

The compilation contract currently supports two methods, one is to use the `Equity` compiler, and the other is to call the `RPC` interface `compile` that is compiled in the original chain; and the contract instantiation is to set the contract script according to the user. The fixed parameters are locked. Compiling and instantiating the contract can refer to the upper part of the [Compile and Instantiate Contract] (https://docs.bytom.io/mydoc_smart_contract_build.cn.html). This document not only introduces the contract. The parameter construction description also details the steps of compiling the contract. The compiler and related tools are located in the [`Equity` compiler] (https://github.com/Bytom/equity) and are developed using the `go` language. Users can download the source code and compile it.



Examples of tool compilation and instantiation are as follows:

```sh
// compile
./equity [contract_name] --bin

// instance
./equity [contract_name] --instance [arguments ...]
```

# 2. Deployment contract

The deployment contract sends a contract transaction, and the specified number of assets are sent to the contract `program` than the original build's `build-transaction` interface. Just set the receiver `control_program` in the output `output` to the specified contract. . Users can refer to the Lock Contract section in [Contract Transaction Description] (https://docs.bytom.io/mydoc_smart_contract_build.cn.html). The structure of the transaction can be referred to in the document. If the contract transaction is successfully sent and the transaction has been successfully chained, the `UTXO` of the contract can be found by calling the `API` interface `list-unspent-outputs`.

The deployment contract transaction template is roughly as follows:

```js
{
  "actions": [
    // inputs
    {
	// btm fee
    },
    {
	amount, asset, spend_account
	// spend user asset
    },

    // outputs
    {
	amount, asset, contract_program
	// receive contract program with instantiated result
    }
  ],
  ...
}
```


# 3. Building a DAPP architecture


The original `blockcenter` server is the officially developed decentralized plug-in wallet server, which developers can call in accordance with the relevant `API` interface. The `DAPP` overall framework model of the original chain is as follows:

![image](https://raw.githubusercontent.com/huangxinglong/picture/master/blockcenter-docs/images/frame.png)


## DAPP front end

There are two main aspects to building a `DAPP` front end: one is the interaction between the front end and the plugin wallet, the other is the logical processing of the front end, and the interaction plug-in wallet with the buffer server is the window for communicating with the blockchain node server, a `DAPP `In order to communicate with the blockchain node, you need to interact with the background server node by means of plugins. In addition to the original plug-in wallet, in addition to interacting with the background server, it also contains some interface 'API' for local business logic processing. For details, please refer to [DAPP Developer Guide] (https://github.com/Bytom/Bystore) /wiki/Dapp-Developer-Guide). Since the original chain is a blockchain system based on the `UTXO` model, the transaction is a structure composed of multiple inputs and multiple outputs, and the position of the transaction input or output also needs to be arranged in order, so the development of `DAPP` requires front-end processing. In addition to the logic of building transactions, the number of calculations involved in the `lock phase unlock` statement in the contract needs to be pre-computed according to the abstract syntax tree. The result of the calculation will be used to build the transaction, and `verify`,` If the -else` other statement type also needs to be pre-verified, it will prevent the user from reporting an error when executing the contract.


From a functional perspective, the front end mainly includes the design of the page, the call of the plugin, the processing of the contract transaction logic, and the interaction of the buffer server. Next, explain these important parts:

- 1) The design of the front-end page is mainly the design of the web interface. This part of the developer can choose the page mode by himself.


- 2) The plugin wallet has been structured and packaged, and the external interface is provided to the `DAPP` developer. The developer only needs to populate the parameters of the plugin according to the rules. For details, please refer to [DAPP Developer Guide] (https) ://github.com/Bytom/Bystore/wiki/Dapp-Developer-Guide)

- 3) The contract transaction than the original chain is a multi-input and multi-output transaction structure. The front end needs to perform some pre-judgment logic processing, and then select the appropriate contract transaction template structure.


- 4) The plugin of DAPP is connected to the decentralized `bycoin` server, which is all package information and transaction information synchronized from the original node server. This part is mainly encapsulated at the plugin wallet level. Users only need to call according to the interface. In addition, developers need to build a buffer server, not only can do some performance processing in the management contract `UTXO` level, but also can do some data storage for `DAPP`. Developers can develop some `RPC` request interfaces based on actual needs, and then set relevant conditions on the front-end page to trigger these `API` calls.


The front-end logic processing flow is roughly as follows:

- Call the plugin, located in the [Bytom-JS-SDK] (https://github.com/Bytom/Bytom-JS-SDK) than the original `chrome` plugin source. The instructions for calling the plugin when developing the original `DAPP` can be Refer to the [Dapp Developer Guide] (https://github.com/Bytom/Bystore/wiki/Dapp-Developer-Guide) and its network configuration is as follows:

  ```js
    window.addEventListener('load', async function() {

      if (typeof window.bytom !== 'undefined') {
        let networks = {
            solonet: ... // solonet bycoin url 
            testnet: ... // testnet bycoin url 
            mainnet: ... // mainnet bycoin url 
        };

        ...

        startApp();
    });
    ```

- Configure the contract parameters, you can use the file configuration method. This step is to let the front end get some fixed contract parameters that need to be used. The front-end configuration file is [`configure.json.js`] (https:// github.com/Bytom/Bytom-Dapp-Demo/blob/master/contracts/configure.json.js), the sample model is as follows:

    ```js
    var config = {
        "solonet": {
            ...         // contract arguments
            "gas": 0.4  // btm fee
        },
        "testnet":{
            ...
        },
        "mainnet":{
            ...
        }
    }
    ```

- Front-end pre-calculation processing, if the contract contains a `lock-unlock` statement, and `Amount` is a numeric expression, then the front end extracts the calculation expression and performs the corresponding pre-calculation. In addition, the front end also needs to prejudge all verifiable `verify` statements to determine whether the transaction is feasible, because once the front end fails to verify these, the contract will inevitably fail. In addition, if the variables involved in the `define` or `assign` statements, the front end also needs to pre-calculate the values ​​of these variables.

- Build a contract trading template. Since the unlocking contract is to unlock the `lock` statement condition, the construction transaction needs to be transformed according to the `lock` statement or the `unlock` statement. The unlock contract transaction is composed of `inputs` and `outputs`. The first `input` input of the transaction is generally fixed, that is, the `hash` value of the contract `UTXO`, except for other input and output. The changes need to be made according to the actual contract in DAPP. The model is as follows:


    ```js
    const input = []
    input.push(spendUTXOAction(utxohash))
    ... // other input

    const output = []
    output.push(controlProgramAction(amount, asset, program))
    ... // other output
    ```


- Start the front-end service

    Compile the front-end commands as follows:

    ```sh
    npm run build
    ```

    Before starting, you need to start the `bufferserver` buffer server, and then start the front-end service. The front-end startup commands are as follows:

    ```sh
    npm start
    ```

## DAPP buffer server

The buffer server is mainly for the efficiency of the management contract `UTXO` level, including how to synchronize the request to the `bycoin` server, in addition to the related transaction records of `DAPP`. The `bycoin` server is a decentralized wallet server than the original chain. The buffer server's `UTXO` is updated synchronously with it, which is the default connection to the original official plugin wallet. Although the `bycoin` server is also managed against all the `UTXO` of the original chain, but because the number of `UTXO` is relatively large, if it is processed directly at this level, the performance of `DAPP` will be poor, so users are advised to build their own. The buffer server is further optimized. In addition, the `DAPP` developer can also build their own decentralized wallet server and develop related plug-ins themselves.


The buffer server architecture can refer to the source code of [bufferserver case] (https://github.com/oysheng/bufferserver). The compilation and startup steps are as follows:


- Compile `bufferserver` source code


    Follow the [`README`] (https://github.com/oysheng/bufferserver/blob/master/README.md) package to install the packages `Mysql` and `Redis` required by the deployment service, then download the source code and compile:

    ```sh
    make all
    ```

    After the compilation is complete, the executable files `api` and `updater` are generated in the `target` directory.


- Start the service

    
   Use the `root` user to create the database and data table with the following commands:


    ```sh
    mysql -u root -p < database/dump.sql
    ```

    Modify the configuration file `config_local.json`, the configuration instructions refer to [`README`] (https://github.com/oysheng/bufferserver/blob/master/README.md) for the `config` configuration parameter.

    Start the `api` and `updater` servers, where `api` is the service process       that provides the `JSON RPC` request, and `updater` is the service process      that provides the synchronization `blockcenter` and blockchain browser data     requests.

     ```sh
    ./target/api config_local.json

    ./target/updater config_local.json
    ```


     After starting the buffer server, you can start the front-end service, and then open the `DAPP` web page `URL` to use.

     Attachment: The `JSON RPC` interface of the buffer server can refer to [`wiki` interface description] (https://github.com/oysheng/bufferserver/wiki).


# bytom DAPP instance

For an example of the original `DAPP`, please refer to [Saving Dividend DAPP] (./instance.md)