# test1

由 `quseijuro-design` 维护的仓库。

## 环境

- Git 安装位置：`D:\Professional\Git`
- 提交身份：`mkt` / `quseijuro@gmail.com`
- 代理（FlClash）：`http://127.0.0.1:7890`

## 常用命令

```bash
# 拉取最新
git pull

# 提交改动
git add .
git commit -m "你的提交说明"
git push
```

## 说明

本仓库通过 SSH 方式连接 GitHub，首次使用需在
GitHub → Settings → SSH and GPG keys 中添加本机公钥（`~/.ssh/id_ed25519.pub`）。

## Skills 目录

`skills/` 目录用于存放**可复用的 WorkBuddy 技能**：每个子目录是一个独立 skill，内含 `SKILL.md`
描述其触发条件与执行步骤，由 WorkBuddy 自动加载并复用。适合把"换机器 / 重装系统 / 帮别人配环境"
这类会重复执行的流程固化下来，避免每次从头排错。

当前包含的技能：

| 技能 | 用途 |
| --- | --- |
| `skills/windows-git-setup` | 在 Windows 上独立安装 Git for Windows、配置提交身份与代理、克隆 GitHub 仓库、生成 SSH 密钥并配置免密推送，完成首个提交（初始化 README）的上推。重点固化了本机三个易踩的坑：安装路径写法、原生 git 对 `/d/` 路径的误解析、SSH 主机密钥缺失。 |
