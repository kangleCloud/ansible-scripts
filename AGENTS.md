# 仓库指南

## 项目结构与模块组织

本仓库是一个用于服务器初始化配置和监控部署的 Ansible Role 集合，并不是应用程序代码库。

主要入口文件包括：

- `roles/server-setup/entrance-main.yml`：用于基础操作系统、OpenSSL 和 OpenSSH 相关配置。
- `roles/prometheus-node-setup/entrance-main.yml`：用于安装 Node Exporter。

每个 Role 的编排逻辑放在：

```text
tasks/main.yml
```

默认变量或软件包元数据放在：

```text
vars/main.yml
```

当前共享 inventory 文件位于：

```text
roles/hosts/host.ini
```

## 构建、测试与开发命令

请从仓库根目录执行 playbook，这样路径解析会更加稳定、可预测：

```bash
ansible-playbook -i roles/hosts/host.ini roles/server-setup/entrance-main.yml
ansible-playbook -i roles/hosts/host.ini roles/prometheus-node-setup/entrance-main.yml
ansible-playbook -i roles/hosts/host.ini roles/server-setup/entrance-main.yml --syntax-check
ansible-playbook -i roles/hosts/host.ini roles/server-setup/entrance-main.yml --check --diff
```

开发过程中可以使用以下 tags 限制执行范围：

```bash
--tags base
--tags ssh
--tags monitoring
```

## 代码风格与命名规范

YAML 使用 **2 个空格缩进**。

Playbook 和 task 文件使用 `kebab-case` 命名，例如：

```text
set-hostname.yml
enable-start-node-exporter-service.yml
```

每个 task 都需要提供清晰、可描述用途的 `name`。

`tasks/main.yml` 应保持简洁，主要用于按顺序编排：

```yaml
include_tasks
```

变量命名遵循现有模式：

路径和软件包常量使用大写蛇形命名，例如：

```text
SOFTWARE_PATH
OPENSSL_SRC_ARCHIVE
```

布尔开关使用类似命名：

```text
install_openssh
```

## 测试指南

当前仓库尚未提交自动化测试套件，因此验证主要基于 playbook 执行。

提交 PR 前，至少需要执行：

```bash
--syntax-check
```

如果变更涉及安装流程或服务管理，还需要在非生产主机组上执行：

```bash
--check --diff
```

并记录人工验证结果，例如：

```bash
systemctl status sshd
systemctl status node_exporter
```

## Commit 与 Pull Request 规范

近期提交历史使用较短的 conventional prefix，例如：

```text
feat:
fix:
docs:
```

每个 commit 应保持单一目的，并在提交摘要中说明基础设施变更，例如：

```text
fix: correct OpenSSH compile task order
```

PR 中应列出：

- 受影响的 roles
- 目标主机组
- `/root/asb_pkgs` 下所需的软件包归档文件
- 已执行的验证命令
- 回滚方案
- 密钥或敏感信息处理注意事项

## 安全与配置建议

不要提交真实密码、私有 inventory 或特定环境的 IP 变更。

当前被跟踪的 inventory 文件包含内联 SSH 配置。因此，在共享分支前，建议优先使用：

- 本地覆盖配置
- Ansible Vault

来管理敏感信息。
