---
title: 卡梅拉架构一体化
tags: architecture system-design kubernetes
key: 121
article_header:
  type: cover
  image:
    src: https://user-images.githubusercontent.com/8369671/114143990-67e94280-9947-11eb-8925-ee610cbe2c71.png
---

# Overview
整理卡梅拉一体化的架构, 从底层的aws -> terraform -> k8s -> spinnaker -> 主体application code

之前, 在多租户(multi tenancy)下资源很难隔离. 如,
- instance level的粒度, 提交spark jar包到Oozie, Oozie到期就submit spark job到yarn, yarn回应一个instance作为spark driver, 之后再回应一些instances作为spark executors
  ![image](https://user-images.githubusercontent.com/8369671/114137718-2eacd480-993f-11eb-8ced-1abc96199ec6.png)
  > spark yarn cluster mode, credit: @goyalsaurabh66

- docker level的粒度, instance level的粒度过大容易造成resource的浪费. 而如果是docker level, 通过co-group和namespace可以更好地隔离划分mem,cpu. 可以做到更小的粒度更精细的掌控
  ![image](https://user-images.githubusercontent.com/8369671/114138516-53ee1280-9940-11eb-9ea0-1fb4cb09e990.png)
  > spark k8s cluster mode, credit: databricks

从上面可以看到演进就是k8s replace了yarn.

# Breakdown
![image](https://user-images.githubusercontent.com/8369671/114144818-4fc5f300-9948-11eb-9f1e-dcd0f0b73df4.png)
> 整体arch

## infra层
使用terraform来管理云机器资源, 然后使用gitlab maintain terraform code, 每次code change都会触发ci来实现`terraform-apply`以CRUD云资源

## orchestration层
使用k8s来编排. 

- 这里可以直接使用eks(aws build好的k8s cluster)
- 或者apply一些ec2来自建k8s cluster

之后暴露k8s master接口即可

## deployment层
这里是k8s master(scheduler和api-server)所接入的yaml(符合k8s格式)暂存的地方. 比如,
- 直接写成k8s-yaml然后通过kubectl submit到k8s-master
- 用spinnaker过渡, 然后gitlab maintain spinnaker code/jsonnet, spinnaker组合多方之后再submit到k8s-master

## image/application层
这里是具体应用层, 通过ci将`code change`打包成image, 然后存放在repo供之后的deploy调用. 

这些application可以是airflow, oozie, db, spark等标准包. 也可以是自定义包, 如spark-sklearn-myOwnFunction image.

## client层
对用户展示, 通常是spinnaker的ui或者自包装ui.

# Reference
0. [使用 Terraform 在 AWS 中国区域实现自动化部署指南系列（一） TERRAFORM 入门](https://aws.amazon.com/cn/blogs/china/aws-china-region-guide-series-terraform1/#:~:text=Terraform%E6%98%AF%E4%B8%80%E4%B8%AAIT%E5%9F%BA%E7%A1%80,%E6%8E%A7%E5%88%B6%E7%9A%84AWS%E5%9F%BA%E7%A1%80%E8%AE%BE%E6%96%BD%E3%80%82)
0. [Amazon EKS 用户指南](https://docs.aws.amazon.com/zh_cn/eks/latest/userguide/getting-started.html)
0. [terraform-aws-eks](https://registry.terraform.io/modules/terraform-aws-modules/eks/aws/latest)
0. 卡梅拉white paper
0. [Spark 迁移到 K8S 在有赞的实践与经验](https://mp.weixin.qq.com/s/PTH0LloSt0-z_glB4f1loQ)
