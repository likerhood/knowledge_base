# 解决Linux运行shell脚本提示No such file or directory错误提示

[toc]



## 场景复现


在Linux上面进行重启springboot项目时，手动在linux上面创建了sh文件，进行运行提示：【No such file or directory】 ![img](img/851d476cf4f9442890e72550455bcc62.png)

## 解决

在其他服务器中写好shell脚本测试正常，但只是复制文字到另一服务器上以脚本方式运行命令时提示No such file or directory错误，那么一般是文件格式是dos格式的缘故，改成unix 格式即可。

附上亲测可用方法：




1、用vim打开该sh文件：【vi 文件名】 ![img](img/99431f8fb86c9bdce7e2d1a01d7159aa.png) 2、输入 :set ff 回车，显示fileformat=dos，需要改变格式 ![img](img/6fd6cbf91a68a31241590727f8f322a1.png) 3、重新设置下文件格式，继续开始输入英文冒号 :set ff=unix ![img](img/3b7fecd2fffaa58bea20599f1d2c57d8.png)


4、保存退出（继续冒号输入） :wq 回车 ![img](img/162a86a823a4e09de31b451d2ddde7d4.png)


5、到这一步，可以查看是否改变格式：vi 文件名.sh --&gt; :set ff 就看看到修改成功 ![img](img/93d3427ace4768f1582bbbd3517a5621.png)

## 附上linux重启jar项目方法

只需修改process里面自己的jar名称即可，jar在lib下面才行，不然自己改重启脚本或者重写吧~

```java
#!/bin/bash
source /etc/profile
process=$1
process='自己jar名称.jar'
pid=$(jps -l|grep $process |awk '{print $1}')
DIR="$( cd "$( dirname "$0"  )" && pwd  )"
parent_dir=`dirname "${DIR}"`
echo $parent_dir
#判断进程是否已经存在
if [ -z "$pid" ];
  then
    echo "pid is empty"
    echo "现在正在启动"
    nohup java  -server -Xms256m -Xmx256m -Xverify:none -XX:+DisableExplicitGC -Djava.awt.headless=true -jar $parent_dir/lib/$process > /dev/null 2>&1 &
else
    echo "pid is not empty,$pid will be shutdown"
    echo "Killing $process"
    echo "------kill--- $pid ----"
    kill -9 $pid
    echo "------ kill success -------"
    echo "The $process will be start after 3 seconds"
    sleep 1
    echo "The $process will be start after 2 seconds"
    sleep 1
    echo "The $process will be start after 1 second"
    sleep 1
    nohup java -server -Xms256m -Xmx256m -Xverify:none -XX:+DisableExplicitGC -Djava.awt.headless=true -jar $parent_dir/lib/$process > /dev/null 2>&1 &
fi
```




