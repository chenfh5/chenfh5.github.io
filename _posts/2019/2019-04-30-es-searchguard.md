---
title: searchguard-6的安装和配置
tags: es
key: 27
modify_date: 2019-04-30 18:00:00 +08:00
---

记录一下searchguard-6的安装和配置过程，

----
# Overview
elasticsearch在暴露了一个node的ip和端口后就可以对整个集群进行各种操作，删索引，改数据等。在重要的项目应用中，需要防范这一点。

目前常见的安全防范方式有，
1. [X-Pack Elasticsearch Security](https://www.elastic.co/guide/en/x-pack/current/elasticsearch-security.html)，收费License
2. [Search Guard](https://github.com/floragunncom/search-guard)，免费开源

下面就Search Guard，将其最小化安装到es集群。

# 版本
- elasticsearch-6.4.2.tar.gz
- search-guard-6-6.4.2-23.1.zip

# 安装search guard
## sg plugin installation
1. tar -zxvf elasticsearch-6.4.2.tar.gz
2. cd elasticsearch-6.4.2
3. bin/elasticsearch-plugin install -b file:///path/to/search-guard-6-6.4.2-23.1.zip

![sg plugin install success](https://upload-images.jianshu.io/upload_images/2189341-6b5cc54e8f5cd190.png)

## sg demo quick installer

![run demo installer, failed](https://upload-images.jianshu.io/upload_images/2189341-533815b7692d7673.png)

![run demo installer2, succeeded](https://upload-images.jianshu.io/upload_images/2189341-bc149a416691eead.png)

1. 对`install_demo_configuration.sh`赋权
2. 运行`install_demo_configuration.sh`，此时该脚本会将秘钥文件生成，并cp到`/config`下，同时append sg配置内容到`/config/elasticsearch.yml`

![sg自动append的esyml](https://upload-images.jianshu.io/upload_images/2189341-e5114eef4e903867.png)

启动es，正常。
通过浏览器访问es集群，不正常，报错如下，

![SSLException](https://upload-images.jianshu.io/upload_images/2189341-b49e853a47f56b78.png)

应该是浏览器没有建立ssl链接，没有深究这方面，换了一种方式，即在esyml里把SSL关闭。

3. 关闭SSL

![esyml](https://upload-images.jianshu.io/upload_images/2189341-9fc242de5273d26e.png)

![es login succeeded](https://upload-images.jianshu.io/upload_images/2189341-5dcee5f23fdc7338.png)

![sg demo config](https://upload-images.jianshu.io/upload_images/2189341-dd1911c822071abb.png)

## sg自定义1
基于demo生成的证书，直接修改原有账户名及其密码，
1. 生成hash新密码

![hash new password](https://upload-images.jianshu.io/upload_images/2189341-6d9f695ddf3e7744.png)

2. 修改`/sgconfig/sg_internal_users.yml`

![image.png](https://upload-images.jianshu.io/upload_images/2189341-45985fb64016b01a.png)

3. 分发新配置到es集群
```
cd ./plugins/search-guard-6/tools

./sgadmin.sh -cd ../sgconfig/ -icl -nhnv \
   -cacert ../../../config/root-ca.pem \
   -cert ../../../config/kirk.pem \
   -key ../../../config/kirk-key.pem
```

![snapshot of new account and password](https://upload-images.jianshu.io/upload_images/2189341-a099c9de070cc5c4.png)

## sg自定义2
sg可以自定义密码和加密方式。首先下载[ssl生成工具](https://github.com/floragunncom/search-guard-ssl)，然后进行自定义配置，

1. git clone --depth=1 https://github.com/floragunncom/search-guard-ssl.git
2. 配置ca

![root-ca](https://upload-images.jianshu.io/upload_images/2189341-58313c9b38ab91de.png)

![signing-ca](https://upload-images.jianshu.io/upload_images/2189341-cc72b9dc974eafc9.png)

3. 配置生成脚本`example.sh`

![root，node，client的ca生成配置](https://upload-images.jianshu.io/upload_images/2189341-78373c9f3bbe5988.png)

4. cp生成的node证书到es/config
- 首先在es/config删除demo生成的(`*.pem`)
- cp search-guard-ssl的`node-*-keystore.jks`和`truststore.jks`到es/config

5. 配置esyml

![esyml](https://upload-images.jianshu.io/upload_images/2189341-3d329e9effbb9e4f.png)

6. cp生成的client证书到/plugins/search-guard-6/sgconfig/

7. 修改`/sgconfig/sg_internal_users.yml`

![生成hash password](https://upload-images.jianshu.io/upload_images/2189341-8af3c60909a67abd.png)

![配置sg_internal_users.yml](https://upload-images.jianshu.io/upload_images/2189341-7533b8d14f68e2da.png)

8. 分发新配置到es集群

![重启使之生效](https://upload-images.jianshu.io/upload_images/2189341-5e6404956e16ccac.png)

# Reference
- [Search Guard Installation](https://docs.search-guard.com/latest/search-guard-installation)
- [Search Guard Demo Installer](https://docs.search-guard.com/latest/demo-installer)
- [ElasticSearch&Search-guard 5 权限配置](https://www.jianshu.com/p/5a42b3560b27)
- [SearchGuard权限配置](https://www.jianshu.com/p/fffec2c39bba)
