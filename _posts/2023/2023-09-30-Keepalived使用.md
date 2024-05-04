---
title: Keepalived使用
tags: [Keepalived]
---

## Keepalived安装

```shell
yum install -y keepalived
```

## Keepalived配置

主服务器配置

`/etc/keepalived/keepalived.conf`

```
! Configuration File for keepalived

global_defs {
    router_id doubucket
    vrrp_skip_check_adv_addr
    vrrp_strict
    vrrp_garp_interval 0
    vrrp_gna_interval 0
}

vrrp_instance VI_1 {
    state MASTER
    interface ens33
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.88.223
    }
}
```

从服务器配置

`/etc/keepalived/keepalived.conf`
```
! Configuration File for keepalived

global_defs {
    router_id doubucket2
    vrrp_skip_check_adv_addr
    vrrp_strict
    vrrp_garp_interval 0
    vrrp_gna_interval 0
}
  
vrrp_instance VI_1 {
    state BACKUP
    interface ens33
    virtual_router_id 51
    priority 80
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.88.223
    }
}
```

## Keepalived相关命令

```shell
# 检查版本
keepalived -v

# 启动
systemctl start keepalived.service
service keepalived start

# 重启
systemctl restart keepalived.service

# 停止
systemctl stop keepalived.service

# 查看状态
systemctl status keepalived.service

# 设置开机启动
chkconfig keepalived on
systemctl enable keepalived.service
```