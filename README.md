## 脚本简介

此脚本基于ansible-playbook role进行编写和实现的

- 修改ansible.cfg中`host_key_checking = False`配置用于解决第一次需要ssh连接

## 支持的操作系统

openEuler-22.03 LTS、Kylin Linux Advanced Server V10 (Lance)

## 支持的安装操作

所有的初始化操作流程是基于ttd文档顺序

- ttd上所有服务器初始化操作
- openssl编译安装
- openssh编译安装
- cmake编译安装
- mysql57安装
- mysql80安装（待完成）
- lua环境安装（ttd中涉及内容）
- lua-nginx安装（ttd中涉及内容）
- nginx（不引入lua）

## ansible中roles执行顺序

baseEnv > openssl> openssh > cmake > mysql ; lua>lua_nginx ; nginx

## entrance_main

此文件为入口文件

```yaml
- hosts: test
  remote_user: root
  gather_facts: yes
  vars:
    - pkgs_path: "/root/asb_pkgs"
  roles:
    #- baseEnv
    #- openSSL
    #- openSSH
    #- cmake
    #- mysql
    #- lua
    #- lua_nginx
    - nginx
```
> `pkgs_path`： 为安装存放路径，所以安装包均需要提前下载，且命名固定

## 所需软件包

|  阶段   |     软件包名称     |     版本      |                           下载地址                           |                备注                |
| :-----: | :----------------: | :-----------: | :----------------------------------------------------------: | :--------------------------------: |
| baseEnv |                    |               |                                                              |                                    |
|         |                    |               |                                                              |                                    |
| openssl |                    |               |                                                              |                                    |
|         |      openssl       |    1.1.1w     | https://github.com/openssl/openssl/releases/download/OpenSSL_1_1_1w/openssl-1.1.1w.tar.gz |                                    |
| openssh |                    |               |                                                              |                                    |
|         |      openssh       |    10.0p2     | https://mirrors.aliyun.com/pub/OpenBSD/OpenSSH/portable/openssh-10.0p2.tar.gz |                                    |
|  cmake  |                    |               |                                                              |                                    |
|         |       cmake        |               |                     cmake-3.27.0.tar.gz                      |                                    |
|   lua   |                    |               |                                                              |                                    |
|         |       luajit       | v2.1-20250529 | https://github.com/openresty/luajit2/archive/refs/tags/v2.1-20250529.tar.gz |        -O luajit-2.1.tar.gz        |
|         |   ngx_devel_kit    |     0.3.4     | https://github.com/vision5/ngx_devel_kit/archive/refs/tags/v0.3.4.tar.gz |   -O ngx_devel_kit-0.3.4.tar.gz    |
|         |  lua-nginx-module  |    0.10.28    | https://github.com/openresty/lua-nginx-module/archive/refs/tags/v0.10.28.tar.gz | -O lua-nginx-module-0.10.28.tar.gz |
|         |   lua-resty-core   |    0.1.31     | https://github.com/openresty/lua-resty-core/archive/refs/tags/v0.1.31.tar.gz |  -O lua-resty-core-0.1.31.tar.gz   |
|         | lua-resty-lrucache |     0.15      | https://github.com/openresty/lua-resty-lrucache/archive/refs/tags/v0.15.tar.gz | -O lua-resty-lrucache-0.15.tar.gz  |
|  nginx  |                    |               |        http://nginx.org/download/nginx-1.24.0.tar.gz         |                                    |
|         |       nginx        |    1.26.0     |        http://nginx.org/download/nginx-1.26.0.tar.gz         |                                    |
|         |                    |               |                                                              |                                    |

> 注意： 以上的安装包均为ttd中对应安装文档中的安装包

## 主机配置

```yaml
[test]
192.168.0.49 NODE_NAME="Proxy-1-49"
```

> NODE_NAME：此变量值用于设置hostname
>
> ROLE_NAME：此变量值用于安装特定软件，如mysql、nginx等

## 软件使用

```bash
cd /etc/ansible
ansible-playbook role/entrance_main.yml
```



