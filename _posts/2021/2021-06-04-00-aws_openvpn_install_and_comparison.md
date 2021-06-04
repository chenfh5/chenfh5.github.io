---
title: aws安装openvpn及部分性能对比
tags: mood vpn
key: 124
article_header:
  type: cover
  image:
    src: https://user-images.githubusercontent.com/8369671/120776756-23e57900-c557-11eb-9586-d1409bf0f6bc.png
---

# Overview
很多时候需要vpn(virtual private network)来保护网络privacy, 以及跨越一些restriction. 所以记录openvpn的安装过程及其对比,
1. 使用aws安装openvpn的主要流程
2. 列举安装后的openvpn与其他vpn之间的[speedtest](https://www.speedtest.net)对比

# How vpn works
利用encryption key在vpn-client与vpn-server之间加密/解密网络数据.

Only your computer and the VPN server know this key.
![image](https://user-images.githubusercontent.com/8369671/120814088-1a710680-c581-11eb-8df7-32e6673a9cd4.png)
> vpn diff, credits drsoft

# Install openvpn on aws
这里采用aws的[ec2](https://aws.amazon.com/cn/ec2/)作为server, 当然可以采用更轻量级的[lightsail](https://aws.amazon.com/cn/lightsail/).

## sign up aws account
这里需要用到真实信用卡, 并临时扣除$1. 否则虽然可以login, 但是有很多restrictions, e.g., 不能launch ec2.

![image](https://user-images.githubusercontent.com/8369671/120780736-f8fd2400-c55a-11eb-9c99-eaecbe1377f2.png)
> aws account homepage

## launch ec2 with openvpn AMI
1. choose `openvpn` AMI
2. choose suitable ec2 instance
3. create a new key pair (you can only download from the web once)

![image](https://user-images.githubusercontent.com/8369671/120781230-732da880-c55b-11eb-8748-7d03d2e14c32.png)
> choose AMI

![image](https://user-images.githubusercontent.com/8369671/120781822-036bed80-c55c-11eb-9ccf-e4e1c4b2a53e.png)
> choose instance 

![image](https://user-images.githubusercontent.com/8369671/120782465-9c026d80-c55c-11eb-86eb-fd00eba39f75.png)
> create & download key pair

ps. 

有需求的话, 这里可以使用[shadowsocks](https://shadowsocks.org/en/index.html)来替换openvpn. 

这次采用openvpn是因为aws free tier集成了它, 使得安装一键化.

当然如果是[shadowsocks](https://juejin.cn/post/6844903958834593806)的话, 就是在linux下pip/wget来安装.

## configure openvpn server using SSH
here `ip1` is your `Public` IPv4 address, `ip2` is your `Private` IPv4 address, 
1. ssh to ec2 from local with `root`
    - ssh -i somepath/your-key-pair.pem root@ec2-ip1.amazonaws.com
    - if the `pem` are too open, then `chmod 400 somepath/your-key-pair.pem` to make it private
    - initial openvpn access server config
    ![image](https://user-images.githubusercontent.com/8369671/120785756-04068300-c560-11eb-8cb4-3c6112361191.png)

2. ssh to ec2 from local with `openvpnas`
    - ssh -i somepath/your-key-pair.pem openvpnas@ec2-ip1.amazonaws.com
3. setup password used by openvpn UI
   - `sudo passwd openvpn`
4. login openvpn web UI(optional)
    - type ip1 in chrome

![image](https://user-images.githubusercontent.com/8369671/120785299-98bcb100-c55f-11eb-9187-3784c27fcfd5.png)
> ip1 and ip2 in aws web

![image](https://user-images.githubusercontent.com/8369671/120786295-a0308a00-c560-11eb-9f10-763bf5d1f29b.png)
> openvpn web UI login

![image](https://user-images.githubusercontent.com/8369671/120787021-7deb3c00-c561-11eb-87d6-5102ec26bb60.png)
> openvpn web UI

## connect to openvpn server using its [client](https://openvpn.net/vpn-server-resources/connecting/)
我的设备是mac和iPhone,
### mac
![image](https://user-images.githubusercontent.com/8369671/120792127-a4ac7100-c567-11eb-9069-4081041f9e4c.png)
> import profile in mac

![image](https://user-images.githubusercontent.com/8369671/120788216-d111be80-c562-11eb-9589-334a14a907f0.png)
> login

### ios
![image](https://user-images.githubusercontent.com/8369671/120792395-0d93e900-c568-11eb-8dd1-2e15f4ca66ab.png)
> import profile in ios

## current usage check
![image](https://user-images.githubusercontent.com/8369671/120792853-b7737580-c568-11eb-8f76-8d3826464b4f.png)
> two users surfing 

## aws free tier limit
如果经常使用刚搭建的vpn上传out/下载in YouTube, 那么流量会飞快消耗. 此时很可能需要额外支付超过每月[15GB](https://www.zhihu.com/question/67967025/answers/updated)的流量

![image](https://user-images.githubusercontent.com/8369671/120832025-c4a55a00-c592-11eb-89ce-655404eeab84.png)
> check ec2 network usage

# Comparison
## details

vpn | no vpn | [openvpn](https://openvpn.net/) | [Hotspot Shield](https://www.hotspotshield.com/) | [VPN - Super Unlimited Proxy](https://apps.apple.com/us/app/vpn-super-unlimited-proxy/id1370293473)
---- | ---- | ---- | ---- | ----
snapshot | ![image](https://user-images.githubusercontent.com/8369671/120794214-82682280-c56a-11eb-8c4f-83f30b570c75.png) | ![image](https://user-images.githubusercontent.com/8369671/120794242-88f69a00-c56a-11eb-9ae4-faddedb51130.png) | ![image](https://user-images.githubusercontent.com/8369671/120794275-914ed500-c56a-11eb-8f98-3b0dee129717.png) | ![image](https://user-images.githubusercontent.com/8369671/120794484-e25ec900-c56a-11eb-96d9-0a48995b5fed.png)

## summary
![image](https://user-images.githubusercontent.com/8369671/120816782-a84df100-c583-11eb-82f5-fb0b8040821d.png)
> comparison

可以看出,
1. normally多了一层vpn会慢一些(encrypt, etc.)
1. openvpn较快
1. hotspot shield较慢

# Reference
1. [setup a FREE VPN server in the cloud(AWS)](https://youtu.be/m-i2JBtG4FE)
1. [Amazon VPC是Amazon EC2的网络层](https://docs.aws.amazon.com/zh_cn/vpc/latest/userguide/what-is-amazon-vpc.html)
1. [What is a VPN?](https://www.hotspotshield.com/what-is-a-vpn/)
