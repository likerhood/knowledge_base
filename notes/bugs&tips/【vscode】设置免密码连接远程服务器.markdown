# vscode设置免密码连接远程服务器

[toc]




![img](img/82c498bd53c671029690b19669b1b8e4.png)

在 Visual Studio&nbsp; [Code](https://so.csdn.net/so/search?q=Code&spm=1001.2101.3001.7020)&nbsp;(VS Code) 中连接到远程服务器并实现免输密码的方法通常使用 SSH 密钥。

## 一、使用密码连接远程服务器

首先需要保证，在vscode里，可以使用密码连接到远程的服务器。

可以参考 [vscode远程连接服务器](https://zhuanlan.zhihu.com/p/671431475)



## 二、免密码连接远程服务器

>按照上述步骤配置好打开vscode进入服务器文件夹都要输入密码，下面的步骤展示如何免密进访问

 [打开cmd](https://so.csdn.net/so/search?q=%E6%89%93%E5%BC%80cmd&spm=1001.2101.3001.7020)终端，输入ssh可以看到我们的计算机已经存在ssh了，然后输入“**ssh-keygen**”命令，一直按回车跳过三个“Enter”，产生公私密钥，


![img](img/f60a62530ea5da700c27959ff1775746.png)


![img](img/907a17f9d754b66b096533c719846873.png)

此时进入此电脑，在文件搜索框中输入“**C:\Users\【用户名】\.ssh**”，访问“**id_rsa.pub**”文件用记事本打开并复制其中的内容，


![img](img/4123944eeccd991c8ac7e775da572d85.png)

打开vscode，首先登录远程服务器，并进入服务器文件夹，

找到远程服务器的 ~/.ssh/authorized_keys 文件中，找到“.ssh”文件下的“authorized_keys”并打开，将刚才生成的公钥pub文件复制的口令粘贴过来保存到这个文件里即可，

>如果“.ssh”文件下没有这个“authorized_keys”文件，可以新建一个。


![img](img/4009e43f8624b93daa05ece9ec6cae29.png)

关闭后再打开vscode，ssh远程连接服务器，就发现已经实现了免密访问。

