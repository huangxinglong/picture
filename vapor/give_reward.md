# 节点发放工具使用方法


1.获取vapor的源码，vapor源码地址：<https://github.com/Bytom/vapor>

2.切换到cmd/votereward 目录

3.本地安装mysql数据库

4.在cmd/votereward下创建reward.json 文件，如下：

    {
     "node_ip": "http://127.0.0.1:9889", //节点地址
     "chain_id": "mainnet",    //节点网络id(不用变动)
     "mysql": { //mysql连接信息
       "connection": {
          "host": "192.168.30.186",
          "port": 3306,
          "username": "root",
          "password": "123456",
          "database": "reward"
      },
      "log_mode": false  // 默认不用管
    },
    "reward_config": {
      "xpub": "9742a39a0bcfb5b7ac8f56f1894fbb694b53ebf58f9a032c36cc22d57a06e49e94ff7199063fb7a78190624fa3530f611404b56fc9af91dcaf4639614512cb64",  //节点公钥(从dashboard上设置获取)
     "account_id": "bd775113-49e0-4678-94bf-2b853f1afe80", //账户id
     "password": "123456",  //密码
     "reward_ratio": 20,   //分给投票人的获奖比例
     "mining_address": "vp1q7muj4wufgpk547rxeaajxelssayg8sfwf4xydw"    //挖矿地址，用get-mining-address 获取挖矿地址，例如：curl -X POST http://127.0.0.1:9889/get-mining-address 获取
    }
    } 
    
5. 执行：6000是指区块的起始位置，7200是发收益截止区块的出块位置，奖励

        ./votereward reward --reward_start_height 6000 --reward_end_height 7200
        
        
6000是指区块的起始位置，7200是发收益截止区块的出块位置，将会发放出块在6000~7200这一段的奖励。每次发放奖励完了以后需要记住自己的截止区块，下一次会变成起始位置，然后截止区块加1200的整数倍。
  
  
  注意：可能需要自己合并utxo





