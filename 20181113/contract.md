##比原合约预编译



各位技术社区的小伙伴们，大家好。在开发合约的过程中你们有没有遇到一些问题呢？比如编译合约的过程中不能实时动态的去检查我们所编译的合约文件是否正确，那么我今天就教大家一种很方便的方法。可以让小伙伴们在编写合约的过程中，可以随时检查自己的合约编写是否正确。

首先要确保我们有go语言开发环境且版本高于1.8，如果没有搭建go语言开发环境，请自行百度。确保go支持的版本已经正确安装：

    $ go version
    $ go env GOROOT GOPATH
         
 获取源代码并编译，参考链接：<https://github.com/Bytom/equity>
 
 编译完了以后我们可以在equity下执行:
 
     ./equity/equity --help
    
  获取合约的命令帮助。返回的截图如下：
     
 ![avatar](https://raw.githubusercontent.com/huangxinglong/picture/master/20181113/帮助命令.png)
 
图中标的1，2，3，4 分别表示执行命令所带参数的含义。然后在项目下面创建一个合约文件(合约文件最好不带任何后缀名)，如下图：

 ![avatar](https://raw.githubusercontent.com/huangxinglong/picture/master/20181113/创建文件.png)
 
然后编写合约，我是用vim编译的合约，大家可以自行选择用vim或者编辑器编写合约。如果编译合约的过程中存在问题，请参考合约开发文档：<https://bytom.github.io/mydoc_RPC_call.cn.html>。下图是我在vim中编写的合约。

 
  ![avatar](https://raw.githubusercontent.com/huangxinglong/picture/master/20181113/创建合约.png)
  
  
  
   合约编写完了以后，如果合约编写错误或者存在语法错误，会出现如下图所示的情况，请检查自己编写的合约
   
   
  ![avatar](https://raw.githubusercontent.com/huangxinglong/picture/master/20181113/合约错误.png)
  
 检查无误以后，在对应的目录下面执行合约文件，然后就可以输出下图所示的二进制。说明合约编写成功
  
  
   ![avatar](https://github.com/huangxinglong/picture/raw/master/20181113/编译合约.png)
 
 
 大家有没有发现很简单呢？快点实践起来吧！如果在开发的过程中遇到问题，请在我们的社区联系我们：<https://github.com/Bytom/>
 
        
 
