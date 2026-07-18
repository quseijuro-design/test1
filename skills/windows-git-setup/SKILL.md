---
name: windows-git-setup
description: >-
  在 Windows 上独立安装 Git for Windows、配置全局身份与代理、克隆 GitHub 仓库、生成首个提交并配置
  SSH 免密推送的完整可复用流程。当用户要在 Windows 上"装 git / 配置 git / 连接 GitHub 仓库 /
  生成第一个提交 / 配置 SSH 推送 / 设置 git 代理"时触发。
license: MIT
metadata:
  author: midor
  version: 1.0.0
  created: 2026-07-18
  last_reviewed: 2026-07-18
  review_interval_days: 180
---

# Windows 独立安装 Git + 首个提交 + SSH 推送

一键式流程：下载安装 Git for Windows 到自定义目录 → 配身份/代理 → 克隆仓库 → 生成 SSH 密钥并配
免密推送 → 建首个提交并 push。重点在于避开本机几个特有的坑（见下）。

## 这个 skill 的作用

当你需要在 Windows 上**从零搭建一套可用的 Git 环境并对接 GitHub** 时，按本 skill 走即可一次性完成：

- 安装**独立的 Git for Windows**（不依赖任何 IDE / 编辑器捆绑的便携版），并加入系统 PATH，使所有终端都能直接用 `git`；
- 配置提交身份（`user.name` / `user.email`）与代理（配合 FlClash 等本地代理客户端，便于访问 GitHub）；
- 克隆指定 GitHub 仓库到本地目录；
- 生成 SSH 密钥、配置免密推送，并完成**首个提交（初始化 README）的上推**。

它最大的价值不是罗列步骤，而是**固化了本机环境下三个极易踩的坑**（安装路径写法、原生 git 对 `/d/` 路径的误解析、SSH 主机密钥缺失），避免每次都从头排错。适合「换机器 / 重装系统 / 帮别人配环境」时直接复用。

## 必读：关键坑（不读必踩）

1. **安装器 `/DIR` 必须用 Windows 反斜杠路径**：`/DIR="D:\Professional\Git"`。
   用前向斜杠 `/DIR=/d/Professional/Git` 会被静默忽略——安装器退出码仍是 0，但实际什么都没装，
   `D:\Professional\Git` 目录不存在。

2. **原生 `git.exe` 会误解析 `/d/` 风格的 Unix 路径**：在本机 Bash 里 `/d/Professional/test1`
   确实是 D 盘，但原生 `git.exe` 把 `-C /d/Professional/test1` 误当成 `C:/d/Professional/test1`
   （错目录），导致 `rev-parse --show-toplevel` 返回错误路径、git 看不到文件、`git add` 报
   `pathspec 'README.md' did not match`。
   **正确写法：所有 git 操作一律用 Windows 原生盘符路径**，例如
   `-C "D:/Professional/test1"`、`core.sshCommand "D:/Professional/Git/usr/bin/ssh.exe"`。
   （同理 `ssh.exe`、`ssh-keyscan.exe` 的调用也都用 `D:/` 形式。）

## 步骤

### 1. 下载安装包（如访问 GitHub 慢，先开代理）
```bash
export http_proxy=http://127.0.0.1:7890
export https_proxy=http://127.0.0.1:7890
URL=$(curl -sL https://api.github.com/repos/git-for-windows/git/releases/latest \
  | grep -oE 'https://github.com/git-for-windows/git/releases/download/[^"]*Git-.*-64-bit.exe' | head -1)
curl -L -o "$TEMP/git-installer.exe" "$URL"
```

### 2. 静默安装（反斜杠 /DIR + /PATH=1 加入系统 PATH）
```powershell
"$TEMP/git-installer.exe" /VERYSILENT /NORESTART /DIR="D:\Professional\Git" /PATH=1
# 验证：D:\Professional\Git\bin\git.exe --version
```
> 新开一个终端（PowerShell / CMD / Windows Terminal）后，`git` 才进 PATH。

### 3. 全局配置：身份 + 代理
```bash
/d/Professional/Git/bin/git.exe config --global user.name "你的名字"
/d/Professional/Git/bin/git.exe config --global user.email "you@example.com"
/d/Professional/Git/bin/git.exe config --global http.proxy http://127.0.0.1:7890
/d/Professional/Git/bin/git.exe config --global https.proxy http://127.0.0.1:7890
```
不用代理时撤销：`git config --global --unset http.proxy` 与 `https.proxy`。

### 4. 克隆仓库（用原生 D:/ 路径）
```bash
/d/Professional/Git/bin/git.exe clone https://github.com/<owner>/<repo>.git "D:/Professional/<repo>"
```
> 仓库若是空的（无提交），clone 后 `git status` 会显示 "No commits yet"，正常。

### 5. 生成 SSH 密钥（无密码，适合无界面环境）
```bash
mkdir -p "C:/Users/$USER/.ssh"
/d/Professional/Git/usr/bin/ssh-keygen.exe -t ed25519 -C "you@example.com" \
  -f "C:/Users/$USER/.ssh/id_ed25519" -N ""
# 把下面这串公钥给用户，去 GitHub 添加：
cat "C:/Users/$USER/.ssh/id_ed25519.pub"
```

### 6. 让 git 走自带 ssh + remote 切到 SSH（都用 D:/ 路径）
```bash
/d/Professional/Git/bin/git.exe config --global core.sshCommand "D:/Professional/Git/usr/bin/ssh.exe"
/d/Professional/Git/bin/git.exe -C "D:/Professional/<repo>" remote set-url origin git@github.com:<owner>/<repo>.git
```

### 7. 用户把公钥加到 GitHub
GitHub → Settings → SSH and GPG keys → New SSH key → Title 随意 → 粘贴 `id_ed25519.pub` → Add。

### 8. 把 github.com 主机密钥写入 known_hosts（否则 push 报 Host key verification failed）
```bash
/D/Professional/Git/usr/bin/ssh-keyscan.exe -t ed25519,rsa github.com >> "C:/Users/$USER/.ssh/known_hosts"
```

### 9. 首个提交 + 推送
```bash
# 仓库里先建一个 README.md（用 Write 工具写入 D:/Professional/<repo>/README.md）
/d/Professional/Git/bin/git.exe -C "D:/Professional/<repo>" add README.md
/d/Professional/Git/bin/git.exe -C "D:/Professional/<repo>" commit -m "chore: 初始化仓库，添加 README"
/d/Professional/Git/bin/git.exe -C "D:/Professional/<repo>" push -u origin main
```
> 之后日常提交直接 `git push` 即可（需在新终端里，git 才在 PATH）。

## 常见问题速查
- `fatal: cannot spawn .../ssh.exe: No such file or directory` → `core.sshCommand` 里用了 `/d/` 写法被误解析，改成 `D:/...`。
- `Host key verification failed` → 缺 known_hosts，跑第 8 步 `ssh-keyscan`。
- `pathspec 'README.md' did not match any files` → 用了 `/d/` 路径导致 git 看错目录，全部改 `D:/`。
- push 弹登录框 / 报错权限 → 没走 SSH 或公钥未加到 GitHub；确认第 5–7 步。
- 安装器跑完 `D:\Professional\Git` 不存在 → `/DIR` 用了斜杠，重跑第 2 步。

## 验证清单
- [ ] `D:\Professional\Git\bin\git.exe --version` 有输出
- [ ] `git config --global --list` 含 user.name / user.email / proxy / core.sshCommand
- [ ] `git -C "D:/Professional/<repo>" log` 能看到首个 commit
- [ ] `git -C "D:/Professional/<repo>" push` 成功，GitHub 网页能看到提交
