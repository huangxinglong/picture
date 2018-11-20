## Bytom快速同步到当前块的高度

当我们在测试环境开发的时候，可能会遇到同步完所有区块需要很长时间。那我们何如能快速同步，方便我们快速开发呢？今天我就教大家通过修改Bytom源码来完成快速同步，下面跟着我一起同步吧！


###step1: 获取当前块的hash
 主网： <https://blockmeta.com/api/v2/latest-block>
 
 测试：<https://blockmeta.com/api/wisdom/latest-block>
 
 返回的hash:
 
![avatar](https://raw.githubusercontent.com/huangxinglong/picture/master/201811/09/图片%201.png)

###step2: 获取源码

要求：go1.8或者更高，并$GOPATH设置为首选目录

安装：确保go支持的版本已经正确安装

    $ go version	
    $ go env GOPATH GOPATH
    
   
获取源码：

     $ git clone https://github.com/Bytom/bytom.git $GOPATH/src/github.com/bytom

###step3: 修改源码

主网：

![avatar](https://raw.githubusercontent.com/huangxinglong/picture/master/201811/09/图片%202.png)

下载好源码后找到bytom/consensus 下的general.go 文件中声明主网参数设置的变量 MainNetParams，在Checkpoints的最后按照之前的格式加上当前块的高度，将第一步块的hash每两位进行分割，每两位前面加上0x。

测试：
![avatar](https://raw.githubusercontent.com/huangxinglong/picture/master/201811/09/图片%203.png)

如果是测试网，找到MainNetParams下面的变量TestNetParams,具体操作参考主网

###step4: 编译运行
参考链接：<https://github.com/Bytom/bytom>
 
###step5: 验证
1.当看到终端（或cmd）出块日志一秒100左右的出块就算操作成功
2.打开本地钱包浏览器，很快就能发现如下截图：

![avatar](https://raw.githubusercontent.com/huangxinglong/picture/master/201811/09/picture4.png)



