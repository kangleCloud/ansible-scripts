## 项目简介

本仓库是一个用于服务器初始化配置和监控部署的 Ansible Role 集合。

当前主入口是 `roles/server-setup/entrance-main.yml`，用于服务器基础环境初始化以及 OpenSSL、OpenSSH 的源码安装。监控相关入口 `roles/prometheus-node-setup/entrance-main.yml` 作为附加 playbook 使用。

## 支持的操作系统

- openEuler 22.03 LTS
- Kylin Linux Advanced Server V10 (Lance)

## 目录结构

```text
roles/
  hosts/host.ini
  server-setup/
    entrance-main.yml
    base_env/
    install_openssl/
    install_openssh/
  prometheus-node-setup/
    entrance-main.yml
```

`server-setup` 当前实际包含以下 role：

- `base_env`：基础环境初始化
- `install_openssl`：OpenSSL 源码安装
- `install_openssh`：OpenSSH 源码安装

每个 role 的编排入口都保持在 `tasks/main.yml`，主要通过 `include_tasks` 顺序组织子任务。

## Inventory 说明

- 实际服务器 inventory 使用 `/etc/ansible/hosts`。
- 仓库中的 `roles/hosts/host.ini` 只是示例模板，用于参考分组写法和变量格式，不作为真实生产执行入口。
- `server-setup` 入口当前使用 `hosts: middleware`，因此 `/etc/ansible/hosts` 中需要维护对应的 `[middleware]` 主机组。
- 监控入口按其自身 playbook 中定义的主机组执行，执行前请在 `/etc/ansible/hosts` 中准备对应分组。

## server-setup 功能说明

`roles/server-setup/entrance-main.yml` 当前默认启用：

- `base_env`：设置主机名、代理、仓库源、系统更新、基础依赖、时间同步、磁盘与文件系统、安全配置、网络配置、执行环境和 Vim。
- `install_openssl`：安装编译依赖、解压并安装 OpenSSL、创建软链接、设置 PATH。
- `install_openssh`：安装依赖、启用 Telnet、解压并编译 OpenSSH、备份配置、安装并启动 `sshd`。

以下组件在入口文件中仍为注释或预留状态，当前 README 不将其视为已启用能力：

- `install_mysql`
- `install_nginx`
- `install_lua`

## 软件包前置条件

仓库中的安装流程默认从目标主机上的 `/root/asb_pkgs` 读取安装包，不会从控制机上传。

`server-setup` 至少需要准备以下归档文件：

| 组件 | 文件名 | 用途 |
| --- | --- | --- |
| OpenSSL | `openssl-1.1.1w.tar.gz` | `install_openssl` 源码安装 |
| OpenSSH | `openssh-10.0p2.tar.gz` | `install_openssh` 源码安装 |

如果需要执行监控入口，还需额外准备对应 exporter 的归档文件。

## 执行方式

请始终从仓库根目录执行 playbook，并显式指定实际 inventory：

```bash
ansible-playbook -i /etc/ansible/hosts roles/server-setup/entrance-main.yml
```

仅检查语法：

```bash
ansible-playbook -i /etc/ansible/hosts roles/server-setup/entrance-main.yml --syntax-check
```

在非生产主机组上预演变更：

```bash
ansible-playbook -i /etc/ansible/hosts roles/server-setup/entrance-main.yml --check --diff
```

按 tag 限制执行范围：

```bash
ansible-playbook -i /etc/ansible/hosts roles/server-setup/entrance-main.yml --tags base
ansible-playbook -i /etc/ansible/hosts roles/server-setup/entrance-main.yml --tags ssh
```

## 监控入口

附加监控入口为：

```bash
ansible-playbook -i /etc/ansible/hosts roles/prometheus-node-setup/entrance-main.yml
```

该入口主要用于安装 Node Exporter，并可根据 inventory 配置附加执行 NGINX Prometheus Exporter。

## 人工验证

`server-setup` 执行完成后，建议优先在目标主机验证：

```bash
systemctl status sshd
systemctl is-enabled sshd
openssl version
ssh -V
```

如果执行了监控入口，再补充验证：

```bash
systemctl status prometheus_node_exporter
ss -lntp | grep 9100
```

## 安全建议

- 不要在仓库中提交真实密码、私有 inventory 或特定环境的 IP 变更。
- `roles/hosts/host.ini` 仅保留示例用途，真实环境建议使用 `/etc/ansible/hosts`、本地覆盖配置或 Ansible Vault 管理敏感信息。
