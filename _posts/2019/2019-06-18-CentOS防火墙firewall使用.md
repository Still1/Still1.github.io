---
title: CentOS防火墙firewall使用
tags: [CentOS]
---

## CentOS防火墙结构

CentOS7以后默认使用firewall配置防火墙规则

<img src="https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20230228051359.png" width="650px" />

firewall与iptables都只用于安全规则的配置，真正根据安全规则工作的是Linux内核的netfilter

## firewall防火墙区域

相当于预设的规则策略，默认为public

<img src="https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20230228051528.png" width="650px" />

## firewall服务命令

```shell
# 安装firewalld
yum install firewalld
# 开启服务
systemctl start firewalld
# 关闭服务
systemctl stop firewalld
# 开机启动服务
systemctl enable firewalld
# 关闭开机启动服务
systemctl disable firewalld
# 查看防火墙状态
systemctl status firewalld
firewall-cmd --state
```

## firewall安全规则设置

永久生效规则（带`--permanent`参数），配置后需要执行`firewall-cmd --reload`才生效

```shell
# 查看所有
firewall-cmd --list-all

# 端口
firewall-cmd --list-ports
firewall-cmd --query-port=80/tcp
firewall-cmd --permanent --add-port=80/tcp
firewall-cmd --permanent --add-port=3000-5000/tcp
firewall-cmd --permanent --remove-port=80/tcp

# 服务
firewall-cmd --get-service
firewall-cmd --list-services
firewall-cmd --query-service ssh
firewall-cmd --permanent --add-service=ssh
firewall-cmd --permanent --remove-service=ssh
firewall-cmd --permanent --remove-service=ssh

# rich-rule
# 允许来自IP地址192.168.1.103的主机访问TCP端口3306-5200
firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="192.168.1.103" port protocol="tcp" port="3306-5200" accept"
firewall-cmd --permanent --remove-rich-rule="rule family="ipv4" source address="192.168.1.103" port protocol="tcp" port="3306" accept"
firewall-cmd --query-rich-rule="rule family="ipv4" source address="192.168.1.103" port protocol="tcp" port="3306" accept"

# 允许来自IP地址192.168.1.1（子网掩码为24位）的主机访问SSH服务
firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="192.168.1.1/24" service name="ssh" accept"
firewall-cmd --permanent --remove-rich-rule="rule family="ipv4" source address="192.168.1.1/24" service name="ssh" accept"
firewall-cmd --query-rich-rule="rule family="ipv4" source address="192.168.1.1/24" service name="ssh" accept"

```

## Docker端口映射的影响

Docker容器端口映射到宿主机，会自动修改宿主机的iptables配置，绕过firewall打开该端口的访问限制

重启firewall后，需要重启Docker，否则Docker无法设置iptables而报错

## 参考文档

* [https://zhuanlan.zhihu.com/p/510863202](https://zhuanlan.zhihu.com/p/510863202)
* [https://juejin.cn/post/6992452262756876296](https://juejin.cn/post/6992452262756876296)