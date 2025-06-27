# 关于idea2023创建maven项目没有src的解决办法

[toc]



创建maven没有src，可以通过添加参数来解决，也可以直接创建

### 1.添加参数：


![img](img/2c007a4d37c1910c3966668186a7838d.png)

在file-&gt;new projects setup-&gt;Settings for New Projects...


![img](img/ca8c5db59292b7ed6d3859403acb5719.png)

配置Maven的Runner的VM Options为-Darchetype=Internal

在下次创建Maven项目时会进行下载，等待下载完成，大约一两分钟左右，项目就会出现src目录了。

### 2.直接创建

新建maven项目，发现没有src/main/java


![img](img/83386d9574e7ba8bc6fc6c111d77ccbf.png)

直接新建文件夹：右击项目名-&gt;new-&gt;Directory


![img](img/802e19213346d00501b9f61ff6f9df4a.png)

&nbsp;可以看到idea给出了快捷创建文件夹的选项，可以根据需要创建，这里点击src/main/java


![img](img/bafd40f35aacf3168f63601ccc4a46ce.png)

回车，可以看到文件夹已经创建成功了


![img](img/608e1c7e6d1f499086bdd26f15d22d3d.png)

