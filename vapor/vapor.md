# vapor主网启动流程




Bystack主网启动流程如下：

## 步骤一：技术操作

### 第一步: 首先是搭建侧链的测试节点

#### 源码搭建

1. golang语言环境安装，参考： <http://www.runoob.com/go/go-environment.html>。
2. github地址：<https://github.com/Bytom/vapor>,然后参考readme.md文件进行搭建侧链节点.
3. 命令：

    make install

    vapord init --chain_id=testnet
    
    vapord node

#### 安装包搭建(这里以苹果电脑为例，window和linux类似)

1. 下载安装包进行安装,下载链接：<https://github.com/Bytom/vapor/releases>；

    ![](https://raw.githubusercontent.com/huangxinglong/picture/master/vapor/5.png)

    下载完以后进行解压，如下：
    ![](https://raw.githubusercontent.com/huangxinglong/picture/master/vapor/7.jpg)

2. 打开terminal(如果找不到，在搜索框中搜索)，然后输入命令：

    ./vapord-darwin_amd64 init --chain_id=testnet --home $HOME/bytom/vapor

    ./vapord-darwin_amd64  node --home $HOME/bytom/vapor

    示例如下图(命令直接复制上面两条))：

    ![](https://raw.githubusercontent.com/huangxinglong/picture/master/vapor/8.png)
    


### 第二步：

搭建好侧链的测试节点以后，在本地打开：<http://127.0.0.1:9889/dashboard>

创建账户：


![](https://raw.githubusercontent.com/huangxinglong/picture/master/vapor/1.jpg)

创建钱包地址：

![](https://raw.githubusercontent.com/huangxinglong/picture/master/vapor/3.jpg)

然后添加微信，发送地址到群主可以领取BTM。领取到BTM以后在交易模块进行新建交易，然后进行跨链转账和投票。

![](https://raw.githubusercontent.com/huangxinglong/picture/master/vapor/2.jpg)




### 第三步

1. 分享自己的账户公钥给为你投票的人，然后让别人为你投票：

![](https://raw.githubusercontent.com/huangxinglong/picture/master/vapor/6.png)

2. 给共识节点投票：

![](https://raw.githubusercontent.com/huangxinglong/picture/master/vapor/9.png)

3. 开启挖矿(开启以后节点就可以正常出块)

![](https://raw.githubusercontent.com/huangxinglong/picture/master/vapor/10.png)


![](https://raw.githubusercontent.com/huangxinglong/picture/master/vapor/11.png)



## 步骤二：反馈节点信息给比原官方

1. 通过dashboard查看自己的节点公钥并与比原官方沟通，更新投票列表。

 
## 步骤三：通过手机进行投票

1. 通过Bycoin手机钱包将比原跨到Vapor上（建议使用Bycoin）

2. 为自己投入100W的比原，社区拉票


##  步骤四：收益分红

1. 搭建使用cli工具对支持者进行收益分红

