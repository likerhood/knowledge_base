# IDEA右键新建时没有Java Class选项-解决办法

[toc]

今天遇到一个比较恶心的情况，就是想要新建Java Class文件，在右击new后面的选项里找不到Java Class这一项。然后经过查询才知道怎么回事，在这里也跟大家普及一下。

## 1、问题描述


右键new没有javaclass选项，而是下图这个页面: ![img](img/af4271b94dadeb2ef7c77cca89674654.png)

## 2、解决办法1（很简单）

自己建的包名为“关键字”，比如说我刚才建的包是 abstract，是关键字，不行。 只需要删了包，重建，再包后边价格下划线即可。例如 abstract_

右键new就有了。问题解决。

## 3、解决办法2（专业）

按步骤点


1. File–Project Structure 
2. 按以下步骤点。 ![img](img/ab9b815a2008ea14a37f6021ec549f9e.png) 
3. 右键new，就有JavaClass了


![img](img/4291a82387467dcac237de920dc8707a.png)

## 4、问题的原因（原理）


其实按以上步骤问题已经解决了，但是问题的原因是什么呢？和设置的原理是什么？接下来就是具体讨论。 ![img](img/0b4ec2eb5711e6c6734fd2ba122854ac.png)

如上图红圈所示，我们可以根据对项目的任意目录进行这五种目录类型标注，这个知识点非常非常重要，必须会。

- Sources： 一般用于标注类似 src 这种可编译目录。有时候我们不单单项目的 src 目录要可编译，还有其他一些特别的目录也许我们也要作为可编译的目录，就需要对该目录进行此标注。只有 Sources 这种可编译目录才可以新建 Java 类和包，这一点需要牢记。 
- Tests： 一般用于标注可编译的单元测试目录。在规范的 maven 项目结构中，顶级目录是 src，maven 的 src 我们是不会设置为 Sources 的，而是在其子目录 main 目录下的 java 目录，我们会设置为 Sources。而单元测试的目录是 src - test - java，这里的 java 目录我们就会设置为 Tests，表示该目录是作为可编译的单元测试目录。一般这个和后面几个我们都是在 maven 项目下进行配置的，但是我这里还是会先说说。从这一点我们也可以看出 IntelliJ IDEA 对 maven 项目的支持是比彻底的。 
- Resources： 一般用于标注资源文件目录。在 maven 项目下，资源目录是单独划分出来的，其目录为：src - main -resources，这里的 resources 目录我们就会设置为 Resources，表示该目录是作为资源目录。资源目录下的文件是会被编译到输出目录下的。 
- Test Resources： 一般用于标注单元测试的资源文件目录。在 maven 项目下，单元测试的资源目录是单独划分出来的，其目录为：src - test -resources，这里的 resources 目录我们就会设置为 Test Resources，表示该目录是作为单元测试的资源目录。资源目录下的文件是会被编译到输出目录下的。 
- Excluded： 一般用于标注排除目录。被排除的目录不会被 IntelliJ IDEA 创建索引，相当于被 IntelliJ IDEA 废弃，该目录下的代码文件是不具备代码检查和智能提示等常规代码功能。 通过上面的介绍，我们知道对于非 maven 项目我们只要会设置 src 即可。

