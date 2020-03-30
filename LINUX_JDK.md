linux下(ubuntu) JDK环境搭建

- 1. 准备好linux环境下的jdk包，并拷贝到linux服务器中
- 2. 使用tar命令解压

> tar -zxvf jdk-xxx-xxx-xxx.tar.gz

- 3. 编辑/etc/profile，在文件末尾加入环境变量

```text
JAVA_HOME=[JDK目录]
CLASSPATH=$JAVA_HOME/lib/
PATH=$PATH:$JAVA_HOME/bin
export JAVA_HOME CLASSPATH PATH
```
- 4. 刷新环境变量配置 

> source /etc/profile

- 5. 使用java -version查看是否安装成功