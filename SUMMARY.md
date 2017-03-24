# Summary

* [引言](introduction.md)
* [开始之前](kai-shi-zhi-qian.md)
* [Docker](docker.md)
  * [安装docker](docker/an-zhuang-docker.md)
  * [12 Factor App](docker/12-factor-app.md)
  * [为项目创建image](docker/wei-xiangmu-chuang-jian-image.md)
  * [搭建docker registry](docker/da-jian-docker-registry.md)
* [Gitlab CI/CD](da-jian-gitlab.md)
  * [搭建Gitlab](da-jian-gitlab/da-jian-gitlab.md)
  * [配置Gitlab Container Registry](da-jian-gitlab/pei-zhi-gitlab-container-registry.md)
  * [配置Gitlab Runner](da-jian-gitlab/pei-zhi-gitlab-runner.md)
* [kubernetes](kubernetes.md)
  * 配置etcd
  * [配置flannel](kubernetes/flannel.md)
  * 配置docker
  * [部署master](kubernetes/bu-shu-master.md)
    * kube-apiserver
    * kube-controller-manager
    * kube-scheduler
    * kube-dns
  * [部署minions](kubernetes/bu-shu-slave.md)
    * [kubelet](kubernetes/bu-shu-slave/kube-let.md)
    * [kube-proxy](kubernetes/bu-shu-slave/kube-proxy.md)
  * [部署dashboard](kubernetes/bu-shu-dashboard.md)
  * [部署heapster](kubernetes/bu-shu-heapster.md)
* [使用kubernetes集群运行项目](bu-shu-di-yi-ge-xiang-mu.md)
  * deploy
  * rolling update
  * [scaling](bu-shu-di-yi-ge-xiang-mu/scaling.md)
  * [service](bu-shu-di-yi-ge-xiang-mu/service.md)
* [ruby项目案例](an-li.md)
  * [.gitlab-ci.yml](an-li/gitlab-ciyml.md)
  * [Gemfile](an-li/gemfile.md)
  * [Dockerfile](an-li/dockerfile.md)
  * [kube-deployment.yml](an-li/kube-deploymentyml.md)
* [深入原理](shen-ru-yuan-li.md)
  * linux namespace
  * docker fs
  * [etcd原理](shen-ru-yuan-li/etcd.md)
  * [kube-proxy如何用iptables实现service的load balance](shen-ru-yuan-li/iptable-load-balance.md)
  * [如何通过flannel实现pod之间的通信](shen-ru-yuan-li/flannel.md)
* [nodejs项目案例](nodejsxiang-mu-an-li.md)
  * [Dockerfile](dockerfile.md)
* [Q&A](Q&A.md)
  * [如何监控pod状态](Q&A/ru-he-jian-kong-pod-zhuang-tai.md)
* [作者](zuo-zhe.md)
