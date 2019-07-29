#  钱包交易所等第三方集成投票

#### 概述

比原的主侧链采用类似 blockstream ElementsProject/Liquid 中 federation 联邦多签发起跨链交易的方式。
federation 收集/监听打入联邦多签地址的交易，然后发起跨链操作。
ElementsProject/Liquid 中还会对 转入侧链交易 进行主链 spv 验证，以确保 该交易 已在主链上真实发生。
同时为了防止回滚（故障或者作弊），可以配置 federation 等待多少次确认之后 才把 主链资产转入侧链/侧链资产转回主链。
侧链如果没人维护、运行，又或者需要硬分叉升级，federation 清算并将资产转回主链。


#### 一.锁资产

##### 主链转侧链

主链资产转侧链：<https://github.com/Bytom/vapor/wiki/API-Doc#mainchainbytom-to-sidechainvapor>


##### 侧链转主链

侧链资产转主链：<https://github.com/Bytom/vapor/wiki/API-Doc#sidechainvapor-to-mainchainbytom>


##### 1. 将资产打到官方的federaion地址，官方会根据转账地址在侧链创建对应的私钥，用户将主链的钱包信息（助记词，备份文件）导入侧链钱包就可以看到自己的账户余额，且可以进行投票

#### 提示：有了侧链的账户和资产，就可以调用接口进行投票等其他操作；


#### 二. 投票

##### 第一步: 调用build-transaction api构建投票交易：

    请求参数：
    {
     "actions": [
    {
      "amount": 100000000 // 投票数
      "type": "spend_account",
      "account_alias": "xxx",
      "asset_id": "ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff",
    },
    {
      "asset_id": "ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff",
      "amount": 3000000, // 手续费
      "type": "spend_account",
      "account_alias": "xxx",
    },
    {
      "amount": 100000000, // 投票数
      "asset_id": "ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff",
      "address": "vp1qdmcx4lculgvzf5sqxwsxyuewc669eatme3pws5",
      "vote": "964909579fac3b527ed48cf037142737799ff6f37b38860f8537e9d761b95a1e4ac209a13b766be5f43a930160dad9355ddcca7db965819767629aff571953bd", //需要投票的公钥
      "type": "vote_output",
    }
    ],
    "allow_additional_actions": false,
    "base_transaction": null
    }

##### 第二步: 调用sign-transaction api对build完成后的交易进行签名


##### 第三步：签名完成后调用submit-transaction api提交交易

##### 备注：第二步和第三步跟普通的交易签名，提交完全一致。参考vapor api文档，api文档地址：<https://github.com/Bytom/vapor/wiki/API-Doc>。

#### 三. 取消投票

##### 1. 调用build-transaction api构建取消投票交易， 请求参数如下：

    {
    "actions": [
    {
      "asset_id": "ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff",
      "amount": 1100000,  //手续费
      "type": "spend_account",
      "account_alias": "xxx",
      "account_id": "",
    },
    {
      "amount": 100000000, // 取消投票的数量
      "asset_id": "ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff",
      "vote": "964909579fac3b527ed48cf037142737799ff6f37b38860f8537e9d761b95a1e4ac209a13b766be5f43a930160dad9355ddcca7db965819767629aff571953bd", // 取消投票对应的公钥
      "type": "veto",
      "account_alias": "xxx",
      "account_id": "",
    },
    {
      "amount": 100000000, // 取消投票的数量
      "address": "vp1qqfr7gvju3s66hrfz225a5s0hyvw8hturfp3hlk",
      "type": "control_address",
      "asset_id": "ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff",
    }
    ],
    "allow_additional_actions": false,
    "base_transaction": null
    }
    
##### 第二步: 调用sign-transaction api对build完成后的交易进行签名

##### 第三步：签名完成后调用submit-transaction api提交交易

##### 备注：第二步和第三步跟普通的交易签名，提交完全一致。参考vapor api文档,api文档地址：<https://github.com/Bytom/vapor/wiki/API-Doc>。

如果有什么问题，可以在社群里面进行提问。



