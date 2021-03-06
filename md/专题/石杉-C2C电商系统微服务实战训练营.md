
<!-- TOC -->

- [01_我们为什么要使用复杂的微服务架构？](#01_我们为什么要使用复杂的微服务架构)
- [02_国内BAT互联网大厂的微服务架构演进路线](#02_国内bat互联网大厂的微服务架构演进路线)
- [03_海外硅谷互联网大厂的微服务架构演进路线](#03_海外硅谷互联网大厂的微服务架构演进路线)
- [04_目前国内公司的主流微服务技术栈介绍](#04_目前国内公司的主流微服务技术栈介绍)
- [05_C2C电商社会化治理平台项目介绍](#05_c2c电商社会化治理平台项目介绍)
- [06_C2C电商社会化治理平台面向的痛点介绍](#06_c2c电商社会化治理平台面向的痛点介绍)
- [07_C2C电商社会化治理平台的解决方案介绍](#07_c2c电商社会化治理平台的解决方案介绍)
- [08_C2C电商社会化治理平台整体架构设计](#08_c2c电商社会化治理平台整体架构设计)
- [09_C2C电商社会化治理平台的微服务拆分设计](#09_c2c电商社会化治理平台的微服务拆分设计)
- [10、为什么微服务化的系统架构必须要有注册中心？](#10为什么微服务化的系统架构必须要有注册中心)
- [11_ZooKeeper、Eureka、Consul、Nacos的选型对比](#11_zookeepereurekaconsulnacos的选型对比)
- [12_SpringCloudAlibaba之 Nacos 注册中心架构原理](#12_springcloudalibaba之-nacos-注册中心架构原理)
- [13、深入 Nacos 服务注册中心的内核原理](#13深入-nacos-服务注册中心的内核原理)
- [14_01_基于Nacos实现高可用服务注册中心部署](#14_01_基于nacos实现高可用服务注册中心部署)
- [14_02_基于Nacos实现高可用服务注册中心部署](#14_02_基于nacos实现高可用服务注册中心部署)
- [14_03_基于Nacos实现高可用服务注册中心部署](#14_03_基于nacos实现高可用服务注册中心部署)
- [14_04_基于Nacos实现高可用服务注册中心部署](#14_04_基于nacos实现高可用服务注册中心部署)
- [14_05_基于Nacos实现高可用服务注册中心部署](#14_05_基于nacos实现高可用服务注册中心部署)
- [14_06_基于Nacos实现高可用服务注册中心部署](#14_06_基于nacos实现高可用服务注册中心部署)
- [15_为什么微服务化的系统必须通过RPC框架进行通信？](#15_为什么微服务化的系统必须通过rpc框架进行通信)
- [16_Feign+Ribbon、Dubbo、gRPC的选型对比](#16_feignribbondubbogrpc的选型对比)
- [18_Dubbo RPC框架集成 Nacos 注册中心](#18_dubbo-rpc框架集成-nacos-注册中心)
- [19_01_基于Dubbo开发C2C电商社会化治理平台人群服务（上）](#19_01_基于dubbo开发c2c电商社会化治理平台人群服务上)
- [20_基于Dubbo开发C2C电商社会化治理平台任务服务](#20_基于dubbo开发c2c电商社会化治理平台任务服务)
- [21_01_基于Dubbo开发C2C电商社会化治理平台权益服务(上)](#21_01_基于dubbo开发c2c电商社会化治理平台权益服务上)
- [22_01、基于Dubbo开发C2C电商社会化治理平台基础服务（1）](#22_01基于dubbo开发c2c电商社会化治理平台基础服务1)

<!-- /TOC -->



# 01_我们为什么要使用复杂的微服务架构？

协调困难、开发效率低、尝试新技术困难。

# 02_国内BAT互联网大厂的微服务架构演进路线

![](../../pic/2020-09-20/2020-09-20-22-04-04.png)

# 03_海外硅谷互联网大厂的微服务架构演进路线

早期的spring cloud微服务体系的组件，是以eureka、feign+ribbon、zuul、hystrix，用zipkin和sleuth做链路监控，stream做消息中间件集成，contract做契约测试支持，当然gateway也可以做网关，consul也是一种注册中心，还有跟spring security配合的安全认证，跟k8s配合的容器支持


# 04_目前国内公司的主流微服务技术栈介绍

因为阿里的dubbo重启活跃维护，同时阿里把自己的微服务技术栈融入进了spring cloud体系和标准，形成了一个spring cloud alibaba微服务技术组件，也就是以nacos、dubbo、seata、sentinal、rocketmq等技术为核心的一套技术体系

- 注册中心：nacos -> eureka
- RPC框架：dubbo -> feign+ribbon
- 分布式事务：seata -> 无
- 限流/熔断/降级：sentinel -> hystrix
- API网关：无 -> zuul

spring cloud netflix微服务技术组件，开始更新的非常不活跃，netflix公司公开宣布他之前开源的一些微服务组件未来就不会怎么更新了，这就导致spring cloud netflix微服务技术组件的未来有点不太光明


spring cloud alibaba技术栈+国内开源的组件（apollo、CAT）+ Prometheus + ELK + Spring Cloud Gateway（Nginx+lua、Kong、Zuul、API网关自研）


# 05_C2C电商社会化治理平台项目介绍

所谓的C2C二手电商平台，就是用户可以在上面作为卖家发布自己的二手商品，然后等待买家来谈，来购买，平台就是作为中间服务方提供一些列的平台功能支持，其实国内大的二手交易平台还是有几个的，同时还有好一些专注于垂直领域的二手电商交易平台，比如说二手奢侈品电商，二手电子产品电商


社会化治理平台：对用户发布的信息进行审核。

通过一个平台，以技术的手段，将有人举报违规的内容推送给部分用户，让用户参与到平台治理中来，用户投票决定某个商品或者评论等内容是否违规，这样平台仅仅作为一个桥梁，让用户进行社会化自治


# 06_C2C电商社会化治理平台面向的痛点介绍




# 07_C2C电商社会化治理平台的解决方案介绍

![社会化治理平台架构设计](../../pic/2020-09-20/2020-09-20-22-25-06.png)




# 08_C2C电商社会化治理平台整体架构设计

# 09_C2C电商社会化治理平台的微服务拆分设计

# 10、为什么微服务化的系统架构必须要有注册中心？

# 11_ZooKeeper、Eureka、Consul、Nacos的选型对比

# 12_SpringCloudAlibaba之 Nacos 注册中心架构原理

# 13、深入 Nacos 服务注册中心的内核原理

# 14_01_基于Nacos实现高可用服务注册中心部署


# 14_02_基于Nacos实现高可用服务注册中心部署

# 14_03_基于Nacos实现高可用服务注册中心部署

# 14_04_基于Nacos实现高可用服务注册中心部署

# 14_05_基于Nacos实现高可用服务注册中心部署

# 14_06_基于Nacos实现高可用服务注册中心部署

# 15_为什么微服务化的系统必须通过RPC框架进行通信？

# 16_Feign+Ribbon、Dubbo、gRPC的选型对比

# 18_Dubbo RPC框架集成 Nacos 注册中心

# 19_01_基于Dubbo开发C2C电商社会化治理平台人群服务（上）

# 20_基于Dubbo开发C2C电商社会化治理平台任务服务

# 21_01_基于Dubbo开发C2C电商社会化治理平台权益服务(上)



# 22_01、基于Dubbo开发C2C电商社会化治理平台基础服务（1）


















