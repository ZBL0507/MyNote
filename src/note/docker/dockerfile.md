# Dockerfile

### 官方文档：https://docs.docker.com/engine/reference/builder/

## Dockerfile 是什么
Docker can build images automatically by reading the instructions from a ```Dockerfile```. 
A ```Dockerfile``` is a text document that contains all the commands a user could call on the command line to assemble an image. 
Using ```docker build``` users can create an automated build that executes several command-line instructions in succession.

## Dockerfile 基础知识
1. 每条保留字指令都必须为大写字母且后面要跟随至少一个参数
2. 指令按照从上到下，顺序执行
3. \# 表示注释
4. 每条指令都会创建一个新的镜像层并对镜像进行提交

## Docker 执行 Dockerfile 的大致流程
1. docker 从基础镜像运行一个容器
2. 执行一条指令并对容器做出修改
3. 执行类似 docker commit 的操作提交一个新的镜像层
4. docker 再基于刚提交的镜像运行一个新容器
5. 执行 dockerfile 中的下一条指令直到所有指令都执行完成

## docker 常用保留字
1. FROM 
   ```text
   基础镜像，当前镜像是基于哪个镜像的，指定一个已经存在的镜像作为模版，第一条必须是FROM
   ```
2. MAINTAINER
   ```text
   镜像维护者的姓名和邮箱地址
   ```
3. RUN
   ```text
   容器构建时需要运行的命令
   
   RUN 是在 docker build 时运行
   
   两种格式：
      1. shell格式
            等同于在终端操作的shell命令
      2. exec格式
            # RUN ["./test.php", "dev", "offline"] 等价于 RUN ./test.php dev offline
   ```
   
4. EXPOSE 
   ```text
   当前容器对外暴露出的端口
   ``` 
   
5. WORKDIR
   ```text
   指定在创建容器后，终端默认登录的进来工作目录，一个落脚点
   ```
   
6. USER
   ```text
   指定该镜像以什么样的用户去执行，如果都不指定，默认是root
   ```
   
7. ENV
   ```text
   设置环境变量
   ```
   
8. VOLUME
   ```text
   容器数据卷，用于数据保存和持久化工作
   ```
   
9. ADD
   ```text
   将宿主机目录下的文件拷贝进镜像且会自动处理URL和解压tar压缩包
   ```
   
10. COPY
   ```text
   类似ADD，拷贝文件和目录到镜像中。

   将从构建上下文目录中<源路径>的文件/目录复制到新到一层到镜像内的<目标路径>位置
   ```

11. CMD
   ```text
   指定容器启动后要干的事情

   Dockerfile中可以有多个CMD指令，但只有最后一个生效，CMD会被 docker run 之后的参数替换

   CMD 是在 docker run 时运行
   RUN 是在 docker build 时运行
   ```

12. ENTRYPOINT
   ```text
   也是用来指定一个容器启动时要运行的命令

   类似于CMD指令，但是ENTRYPOINT不会被docker run后面的命令覆盖，而且这些命令行参数会被当作参数送给ENTRYPOINT指令指定的程序
   ```


## 自定义镜像 mycentosjava8
1. 要求： centos 镜像具备 vim + ifconfig + jdk8
2. 编写Dockerfile
   ```dockerfile
   FROM centos
   MAINTAINER ZBL<ZBL@qq.com>

   ENV MYPATH /usr/local
   WORKDIR $MYPATH

   #安装vim编译器
   RUN yum -y install vim
   #安装ifconfig命令查看网络ip
   RUN yum -y install net-tools
   #安装Java8以及lib库
   RUN yum -y install glibc.i686
   RUN mkdir /usr/local/java
   #ADD是相对路径jar，把jdk-8u171-linux-x64.tar.gz添加到容器中，安装包必须要和Dockerfile文件在同一位置
   ADD jdk-8u171-linux-x64.tar.gz /usr/local/java/
   # 配置java环境变量
   ENV JAVA_HOME /usr/local/java/jdk1.8.0_171
   ENV JRE_HOME $JAVA_HOME/jre
   ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib:$CLASSPATH
   ENV PATH $JAVA_HOME/bin:$PATH

   EXPOSE 80
   
   CMD echo $MYPATH
   CMD echo "success---------->ok!"
   CMD /bin/bash


   ```

3. 构建镜像
   ```text
   docker build -t 新镜像名称:TAG . 
   
   注意，上面TAG后面有个空格，有个点
   ```



