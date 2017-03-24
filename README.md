# 引言

有一篇很出名的文章：[https://circleci.com/blog/its-the-future/](https://circleci.com/blog/its-the-future/ "asdfasdf")，文章大意是一个boss想要部署一个简单的CRUL rails项目，他咨询一位专家，最后专家给出他的建议是讲他的简单的app拆分成12个微服务\(microservices\)，运行在docker container中，彼此通过api调用。然后将这些微服务部署在一个由8台机器组成的kubernetes集群中。

So I just need to split my simple CRUD app into 12 microservices, each with their own APIs which call each others’ APIs but handle failure resiliently, put them into Docker containers, launch a fleet of 8 machines which are Docker hosts running CoreOS, “orchestrate” them using a small Kubernetes cluster running etcd, figure out the “open questions” of networking and storage, and then I continuously deliver multiple redundant copies of each microservice to my fleet. Is that it?

### 为什么要写这本书

总结技术要点，为团队成员当做技术手册使用。

为感兴趣的朋友提供一些帮助，避免一些弯路。、

后端、运维工程师



