创建项目镜像
========

创建项目镜像就是针对这个项目写好一份Dockerfile，安装程序运行所需要的**所有依赖**，最后通过`docker build --pull -t some_name:some_tag .`命令构建打包成为一个image。最终项目源码、依赖都存储在image中，项目的依赖有了绝对的保证。


课程格子的以Ruby项目居多，针对一般的Ruby项目，会有如下的Dockerfile。其他语言的项目原理一样，Dockerfile大同小异。

```bash
FROM ruby:2.3 # ruby的image的源image一般为Debain

# 设置时区
RUN ln -snf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo Asia/Shanghai > /etc/timezone

# 拷贝项目
ADD / /app/
WORKDIR /app

# 如果项目中存在私有依赖，需要配置ssh_key
RUN mkdir -p /root/.ssh && \
    echo "StrictHostKeyChecking no" > /root/.ssh/config && \
    chmod -R 700 /root/.ssh
ADD path/to/id_rsa /root/.ssh/

# 安装依赖，这里安装了production与test两个环境的依赖，则这个image可以用来运行测试，也可以用来线上部署
RUN bundle install --without development

# 处于安全考虑，可以在此移除 ssh_key
RUN rm /root/.ssh/id_rsa

# 这里CMD与ENTRYPOINT根据需要自行配置
CMD puma -p 80 -t 16:64
```

而复杂一些的项目，则可能需要提前安装一些系统库，根据image的系统，自行配置国内的源，来加速一些库的安装速度，以Debain 8为例

```bash
# 设置aliyun的debine源
RUN echo "deb http://mirrors.aliyun.com/debian/ jessie main non-free contrib\n\
deb http://mirrors.aliyun.com/debian/ jessie-proposed-updates main non-free contrib\n\
deb-src http://mirrors.aliyun.com/debian/ jessie main non-free contrib\n\
deb-src http://mirrors.aliyun.com/debian/ jessie-proposed-updates main non-free contrib"\
> /etc/apt/sources.list

# 安装系统依赖
RUN apt-get update && \
    apt-get install -y xxxx xxxx xxxx && \
    apt-get clean
```

更简单的方式是直接下载可运行的二进制程序，例如wkhtmltopdf，这样这个容易就可以轻易的使用wkhtmlto*命令导出pdf或图片了。一定要记得删除压缩的tar文件，这样可以使最终生成的docker image占用更少的磁盘空间

```bash
RUN wget https://downloads.wkhtmltopdf.org/0.12/0.12.4/wkhtmltox-0.12.4_linux-generic-amd64.tar.xz && \
    tar xvf wkhtmltox-0.12.4_linux-generic-amd64.tar.xz && \
    rm wkhtmltox-0.12.4_linux-generic-amd64.tar.xz

ENV PATH $PATH:/wkhtmltox/bin
```

### 问题出现

解决了项目的依赖，还没来得及高兴，就遇到了一个问题：一个项目构建Docker Image时间比较长。一个最简单的ruby项目，从头build一遍，也要花上6-10分钟左右。因为有下载源镜像、更新apt-get，安装gem等步骤，均比较耗时。

而且在我的计划中，项目的每个push到gitlab的commit都需要构建image并测试，那么这么慢的build image时间是不能接受的。

下一节我就针对国内的环境，说说为了提升build速度，我都做了哪些措施。

<!--##### 启动复杂
启动项目时需要先安装所有依赖，需要预编译一些代码，才能运行。第一次在新环境启动一个项目需要几分钟才行，对项目扩展并不友好。-->