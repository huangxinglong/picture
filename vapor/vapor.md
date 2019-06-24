# vapor跨链交易和投票


比原链bysatck开放平台底层依托的侧链测试网落今天正式上线，该链是bytom的侧链(vapor)，它的诞生主要为了提升Bystack的效率以及服务于垂直领域的应用。它全新的混合共识算法和高性能成为他最大的亮点和核心优势。目前基本的开发工作已经完成可以正式对外公测，广大社区开发者和爱好者可以体验跨链交易，共识投票等！


下面我们来看一下如何是用跨链交易和进行侧链的共识投票！

### 第一步: 首先是搭建侧链的测试节点

#### 源码搭建

1. golang语言环境安装，参考： <http://www.runoob.com/go/go-environment.html>。
2. github地址：<https://github.com/Bytom/vapor>,然后参考readme.md文件进行搭建侧链节点.

#### 安装包搭建(这里以苹果电脑为例，window和linux类似)

1. 下载安装包进行安装,下载链接：<https://github.com/Bytom/vapor/releases>；

    ![](https://raw.githubusercontent.com/huangxinglong/picture/master/vapor/5.jpg)

    下载完以后进行解压，如下：
    ![](https://raw.githubusercontent.com/huangxinglong/picture/master/vapor/7.jpg)

2. 打开terminal(如果找不到，在搜索框中搜索)，然后输入命令：

    ./vapord-darwin_amd64 init --chain_id=vapor --home $HOME/bytom/vapor
    ./vapord-darwin_amd64  node --home $HOME/bytom/vapor

    示例如下图(命令直接复制上面两条))：

    ![](https://raw.githubusercontent.com/huangxinglong/picture/master/vapor/8.jpg)


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

![](https://raw.githubusercontent.com/huangxinglong/picture/master/vapor/6.jpg)

2. 为你的小伙伴投票：

![](https://raw.githubusercontent.com/huangxinglong/picture/master/vapor/9.jpg)



