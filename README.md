## 项目简介

本仓库是一个用于服务器初始化配置和监控部署的 Ansible Role 集合。

当前主要入口如下：

- `roles/server-setup/entrance-main.yml`：基础环境、OpenSSL、OpenSSH 相关配置
- `roles/prometheus-node-setup/entrance-main.yml`：Prometheus Node Exporter 安装与服务配置

## 支持的操作系统

- openEuler 22.03 LTS
- Kylin Linux Advanced Server V10 (Lance)

## 目录结构

```text
roles/
  hosts/host.ini
  server-setup/
    entrance-main.yml
  prometheus-node-setup/
    entrance-main.yml
```

各 role 的编排入口保持在 `tasks/main.yml`，具体任务按顺序通过 `include_tasks` 组织。

## Inventory 说明

- `roles/hosts/host.ini` 是示例模板，不应保存真实密码、私有 IP 或生产环境 SSH 信息。
- 执行前请替换占位值，或使用本地/私有 inventory 文件覆盖。
- 当前 playbook 使用的小写主机组包括 `middleware` 和 `node_exporter`，示例 inventory 已与之对齐。

## 软件包前置条件

仓库中的安装流程默认从目标主机上的 `/root/asb_pkgs` 读取安装包，不会从控制机上传。

至少需要准备以下归档文件：

| 组件 | 文件名 | 用途 |
| --- | --- | --- |
| OpenSSL | `openssl-1.1.1w.tar.gz` | `server-setup` 编译安装 |
| OpenSSH | `openssh-10.0p2.tar.gz` | `server-setup` 编译安装 |
| Node Exporter | `node_exporter-1.10.2.linux-amd64.tar.gz` | `prometheus-node-setup` 安装 |

## 执行方式

请始终从仓库根目录执行 playbook：

```bash
ansible-playbook -i roles/hosts/host.ini roles/server-setup/entrance-main.yml
ansible-playbook -i roles/hosts/host.ini roles/prometheus-node-setup/entrance-main.yml
```

仅检查 YAML/Ansible 语法：

```bash
ansible-playbook -i roles/hosts/host.ini roles/server-setup/entrance-main.yml --syntax-check
ansible-playbook -i roles/hosts/host.ini roles/prometheus-node-setup/entrance-main.yml --syntax-check
```

在非生产主机组上预演变更：

```bash
ansible-playbook -i roles/hosts/host.ini roles/server-setup/entrance-main.yml --check --diff
ansible-playbook -i roles/hosts/host.ini roles/prometheus-node-setup/entrance-main.yml --check --diff
```

按 tag 限制执行范围：

```bash
ansible-playbook -i roles/hosts/host.ini roles/server-setup/entrance-main.yml --tags base
ansible-playbook -i roles/hosts/host.ini roles/server-setup/entrance-main.yml --tags ssh
ansible-playbook -i roles/hosts/host.ini roles/prometheus-node-setup/entrance-main.yml --tags monitoring
```

## Node Exporter 说明

`roles/prometheus-node-setup/entrance-main.yml` 会执行以下操作：

- 检查目标主机上的 `/root/asb_pkgs/node_exporter-1.10.2.linux-amd64.tar.gz`
- 解压并安装 `node_exporter` 到 `/usr/local/prometheus-exporters`
- 下发 `prometheus_node_exporter.service`
- 启用并启动 systemd 服务
- 校验服务状态并输出 `9100` 端口监听结果

## 人工验证

安装流程或服务管理相关变更完成后，建议在目标主机执行：

```bash
systemctl is-active prometheus_node_exporter
systemctl status prometheus_node_exporter
ss -lntp | grep 9100
```

## 安全建议

- 不要提交真实密码、私有 inventory 或特定环境的 IP 变更。
- 对共享分支，优先使用本地覆盖配置或 Ansible Vault 管理敏感信息。


