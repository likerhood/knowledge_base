

### win10系统下inno setup 打包程序因权限不足无法执行问题解决方案

因电脑更新，今年单位统一采购的电脑配置的是win10 系统（神州网信政府版）这个系统很多功能被隐藏或没有打开，所以用着很不习惯，也折腾了很久。近期使用python开发了一个内网自动监测和数据下载的程序，通过pyinstaller打包后，使用inno setup 编译器再封装为安装包。 以为万事大吉，但安装后执行程序时，居然没有反映，经查找，是由于win10系统权限管理非常严格，因为我写的程序在运行时会自动在程序目录中创建一个“temp”文件夹，用于下载数据时文件格式转换过程中临时数据的存放，win10系统中，C盘的数据如果你要进行修改或删除，每次都会提示要有管理员的权限（就是要能读、写的权限），我试着将安装目录所有用户增加完全控制的权限后，程序正常执行。但每次都要通过手工增加这个权限也是非常麻烦的，因此上网找了好多资料，都是说要增加管理员权限，即让打包后的程序使用管理员身份运行。经过测试，有两个解决方案，在这边记录一下，省得每次都要到处找资料。

### 解决方案1：修改inno setup生成的默认编译代码。

启动inno setup 进行编译时，输入相关编译基础信息后，会根据输入的相关内容自动生成编译脚本，在脚本中加入以下2行脚本内容： [Dirs] Name: {app}; Permissions: users-full 因为默认生成的脚本是没有[Dirs]这个字段的，增加这两行脚本后，打包的程序的安装目录拥有完全控制的权限。

### 解决方案2：使用Resource Hacker 修改 INNO安装目录下的SetupLdr.e32文件中的Manifest内容

修改SetupLdr.e32第24行内容： 原文：<requestedExecutionLevel level="asInvoker" uiAccess="false"/> 修改为：<requestedExecutionLevel level="requireAdministrator" uiAccess="false"/> 注意编译后保存！！！ 文中用到的Resource Hacker自行百度搜索下载 。

方案2参考：CSDN博主「UnkownState」文章 原文链接： [解决inno setup打包，执行权限导致无法执行问题_inno编译后的安装包无法打开-CSDN博客](https://blog.csdn.net/andrew57/article/details/51315939) 



***

**声明：**本文转载自 [win10系统下inno setup 打包程序因权限不足无法执行问题解决方案_inno 打包文件 ,安装包文件夹权限不足-CSDN博客](https://blog.csdn.net/wubingliang79/article/details/119981727)  ，版权归原作者所有。如有侵权，请联系删除。 

