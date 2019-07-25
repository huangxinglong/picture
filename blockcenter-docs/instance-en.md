# Savings Dividend DAPP

##saving dividend agreement

The savings dividend contract means that the project party has initiated a lockout plan (ie, a savings contract and a cash withdrawal contract). The user can freely choose the lock amount to participate in the plan during the preparation period, and can automatically acquire the lock position after the lockout expires. profit. Users can participate in the savings during the preparation period (`dueBlockHeight`), and can obtain the same amount of savings instrument assets by `1:1` according to the contract, and the assets of the user lock (`deposit`) will be placed in the cash withdrawal contract, and The project party cannot be used. When the lockout period (`expireBlockHeight`) arrives, the user can call the cash withdrawal contract to take out the assets that he saved and take it out together with the interest. The schematic is as follows:

![image](https://raw.githubusercontent.com/huangxinglong/picture/master/blockcenter-docs/images/en-diagram.png)

As can be seen from the above figure, the project party has released a lockout project with a profit of `20%`, in which the savings contract `FixedLimitCollect` locks the `1000` bill assets (`bill`) and the project party will be `200 `A savings asset (`deposit`) is locked into the interest contract. After the project party has released the contract, all users can participate. For example, in the above figure, the `user1` call contract saves `500`, and the `500` savings assets will be locked in the cash withdrawal contract `FixedLimitProfit`, while `user1` gets the `500` ticket assets, and the remaining zeros are changed. Assets will continue to be locked in the savings contract `FixedLimitCollect`, and so on, `user2` and `user3` are the same process until the savings contract has no assets. The cash withdrawal contract `FixedLimitProfit` is roughly the same as the savings contract model, except that the cash withdrawal contract is composed of multiple `UTXO`s, and the user can operate in parallel when withdrawing cash. However, if the face value in the contract cannot support the user to withdraw cash at one time, it needs to be extracted multiple times. For example, `user1` has `500` bill assets, and the total amount of principal and interest that can be obtained is `600`, but the depreciated `UTXO` face value is `500`, then `user1` can only take up to `500` at a time, and the rest `100` needs to construct a transaction to withdraw cash.


## Contract source code
```js
// Savings contract
import "./FixedLimitProfit"
contract FixedLimitCollect(assetDeposited: Asset,
                        totalAmountBill: Amount,
                        totalAmountCapital: Amount,
                        dueBlockHeight: Integer,
                        expireBlockHeight: Integer,
                        additionalBlockHeight: Integer,
                        banker: Program,
                        bankerKey: PublicKey) locks billAmount of billAsset {
    clause collect(amountDeposited: Amount, saver: Program) {
        verify below(dueBlockHeight)
        verify amountDeposited <= billAmount && totalAmountBill <= totalAmountCapital
        define sAmountDeposited: Integer = amountDeposited/100000000
        define sTotalAmountBill: Integer = totalAmountBill/100000000
        verify sAmountDeposited > 0 && sTotalAmountBill > 0
        if amountDeposited < billAmount {
            lock amountDeposited of assetDeposited with FixedLimitProfit(billAsset, totalAmountBill, totalAmountCapital, expireBlockHeight, additionalBlockHeight, banker, bankerKey)
            lock amountDeposited of billAsset with saver
            lock billAmount-amountDeposited of billAsset with FixedLimitCollect(assetDeposited, totalAmountBill, totalAmountCapital, dueBlockHeight, expireBlockHeight, additionalBlockHeight, banker, bankerKey)
        } else {
            lock amountDeposited of assetDeposited with FixedLimitProfit(billAsset, totalAmountBill, totalAmountCapital, expireBlockHeight, additionalBlockHeight, banker, bankerKey)
            lock billAmount of billAsset with saver
        }
    }
    clause cancel(bankerSig: Signature) {
        verify above(dueBlockHeight)
        verify checkTxSig(bankerKey, bankerSig)
        unlock billAmount of billAsset
    }
}
```

```js
// Cash withdrawal contract (principal plus interest)
contract FixedLimitProfit(assetBill: Asset,
                        totalAmountBill: Amount,
                        totalAmountCapital: Amount,
                        expireBlockHeight: Integer,
                        additionalBlockHeight: Integer,
                        banker: Program,
                        bankerKey: PublicKey) locks capitalAmount of capitalAsset {
    clause profit(amountBill: Amount, saver: Program) {
        verify above(expireBlockHeight)
        define sAmountBill: Integer = amountBill/100000000
        define sTotalAmountBill: Integer = totalAmountBill/100000000
        verify sAmountBill > 0 && sTotalAmountBill > 0 && amountBill < totalAmountBill
        define gain: Integer = totalAmountCapital*sAmountBill/sTotalAmountBill
        verify gain > 0 && gain <= capitalAmount
        if gain < capitalAmount {
            lock amountBill of assetBill with banker
            lock gain of capitalAsset with saver
            lock capitalAmount - gain of capitalAsset with FixedLimitProfit(assetBill, totalAmountBill, totalAmountCapital, expireBlockHeight, additionalBlockHeight, banker, bankerKey)
        } else {
            lock amountBill of assetBill with banker
            lock capitalAmount of capitalAsset with saver
        }
    }
    clause cancel(bankerSig: Signature) {
        verify above(additionalBlockHeight)
        verify checkTxSig(bankerKey, bankerSig)
        unlock capitalAmount of capitalAsset
    }
}
```

The source code description of the contract can be specifically referred to [`Equity Contract Introduction] (https://docs.bytom.io/mydoc_smart_contract_overview.cn.html).


### Precautions:

- The time period is not a specific time, but is estimated by the block height (the average block time interval is approximately `2.5 minutes)
- The accuracy of the original is `8`, ie `1BTM = 100000000 neu`. Normally, the calculations are in units of `neu`, but the maximum value of the `int64` type of the virtual machine is `9223372036854775807`, in order to Avoiding the value being too large causes the calculation to overflow, so an amount limit is imposed on the calculated amount (ie `amountBill/100000000`)
- In addition, `clause cancel` is the management method of the project party. If the savings or withdrawal is not full, the project party can also recover the remaining assets.


## Compiling and instantiating contracts

Compile the `Equity` contract to refer to the introduction of [`Equity` compiler] (https://github.com/Bytom/equity). If the parameters of the savings contract `FixedLimitCollect` are as follows:

```
assetDeposited          :c6b12af8326df37b8d77c77bfa2547e083cbacde15cc48da56d4aa4e4235a3ee
totalAmountBill         :10000000000
totalAmountCapital      :20000000000
dueBlockHeight          :1070
expireBlockHeight       :1090
additionalBlockHeight   :1100
banker                  :0014dedfd406c591aa221a047a260107f877da92fec5
bankerKey               :055539eb36abcaaf127c63ae20e3d049cd28d0f1fe569df84da3aedb018ca1bf
```

Where `bankerKey` is the administrator's `publicKey`, which can be accessed via the interface [`list-pubkeys`] (https://github.com/Bytom/bytom/wiki/API-Reference#list-pubkeys) Get, note that the administrator needs to save the corresponding `rootXpub` and `Path`, otherwise the `clause cancel` cannot be called correctly.


The instantiation contract command is as follows:

```sh
// Savings contract
./equity FixedLimitCollect --instance c6b12af8326df37b8d77c77bfa2547e083cbacde15cc48da56d4aa4e4235a3ee 10000000000 20000000000 1070 1090 1100 0014dedfd406c591aa221a047a260107f877da92fec5 055539eb36abcaaf127c63ae20e3d049cd28d0f1fe569df84da3aedb018ca1bf


// cash withdrawal contract
./equity FixedLimitProfit --instance c6b12af8326df37b8d77c77bfa2547e083cbacde15cc48da56d4aa4e4235a3ee 10000000000 20000000000 1090 1100 0014dedfd406c591aa221a047a260107f877da92fec5 055539eb36abcaaf127c63ae20e3d049cd28d0f1fe569df84da3aedb018ca1bf
```

## Publishing contract transactions

The contract transaction is about to lock the asset into the contract. Since it is currently not possible to construct a contract transaction on the original `dashboard`, external tools are required to send contract transactions, such as `postman`. According to the above diagram, the project party needs to issue a savings contract of '1000` savings assets and a '200' interest asset withdrawal contract. Suppose the project party needs to issue a '1000` savings asset (if the precision is `8`, then `1000` is represented as `100000000000` in the original chain), then he needs to lock the corresponding number of tickets. In the savings contract, the trading template is as follows:

```js
{
  "base_transaction": null,
  "actions": [
    {
      "account_id": "0ILGLSTC00A02",
      "amount": 20000000,
      "asset_id": "ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff",
      "type": "spend_account"
    },
    {
      "account_id": "0ILGLSTC00A02",
      "amount": 100000000000,
      "asset_id": "13016eff73ffb7539a69e122f80f5c1cc94446773ac3f64dec290429f87e73b3",
      "type": "spend_account"
    },
    {
      "amount": 100000000000,
      "asset_id": "13016eff73ffb7539a69e122f80f5c1cc94446773ac3f64dec290429f87e73b3",
      "control_program": "20055539eb36abcaaf127c63ae20e3d049cd28d0f1fe569df84da3aedb018ca1bf160014dedfd406c591aa221a047a260107f877da92fec5024c04024204022e040500c817a8040500e40b540220c6b12af8326df37b8d77c77bfa2547e083cbacde15cc48da56d4aa4e4235a3ee4d3b02597a642f0200005479cda069c35b797ca153795579a19a695a790400e1f5059653790400e1f505967c00a07c00a09a69c35b797c9f9161644d010000005b79c2547951005e79895d79895c79895b7989597989587989537a894caa587a649e0000005479cd9f6959790400e1f5059653790400e1f505967800a07800a09a5c7956799f9a6955797b957c96c37800a052797ba19a69c3787c9f91616487000000005b795479515b79c1695178c2515d79c16952c3527994c251005d79895c79895b79895a79895979895879895779895679890274787e008901c07ec1696399000000005b795479515b79c16951c3c2515d79c16963aa000000557acd9f69577a577aae7cac890274787e008901c07ec169515b79c2515d79c16952c35c7994c251005d79895c79895b79895a79895979895879895779895679895579890274787e008901c07ec169632a020000005b79c2547951005e79895d79895c79895b7989597989587989537a894caa587a649e0000005479cd9f6959790400e1f5059653790400e1f505967800a07800a09a5c7956799f9a6955797b957c96c37800a052797ba19a69c3787c9f91616487000000005b795479515b79c1695178c2515d79c16952c3527994c251005d79895c79895b79895a79895979895879895779895679890274787e008901c07ec1696399000000005b795479515b79c16951c3c2515d79c16963aa000000557acd9f69577a577aae7cac890274787e008901c07ec16951c3c2515d79c169633b020000547acd9f69587a587aae7cac747800c0",
      "type": "control_program"
    }
  ],
  "ttl": 0,
  "time_range": 1521625823
}
```

After the contract transaction is successful, the `UTXO` corresponding to the contract `control_program` will be queried by all users, using the interface than the original chain [`list-unspent-outputs`] (https://github.com/Bytom/bytom/ Wiki/API-Reference#list-unspent-outputs) to query.

In addition, the developer needs to store the `assetID` and `program` of the contract `UTXO` to be called in the `config` configuration file of the `DAPP` front-end page and the `bufferserver` buffer server. As shown above:

```
// Savings contract
assetID：13016eff73ffb7539a69e122f80f5c1cc94446773ac3f64dec290429f87e73b3
program：20055539eb36abcaaf127c63ae20e3d049cd28d0f1fe569df84da3aedb018ca1bf160014dedfd406c591aa221a047a260107f877da92fec5024c04024204022e040500c817a8040500e40b540220c6b12af8326df37b8d77c77bfa2547e083cbacde15cc48da56d4aa4e4235a3ee4d3b02597a642f0200005479cda069c35b797ca153795579a19a695a790400e1f5059653790400e1f505967c00a07c00a09a69c35b797c9f9161644d010000005b79c2547951005e79895d79895c79895b7989597989587989537a894caa587a649e0000005479cd9f6959790400e1f5059653790400e1f505967800a07800a09a5c7956799f9a6955797b957c96c37800a052797ba19a69c3787c9f91616487000000005b795479515b79c1695178c2515d79c16952c3527994c251005d79895c79895b79895a79895979895879895779895679890274787e008901c07ec1696399000000005b795479515b79c16951c3c2515d79c16963aa000000557acd9f69577a577aae7cac890274787e008901c07ec169515b79c2515d79c16952c35c7994c251005d79895c79895b79895a79895979895879895779895679895579890274787e008901c07ec169632a020000005b79c2547951005e79895d79895c79895b7989597989587989537a894caa587a649e0000005479cd9f6959790400e1f5059653790400e1f505967800a07800a09a5c7956799f9a6955797b957c96c37800a052797ba19a69c3787c9f91616487000000005b795479515b79c1695178c2515d79c16952c3527994c251005d79895c79895b79895a79895979895879895779895679890274787e008901c07ec1696399000000005b795479515b79c16951c3c2515d79c16963aa000000557acd9f69577a577aae7cac890274787e008901c07ec16951c3c2515d79c169633b020000547acd9f69587a587aae7cac747800c0

// cash withdrawal contract
assetID：c6b12af8326df37b8d77c77bfa2547e083cbacde15cc48da56d4aa4e4235a3ee
program：20055539eb36abcaaf127c63ae20e3d049cd28d0f1fe569df84da3aedb018ca1bf160014dedfd406c591aa221a047a260107f877da92fec5024c040242040500c817a8040500e40b540220c6b12af8326df37b8d77c77bfa2547e083cbacde15cc48da56d4aa4e4235a3ee4caa587a649e0000005479cd9f6959790400e1f5059653790400e1f505967800a07800a09a5c7956799f9a6955797b957c96c37800a052797ba19a69c3787c9f91616487000000005b795479515b79c1695178c2515d79c16952c3527994c251005d79895c79895b79895a79895979895879895779895679890274787e008901c07ec1696399000000005b795479515b79c16951c3c2515d79c16963aa000000557acd9f69577a577aae7cac747800c0
```


## Savings Dividend DAPP Architecture Model

The `DAPP` overall framework model of the original chain describes the general structural model of `DAPP`, combined with the case of the savings dividend contract, the specific process is as follows:

![image](https://raw.githubusercontent.com/huangxinglong/picture/master/blockcenter-docs/images/en-flow.png)

### DAPP front end
    
The front-end logic processing flow of the savings dividend contract is as follows:

- 1) Call the plugin
  
The source code of the original `chrome` plugin is located at [Bytom-JS-SDK] (https://github.com/Bytom/Bytom-JS-SDK). For instructions on calling plugins when developing the original `DAPP`, refer to [Dapp Developer]. Guide](https://github.com/Bytom/Bystore/wiki/Dapp-Developer-Guide)

- 2) Configure contract parameters

The parameters to be instantiated in the `Dapp demo` are `assetDeposited`, `totalAmountBill`, `totalAmountCapital`, `dueBlockHeight`, `expireBlockHeight`, `additionalBlockHeight`, `banker`, `bankerKey`. The front-end configuration file is [`configure.json.js`] (https://github.com/Bytom/Bytom-Dapp-Demo/blob/master/contracts/configure.json.js)

```js
    var config = {
        "solonet": {
            "depositProgram": "2091194ddbf3614cafbadb1274c33e61afd4d5044c6ec4c30f8202980199c30083160014c800033d5e94de5f22e23a6d3cbeaed87b55bd640600204aa9d101050010a5d4e8203310d9951697418af3cdbe7a9cdde1dc49bb5439503dacb33828d6c9ef5af5a24dfc01567a64f5010000c358797ca153795579a19a6957790400e1f5059653790400e1f505967c00a07c00a09a69c358797c9f91616429010000005879c2547951005b79895a7989597989587989537a894c9a567a649300000057790400e1f5059653790400e1f505967800a07800a09a5a7956799f9a6955797b957c96c37800a052797ba19a69c3787c9f9161647c0000000059795479515979c1695178c2515b79c16952c3527994c251005b79895a79895979895879895779895679890274787e008901c07ec169638e0000000059795479515979c16951c3c2515b79c169639a000000567a567aae7cac890274787e008901c07ec169515879c2515a79c16952c3597994c251005a79895979895879895779895679895579890274787e008901c07ec16963f0010000005879c2547951005b79895a7989597989587989537a894c9a567a649300000057790400e1f5059653790400e1f505967800a07800a09a5a7956799f9a6955797b957c96c37800a052797ba19a69c3787c9f9161647c0000000059795479515979c1695178c2515b79c16952c3527994c251005b79895a79895979895879895779895679890274787e008901c07ec169638e0000000059795479515979c16951c3c2515b79c169639a000000567a567aae7cac890274787e008901c07ec16951c3c2515a79c16963fc010000567a567aae7cac747800c0",
            "profitProgram": "2091194ddbf3614cafbadb1274c33e61afd4d5044c6ec4c30f8202980199c30083160014c800033d5e94de5f22e23a6d3cbeaed87b55bd640600204aa9d101050010a5d4e820666f298d34806a6db0528c3b3d081bc00fa58aa393e7c42f90d67eb7db2a524f4c9a567a649300000057790400e1f5059653790400e1f505967800a07800a09a5a7956799f9a6955797b957c96c37800a052797ba19a69c3787c9f9161647c0000000059795479515979c1695178c2515b79c16952c3527994c251005b79895a79895979895879895779895679890274787e008901c07ec169638e0000000059795479515979c16951c3c2515b79c169639a000000567a567aae7cac747800c0",
            "assetDeposited": "3310d9951697418af3cdbe7a9cdde1dc49bb5439503dacb33828d6c9ef5af5a2",
            "assetBill": "666f298d34806a6db0528c3b3d081bc00fa58aa393e7c42f90d67eb7db2a524f",
            "totalAmountBill": 1000000000000,
            "totalAmountCapital": 2000000000000,
            "dueBlockHeight": 0,
            "expireBlockHeight": 0,
            "banker": "0014c800033d5e94de5f22e23a6d3cbeaed87b55bd64",
            "gas": 0.4
        },
        "testnet":{
            "depositProgram": "20f39af759065598406ca988f0dd79af9175dd7adcbe019317a2d605578b1597ac1600147211ec12410ce8bd0d71cab0a29be3ea61c71eb103c8260203da240203da2402060080f420e6b50600407a10f35a2000d38a1c946e8cba1a69493240f281cd925002a43b81f516c4391b5fb2ffdacd4d4302597a64370200005479cda069c35b790400e1f5059600a05c797ba19a53795579a19a695a790400e1f5059653790400e1f505967800a07800a09a6955797b957c9600a069c35b797c9f9161645b010000005b79c2547951005e79895d79895c79895b7989597989587989537a894ca4587a64980000005479cd9f6959790400e1f5059653790400e1f505967800a07800a09a5c7956799f9a6955797b957c967600a069c3787c9f91616481000000005b795479515b79c1695178c2515d79c16952c3527994c251005d79895c79895b79895a79895979895879895779895679890274787e008901c07ec1696393000000005b795479515b79c16951c3c2515d79c16963a4000000557acd9f69577a577aae7cac890274787e008901c07ec169515b79c2515d79c16952c35c7994c251005d79895c79895b79895a79895979895879895779895679895579890274787e008901c07ec1696332020000005b79c2547951005e79895d79895c79895b7989597989587989537a894ca4587a64980000005479cd9f6959790400e1f5059653790400e1f505967800a07800a09a5c7956799f9a6955797b957c967600a069c3787c9f91616481000000005b795479515b79c1695178c2515d79c16952c3527994c251005d79895c79895b79895a79895979895879895779895679890274787e008901c07ec1696393000000005b795479515b79c16951c3c2515d79c16963a4000000557acd9f69577a577aae7cac890274787e008901c07ec16951c3c2515d79c1696343020000547acd9f69587a587aae7cac747800c0",
            "profitProgram": "20f39af759065598406ca988f0dd79af9175dd7adcbe019317a2d605578b1597ac1600147211ec12410ce8bd0d71cab0a29be3ea61c71eb103c8260203da2402060080f420e6b50600407a10f35a20f855baf98778a892bad0371f5afca845191824dc8584585d566fbbc8ef1f304c4ca4587a64980000005479cd9f6959790400e1f5059653790400e1f505967800a07800a09a5c7956799f9a6955797b957c967600a069c3787c9f91616481000000005b795479515b79c1695178c2515d79c16952c3527994c251005d79895c79895b79895a79895979895879895779895679890274787e008901c07ec1696393000000005b795479515b79c16951c3c2515d79c16963a4000000557acd9f69577a577aae7cac747800c0",
            "assetDeposited": "00d38a1c946e8cba1a69493240f281cd925002a43b81f516c4391b5fb2ffdacd",
            "assetBill": "f855baf98778a892bad0371f5afca845191824dc8584585d566fbbc8ef1f304c",
            "totalAmountBill": 100000000000000,
            "totalAmountCapital": 200000000000000,
            "dueBlockHeight": 140506,
            "expireBlockHeight": 140506,
            "banker": "00147211ec12410ce8bd0d71cab0a29be3ea61c71eb1",
            "gas": 0.4
        }
    }
    ```


- 3) Front-end pre-calculation processing

 Taking the savings contract `FixedLimitCollect` as an example, the front end needs to pre-judicate the `verify` statement for the contract, in case the user fails to execute the parameter. In addition, the contract `billAmount of billAsset` indicates the locked assets and quantity, while `billAmount`, `billAsset` and `utxohash` are stored in the data table of the buffer server, so the front end needs to call `list-utxo` to find and All unspent utxo associated with the asset `asset` and `program`. For details, please refer to [`DAPP DEMO` Front End Case] ​​(https://github.com/Bytom/Bytom-Dapp-Demo/tree/master/src).

- 4) Transaction composition
        
The original transaction is a multi-input and multi-output template structure. If the contract contains multiple `lock` or `unlock` statements, then the user needs to construct a multi-input and multi-output transaction template. At the same time, the construction transaction needs to be based on The lock` statement or the `unlock` statement is used to transform. For the transaction structure, please refer to the [savings contract trading model] (https://github.com/Bytom/Bytom-Dapp-Demo/tree/master/src/components/layout/save) and [cash contract trading model] (https: //front-end source code for github.com/Bytom/Bytom-Dapp-Demo/tree/master/src/components/layout/profit).

- The transaction `input` structure is as follows:

`spendUTXOAction(utxohash)` indicates the contract `utxo`, where `utxohash` represents the `hash` of the contract `UTXO`, and `spendWalletAction(amount, Constant.assetDeposited)` represents the amount of savings or withdrawals entered by the user (only Contains the contract that requires asset exchange), and the asset type is fixed by the front end.

```ecmascript 6
    export function spendUTXOAction(utxohash){
        return {
            "type": "spend_utxo",
            "output_id": utxohash
        }
    }

    export function spendWalletAction(amount, asset){
        return {
            "amount": amount,
            "asset": asset,
            "type": "spend_wallet"
        }
    }

    const input = []
    input.push(spendUTXOAction(utxohash))
    input.push(spendWalletAction(amount, Constant.assetDeposited))
    ```


- The transaction `output` structure is as follows:

According to the `if-else` decision logic in the contract, the following is the construction model of the `output` of the savings dividend contract.

```ecmascript 6
    export function controlProgramAction(amount, asset, program){
        return {
            "amount": amount,
            "asset": asset,
            "control_program": program,
            "type": "control_program"
        }
    }

    export function controlAddressAction(amount, asset, address){
        return {
            "amount": amount,
            "asset": asset,
            "address": address,
            "type": "control_address"
        }
    }

    const output = []
    if(amountDeposited < billAmount){
        output.push(controlProgramAction(amountDeposited, Constant.assetDeposited, Constant.profitProgram))
        output.push(controlAddressAction(amountDeposited, billAsset, saver))
        output.push(controlProgramAction((billAmount-amountDeposited), billAsset, Constant.depositProgram))
    }else{
        output.push(controlProgramAction(amountDeposited, Constant.assetDeposited, Constant.profitProgram))
        output.push(controlAddressAction(billAmount, billAsset, saver))
    }
    ```

- 5) Start the front-end service
  
    Compile the front-end commands as follows:
    
    ```sh
    npm run build
    ```

    Before starting, you need to start the `bufferserver` buffer server, and then start the front-end service. The front-end startup commands are as follows:

    ```sh
    npm start
    ```

### DAPP buffer server
  
The buffer server is mainly for the efficiency of the management contract `UTXO` level, including how to synchronize the request to the `bycoin` server, in addition to the related transaction records of `DAPP`. For details, please refer to [`bufferserver` source code] (https://github.com/oysheng/bufferserver).

- 1) The structure of the savings dividend contract is as follows:

- Buffer server composition, currently designed `3` data table: `base`, `utxo` and `balance` table. The `base` table is used to initialize the contract `program` that the `DAPP` focuses on, that is, when looking for the `utxo` collection, just filter out the corresponding `program` and assets; the `utxo` table is The `utxo` collection of the `DAPP` contract, whose data is synchronized from the `bycoin` server in real time, is mainly to improve the concurrency of `DAPP`; the `balance` table is for recording the list of transactions in which the user participates in the contract.

- The backend service consists of an `API` process and a synchronous process, where the `API` service process is used to manage external user requests, and the synchronization process contains two aspects: one is to synchronize `utxo` from the `bycoin` server, another One is to query the transaction status through the blockchain browser.

  - The project administrator calls the `update-base` interface to update the contract `program` and `asset` that the `DAPP` focuses on. The `utxo` synchronization process periodically scans and updates the information in the local `utxo` table according to the records in the `base` table, and periodically unlocks the locked `utxo` according to the timeout period.

  - Whether the user needs to query the contract's `utxo` before calling save or cashout, the available `utxo` collection contains unconfirmed `utxo`. When the user clicks on the deposit or cashout button at the front end, it will call the `utxo` optimal matching algorithm to select the best `utxo`, then call the `update-utxo` interface to lock the `utxo`, and finally the user can Create a transaction, sign a transaction, and submit a transaction by calling the build transaction interface of the `bycoin` server through the plugin wallet. If all contracts `utxo` are locked, the lock time of the first `utxo` will be shortened to `60s`, which is set to ensure that unconfirmed transactions are successfully verified and unconfirmed `utxo` is generated. . If the time interval does not produce a new `utxo`, then the previous user is not considered to have generated a transaction, then the `utxo` can be spent again after `60s`.

- After the user sends the transaction successfully, two `balance` records will be generated. The default status is Failed. The transaction ID is used to query the blockchain browser for the transaction status. If the transaction is successful, the transaction status of `balance` will be updated. In addition, the `balance` list table of the front page only shows the record of the transaction success.


- 2) Compile `bufferserver` source code

    Follow the [`README`] (https://github.com/oysheng/bufferserver/blob/master/README.md) package to install the packages `Mysql` and `Redis` required by the deployment service, then download the source code and compile:

    ```sh
    make all
    ```
    
    After the compilation is complete, the executable files `api` and `updater` are generated in the `target` directory.

- 3) Start the service
    
    Use the `root` user to create the database and data table with the following commands:

    ```sh
    mysql -u root -p < database/dump.sql
    ```

    Modify the configuration file `config_local.json`. The field description refers to the detailed description of the `config` configuration parameter of [`README`] (https://github.com/oysheng/bufferserver/blob/master/README.md).


    Start the `api` and `updater` servers,     where `api` is the service process that provides the `JSON RPC` request, and `updater` is the service process that provides the synchronization `blockcenter` and blockchain browser data requests.

     ```sh
    ./target/api config_local.json

    ./target/updater config_local.json
    ```