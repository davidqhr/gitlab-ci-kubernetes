加快build速度
===========

这一节的内容不是必须的，如果你认为build image的速度合理，可以接受，不会影响团队的开发、部署节奏，请直接跳过本节。

上一节提到，build一个image可能需要以下步骤。

1. 下载基础镜像(ruby:2.3)
2. 安装系统库
3. 安装语言库

接下来我就针对这些步骤，说说怎么加快build速度。

### 搭建私有registry，缓存常用的基础镜像
例如项目中常用到的基础镜像有

- ubuntu:14.04
- ruby:2.3
- elixir:1.3
- nodejs:7

当Dockerfile以这些镜像为基础的时候，执行`docker build`时，dockerd会去docker的官方registry下载基础镜像，在国内的网络环境条件下，速度慢不说，还经常被重置连接。

所以我搭建了一个私有registry来存储常用的image。搭建registry的细节，我将在后边的章节提到。先来看看，假如已经有了一个可用的私有registry以后，我们如何创建一个基础镜像的mirror。

```bash
# 从官方下载
docker pull ruby:2.3

# 打tag
docker tag ruby:2.3 registry.example.com:5555/ruby:2.3

# 登录私有registry
docker login registry.example.com:5555 -u username -p 

# 将基础镜像上传
docker push registry.example.com:5555/ruby:2.3
```

这样之后，在Dockerfile中，我们就可以这么写：

```bash
FROM registry.example.com:5555/ruby:2.3
RUN ...
RUN ...
```

把私有registry的机器与build image的机器放在同一个局域网内，那么下载一个镜像的速度就会很快，这样优化之后，通常可以在几十秒内搞定镜像下载的这个步骤。

只不过需要注意的是，需要确保build image的时候要有registry的权限。具体权限的操作细节，会在以后的gitlab ci章节中提到。

### 为项目创建创建特有基础镜像，缓存系统依赖

虽然上一步骤提到了缓存基础镜像，但是安装系统依赖的步骤依然很耗时间，比如`apt-get update && apt-get install -y lib-aaa lib-bbb`。这些依赖都是每个项目特有的，并且不会轻易变的。如果每次build的时候，都重复执行这些步骤，那耗时是肯定的。

解决办法和容易：**为项目构建一个特有的基础镜像**。

比如一个Elixir项目，使用phoenix作为框架，需要nodejs和elixir环境，并且需要使用wkhtmltopdf来导出内容，并且需要安装字体。最初项目的完整的Dockerfile如下：

```bash
FROM elixir:1.3

# 安装系统依赖、配置系统环境

ENV SASS_BINARY_SITE https://npm.taobao.org/mirrors/node-sass/
RUN ln -snf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo Asia/Shanghai > /etc/timezone

RUN wget https://npm.taobao.org/mirrors/node/v7.7.3/node-v7.7.3-linux-x64.tar.xz 2>&1 1> /dev/null && \
    tar -C /usr/local/ --strip-components 1 -xvf /node-v7.7.3-linux-x64.tar.xz 2>&1 1> /dev/null && \
    rm /node-v7.7.3-linux-x64.tar.xz

RUN mkdir -p /root/.ssh && \
    echo "StrictHostKeyChecking no" > /root/.ssh/config && \
    chmod -R 700 /root/.ssh

RUN wget https://downloads.wkhtmltopdf.org/0.12/0.12.4/wkhtmltox-0.12.4_linux-generic-amd64.tar.xz && \
    tar xvf wkhtmltox-0.12.4_linux-generic-amd64.tar.xz && \
    rm wkhtmltox-0.12.4_linux-generic-amd64.tar.xz

ENV PATH $PATH:/wkhtmltox/bin

COPY /fonts/* /usr/share/fonts/truetype/cn/
RUN fc-cache

# 项目相关

ADD / /app
WORKDIR /app

ENV MIX_ENV prod
ENV HEX_UNSAFE_REGISTRY 1

RUN mix local.hex --force && \
    mix hex.config mirror_url https://hexpm.upyun.com && \
    mix hex.config repo_url https://hexpm.upyun.com && \
    mix deps.get && \
    mix deps.compile && \
    mix compile

RUN npm install --registry=https://registry.npm.taobao.org --disturl=https://npm.taobao.org/dist

RUN node_modules/.bin/brunch build --production && \
    mix phoenix.digest
```

首先，我们先创建一个项目特有的基础镜像，命名为`registry.classbox.com/elixir-node-pdf:1.0`，Dockerfile如下

```
FROM elixir:1.3

# 安装系统依赖、配置系统环境

ENV SASS_BINARY_SITE https://npm.taobao.org/mirrors/node-sass/
RUN ln -snf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo Asia/Shanghai > /etc/timezone

RUN wget https://npm.taobao.org/mirrors/node/v7.7.3/node-v7.7.3-linux-x64.tar.xz 2>&1 1> /dev/null && \
    tar -C /usr/local/ --strip-components 1 -xvf /node-v7.7.3-linux-x64.tar.xz 2>&1 1> /dev/null && \
    rm /node-v7.7.3-linux-x64.tar.xz

RUN mkdir -p /root/.ssh && \
    echo "StrictHostKeyChecking no" > /root/.ssh/config && \
    chmod -R 700 /root/.ssh

RUN wget https://downloads.wkhtmltopdf.org/0.12/0.12.4/wkhtmltox-0.12.4_linux-generic-amd64.tar.xz && \
    tar xvf wkhtmltox-0.12.4_linux-generic-amd64.tar.xz && \
    rm wkhtmltox-0.12.4_linux-generic-amd64.tar.xz

ENV PATH $PATH:/wkhtmltox/bin

COPY /fonts/* /usr/share/fonts/truetype/cn/
RUN fc-cache
```

build 并且上传到私有仓库

```bash
docker build -t registry.classbox.com/elixir-node-pdf:1.0 .
docker login registry.classbox.com -u username -p
docker push registry.classbox.com/elixir-node-pdf:1.0
```

修改项目的Dockerfile为

```bash
FROM registry.classbox.com/elixir-node-pdf:1.0

ADD / /app
WORKDIR /app

ENV MIX_ENV prod
ENV HEX_UNSAFE_REGISTRY 1

RUN mix local.hex --force && \
    mix hex.config mirror_url https://hexpm.upyun.com && \
    mix hex.config repo_url https://hexpm.upyun.com && \
    mix deps.get && \
    mix deps.compile && \
    mix compile

RUN npm install --registry=https://registry.npm.taobao.org --disturl=https://npm.taobao.org/dist

RUN node_modules/.bin/brunch build --production && \
    mix phoenix.digest
```

这样一来，安装系统依赖的时间就全都省下来了，每次build时，下载的特有基础镜像就已经包含了所有的系统依赖。如果将来需要修改项目的系统依赖，只需要修改一下特有的基础镜像即可。

<!--这里大家可能有个疑问，既然系统依赖，可以通过提前构建好镜像，但是为什么项目的语言级别依赖库需要每次都重新安装，而不能也提前存好呢？这是因为：创建项目特有基础镜像一般是手工来完成的。语言级别的依赖更改的频率远高于系统依赖。-->

### 使用mirror，加速语言依赖的安装

其实在本节的代码中已经存在了一些使用mirror来加速依赖安装的例子

- apt-get 使用阿里源
- npm 使用淘宝源
- gems 使用ruby-china源
- hex 使用upyun源

这里我还针对ruby gems搭建了一个私有的mirror，进一步的提升下载gem的速度。使用的是[https://github.com/bundler/gemstash](https://github.com/bundler/gemstash)。在bundle之前，将source换成私有mirror的地址。

### 总结

优化的手段其实总结起来就是：缓存。经过这些步骤优化过后，一个项目的构建时间能缩短60-70%。