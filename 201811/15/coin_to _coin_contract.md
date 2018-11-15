# 币币合约开发指南

上一篇文章介绍了方便大家动态编译合约的方法，今天我就带领大家来实际的开发合约。为了让大家更方便理解，就直接用postman请求。

###参考合约

    contract TradeOffer(assetRequested: Asset,
                    amountRequested: Amount,
                    seller: Program,
                    cancelKey: PublicKey) locks      
                    valueAmount of valueAsset {
    clause trade() {
    lock amountRequested of assetRequested with seller
    unlock valueAmount of valueAsset
    }
    clause cancel(sellerSig: Signature) {
    verify checkTxSig(cancelKey, sellerSig)
    unlock valueAmount of valueAsset
    }
    }
 
###锁定合约

####  第一步，编译合约：
我们以上一次创建的 TradeOffer合约文件为例，首先编译合约，判断我们的合约是否正确。

      ./equity/equity TradeOffer --bin
         
 返回如下：
 
 
  
 
##锁定合约
###第一步，调用
