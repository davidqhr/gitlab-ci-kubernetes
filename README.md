# 引言

有一篇很出名的文章：[https://circleci.com/blog/its-the-future/](https://circleci.com/blog/its-the-future/ "asdfasdf")，文章大意是一个boss想要部署一个简单的CRUL rails项目，他咨询一位专家，专家极力反对他将项目部署在heroku上，并给出的建议是将他的简单的app拆分成12个微服务\(microservices\)，运行在docker container中，彼此通过api调用。然后将这些微服务部署在一个由8台机器组成的kubernetes集群中。

我在看到这篇文章的时候，并不是很理解拆分microservices来开发和部署的方式，没有体会到这样做有哪些优势。

当时的课程格子后端结构也是比较简单：

* 一个巨大rails项目，承载了app端的所有版本的api、很多h5页面、以及运营同学的运营工具
* 一个独立爬虫系统
* 后端存储服务，如MySQL，PG，Redis，ElasticSearch

为什么要写这本书

总结技术要点，为团队成员当做技术手册使用。

为感兴趣的朋友提供一些帮助，避免一些弯路。、

后端、运维工程师

