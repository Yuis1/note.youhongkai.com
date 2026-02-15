---
{"dg-publish":true,"permalink":"/CS计算机科学/后端开发/nodejs/多用户共享的前端工具链安装指南（NVM、Node、pnpm）/","noteIcon":"","created":"2026-01-12T12:51:07.442+08:00","updated":"2026-02-16T04:14:31.899+08:00"}
---

## 背景知识

- **NVM 是 shell 脚本**：通过在 `~/.bashrc` / `~/.zshrc` 里 `source nvm.sh`，动态修改 `PATH` 来切换 Node 版本 → 天然是“每用户生效”。
    
- **共享系统级 Node 很容易乱**：版本升级/卸载、权限、并发写入、项目抢环境都会导致不可控。
    
- **真正占空间的是依赖包**：省空间的关键点不是 Node，而是 **pnpm store**。  
    **最佳实践：NVM 每用户 + pnpm store 共享**（省空间最大、权限最少坑）。

## 前置准备

支持：Linux（Ubuntu/Debian/CentOS/RHEL）、bash/zsh

```bash
sudo apt-get update
sudo apt-get install -y curl git ca-certificates
```

---

## 1) 创建共享目录（只共享 pnpm store）

建议使用专用组 `nodejs`（比 `users` 更可控）：

```bash
sudo groupadd nodejs || true
sudo mkdir -p /opt/nodejs/pnpm-store

sudo chgrp -R nodejs /opt/nodejs
sudo chmod -R 2775 /opt/nodejs

sudo usermod -aG nodejs <你的用户名>
# 让组权限生效：重新登录 或 newgrp nodejs
```

## 2) 黄金安装顺序

1. **NVM（每用户）**
2. **Node（通过 NVM，每用户）**
3. **pnpm（每用户安装，但 store 指向共享目录）**


## 3) 安装 NVM + Node + pnpm（每个用户都做一次）

### 3.1 安装 NVM

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
. "$HOME/.nvm/nvm.sh"
```

### 3.2 安装 Node（建议 LTS）

```bash
nvm install --lts
nvm use --lts
nvm alias default 'lts/*'
node -v
npm -v
```

### 3.3 安装 pnpm（推荐：独立安装脚本）

> 说明：Corepack 也能用，但遇到版本/权限问题时，独立安装更可控、排查更简单。

```bash
curl -fsSL https://get.pnpm.io/install.sh | sh -
```

把 pnpm 加入 PATH（若安装脚本已自动加过，这段可忽略）：

```bash
# ~/.bashrc 或 ~/.zshrc
export PNPM_HOME="$HOME/.local/share/pnpm"
export PATH="$PNPM_HOME:$PATH"
```

验证：

```bash
pnpm -v
```

---

## 4) 让 pnpm 使用共享 store（核心省空间点）

如果 store 在 `/opt`、而全局目录在 `$HOME/.local/...`，**跨设备**时有时会出链接类错误/性能问题（尤其容器、挂载盘、WSL 这类环境更容易）。如果你后面遇到奇怪的 link 报错，最省心的做法是把 store 放回 `$HOME` 同一盘。
```bash
pnpm config set store-dir "$HOME/.local/share/pnpm/store" --global
```

坑：`.bashrc` 设置 `PNPM_STORE_DIR`  无效。因为不是 pnpm 官方会读取/生效的环境变量，维护者也明确表示没有类似 `PNPM_STORE_DIR` 这样的环境变量，推荐用 `pnpm config set store-dir ...` 来控制 store 位置。

## 5) 强烈建议：不要共享 pnpm 全局安装目录（避免 EPERM）

**不要**把 `pnpm add -g` 的全局目录指向 `/opt/...`。  
原因：多人更新同一个全局目录会出现**权限冲突**、**版本漂移**，以及 **EPERM chmod**。

推荐做法：

- **共享 store**（省空间）
- **全局 CLI 工具每用户安装到自己的 PNPM_HOME**（最稳）

安装全局工具示例：

```bash
pnpm add -g @openai/codex @google/gemini-cli
```

---

## 6) 常见坑与排查（最简版）

### 6.1 共享目录权限不生效

```bash
groups
newgrp nodejs
ls -ld /opt/nodejs /opt/nodejs/pnpm-store
# 期望：drwxrwsr-x（有 s 表示 setgid 生效）
```

### 6.2 pnpm 没走共享 store

```bash
echo $PNPM_STORE_DIR
pnpm store path
```

---

## 7) 一键验收

```bash
command -v nvm && nvm --version
node -v && npm -v
pnpm -v
pnpm store path
```


## pnpm并发导致内存溢出OOM

**原因**：node每个进程几个G内存，多进程叠加 → 内存 + swap 全部打满 → 系统卡死。另外，大量小文件导致频繁扫描，硬盘被打满，性能下降。

### 解决方案
1. WSL优化内存：  
在Windows下，%UserProfile%\.wslconfig
```
[wsl2]
swap=32GB
swapFile=C:\\wsl-swap.vhdx

[experimental]
# 缓慢回收内存
autoMemoryReclaim=gradual
```


2. 限制每个 Node.js 进程最多只能使用 4GB 的堆内存。  
在 .bashrc 中添加：
```
export NODE_OPTIONS="--max-old-space-size=4096"
```

3. 限制 pnpm 并发（尤其是 pnpm 10 的自适应网络并发可能更激进）：
```
pnpm config set network-concurrency 4
pnpm config set child-concurrency 2
pnpm config set fetch-retries 2
pnpm config set prefer-offline true
```

4. 告诉AI Coding工具：pnpm 固定使用全局 store（不要放项目里），否则频繁扫描海量小文件容易卡死。如果 store 在项目目录，以下程序都会把这些文件一起扫一遍：
	- 每次 install / script 执行
	- agent 扫描项目目录
	- VSCode 文件监听
	- git 状态检测

5. 最后在内存紧张时，杀死耗内存大户，保住系统不死：
```
sudo apt update
sudo apt install -y earlyoom
sudo systemctl enable --now earlyoom
```