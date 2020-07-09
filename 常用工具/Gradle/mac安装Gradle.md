## 1-下载

下载地址:https://gradle.org/install/

## 2-设置环境变量

Gradle可在所有主要操作系统上运行，并且仅需要安装[Java JDK或JRE](http://www.oracle.com/technetwork/java/javase/downloads/index.html)版本8或更高版本。要检查，请运行`java -version`：

![image-20200619153906818](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200619153906818.png)

打开终端

vi ~/.base_profile   **(注：找到自己配置环境变量的文件)**

加入

```
GRADLE_HOME=/Users/caoliang/Project/Gradle/gradle-6.5
export GRADLE_HOME
export PATH=$PATH:$GRADLE_HOME/bin
```

/Users/caoliang/Project/Gradle/gradle-6.5  这里是你下载后解压的位置

:wq一下  

不行就  :wq!  这里是强制执行

再不行就 sudo su  切换到炒鸡管理员模式

保存之后使配置文件生效source ~/.base_profile



## 3-完成

验证一下  

gradle -v

![image-20200619154156526](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200619154156526.png)