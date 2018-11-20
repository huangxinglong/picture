## Bytom quickly syncs to the latest block

When we are developing in a test environment, it may take a long time to synchronize all the blocks. So how can we quickly synchronize and facilitate our rapid development? Today I will teach you to complete the fast synchronization by modifying the Bytom source. Let's synchronize with me!


###Step one: Get the hash of the current block
 Main network： <https://blockmeta.com/api/v2/latest-block>
 
 Test network： <https://blockmeta.com/api/wisdom/latest-block>
 
 Returned hash:
 
![avatar](https://raw.githubusercontent.com/huangxinglong/picture/master/201811/09/图片%201.png)

###Step two: Get the source code of Bytom

Requirements: go1.8 or higher, and $GOPATH is set as the preferred directory

Installation: Make sure the version supported by go is installed correctly

    $ go version	
    $ go env GOPATH GOPATH
    
   
Get the source code：

     $ git clone https://github.com/Bytom/bytom.git $GOPATH/src/github.com/bytom

###Step three: Modify source

Main network：

![avatar](https://raw.githubusercontent.com/huangxinglong/picture/master/201811/09/图片%202.png)

After downloading the source code, find the variable MainNetParams that declares the parameters of the main network in the general.go file under bytom/consensus. At the end of the Checkpoints, add the height of the current block according to the previous format, and perform the hash of the first block every two digits. Split, plus 0x before every two digits.

Test network：
![avatar](https://raw.githubusercontent.com/huangxinglong/picture/master/201811/09/图片%203.png)

If it is a test network, find the variable TestNetParams under MainNetParams, the specific operation refers to the main network.

###Step four: Compile and run

Reference link：<https://github.com/Bytom/bytom>
 
###Step five: Verification
1. When you see the terminal (or cmd) out of the block log for about 100 seconds, the operation is successful.
2. Open the local wallet browser and you will soon find the following screenshot:


![avatar](https://raw.githubusercontent.com/huangxinglong/picture/master/201811/09/picture4.png)



