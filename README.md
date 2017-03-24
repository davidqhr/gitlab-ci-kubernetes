Gitlab-ci、kubernetes最佳实践
=======

有一篇很出名的文章：[It's the future](https://circleci.com/blog/its-the-future/)，文章大意是一个boss想要部署一个简单的CRUL rails项目，他咨询一位专家，专家极力反对他将项目部署在heroku上，并给出的建议是将他的简单的app拆分成12个微服务\(microservices\)，运行在docker container中，彼此通过api调用。然后将这些微服务部署在一个由8台机器组成的kubernetes集群中。

初看这篇文章的时候，我并没有理解这种部署的优势。但随着之后[课程格子](https://www.kechenggezi.com)部署方式改进，我重新回顾了这篇文章，发现最终的解决方案和文中提到的如出一辙。

# 导读

如果你想要快速搭建一个gitlab + kubernetes + docker环境，走通整个部署流程，请阅读1-5章。
如果你想要更深入的了解整个系统，请阅读6-10章。

kubernetes 1.5.4
docker 1.13

希望你能 enjoy Gitlab CI + Kubernetes 之旅！

[开始看书](https://davidqhr.gitbooks.io/gitlab-ci-kubernetes/)

本书源码在 GitHub 上，[欢迎参与写书](https://github.com/davidqhr/gitlab-ci-kubernetes)