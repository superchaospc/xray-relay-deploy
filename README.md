# xray-relay-deploy

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
![Claude Code](https://img.shields.io/badge/Claude%20Code-skill-d97757)
![Codex](https://img.shields.io/badge/Codex-skill-412991)
![Shell](https://img.shields.io/badge/shell-bash-4EAA25?logo=gnu-bash&logoColor=white)

**English** | [中文说明](#中文说明)

A **Claude Code / Codex skill** that installs and remotely operates the
[`xray-relay`](https://github.com/superchaospc/xray-relay) one-click
**VLESS + REALITY** relay script (TCP + XTLS-Vision transport) on a VPS over SSH. It
drives the script's 16-item interactive menu **non-interactively** by feeding stdin over
SSH — it does **not** reimplement any relay logic.

This is the sibling of
[`xray-xhttp-relay-deploy`](https://github.com/superchaospc/xray-xhttp-relay-deploy),
which drives the XHTTP+REALITY variant. Same drive-the-menu mechanism; this one targets
the **TCP + XTLS-Vision** transport. (The XHTTP sibling shares the identical 16-item menu
and was live-tested 16/16 on a real VPS, which cross-validates these sequences.)

## What it does

Once installed, the skill lets the agent — on a VPS you name (SSH alias or host) —

- **deploy** a fresh VLESS+REALITY relay (residential SOCKS5 multi-hop or VPS-direct)
- **manage nodes**: add / delete / rename, single or batch (menus 2, 3, 12–16)
- **operate**: status, traffic, diagnostics, change port, update/restart Xray, monitoring/email alerts, uninstall
- **retrieve** the generated VLESS links and subscription

The mechanism: the script reads every prompt through `prompt_read`, which exits cleanly
on stdin EOF, so a fixed answer sequence drives any menu function. The exact sequence for
each menu item lives in [`references/menu-actions.md`](references/menu-actions.md).

## Install (Claude Code + Codex)

```bash
# 1. Clone into Claude Code's skills dir
git clone https://github.com/superchaospc/xray-relay-deploy \
  ~/.claude/skills/xray-relay-deploy

# 2. To use it in Codex too, run the bundled installer once
~/.claude/skills/xray-relay-deploy/install.sh
```

Claude Code and Codex share the same `SKILL.md` format — `install.sh` just symlinks the
skill into Codex's dirs (`~/.codex/skills`, `~/.agents/skills`). Self-contained, portable
(uninstalled agents skipped), idempotent. Restart Codex so it rescans.

The skill triggers on phrases like "在 vps 上装一下中转脚本", "给 japan 加几个直连节点",
"看下 vps1 的流量", "部署 xray 中转", "批量加住宅节点".

## Authorization & safety

This skill operates the user's **own** script on the user's **own** VPS — that's the
intended use. Destructive functions (delete / change-port / uninstall / fresh-install,
which overwrites a live config) are gated: the skill shows exactly what it will run and on
which target, and waits for confirmation before sending.

---

## 中文说明

一个 **Claude Code / Codex skill**,通过 SSH 在 VPS 上安装并远程操作
[`xray-relay`](https://github.com/superchaospc/xray-relay) 一键 **VLESS + REALITY** 中转脚本
(TCP + XTLS-Vision 传输)。它**非交互地**驱动脚本的 16 项交互菜单(通过 SSH 喂 stdin),
**不重新实现**任何中转逻辑。

它是 [`xray-xhttp-relay-deploy`](https://github.com/superchaospc/xray-xhttp-relay-deploy)
(驱动 XHTTP+REALITY 版)的姊妹 skill。驱动机制相同,这个针对 **TCP + XTLS-Vision** 传输。
(XHTTP 姊妹版共用完全相同的 16 项菜单,且已在真实 VPS 上 16/16 实测通过,反向印证了这里的序列。)

### 能做什么

装上后,agent 可以在你指定的 VPS(SSH 别名或主机)上:

- **部署**全新 VLESS+REALITY 中转(住宅 SOCKS5 多跳 或 VPS 直连)
- **管理节点**:单条/批量 添加、删除、改名(菜单 2、3、12–16)
- **运维**:状态、流量、诊断、改端口、更新/重启 Xray、监控邮件告警、卸载
- **取回**生成的 VLESS 链接和订阅

原理:脚本每个 prompt 都经 `prompt_read` 读取,stdin 到 EOF 时干净退出,所以一串固定答案就能
驱动任意菜单功能。每个菜单项的精确序列见
[`references/menu-actions.md`](references/menu-actions.md)。

### 安装(Claude Code + Codex)

```bash
# 1. clone 到 Claude Code 的 skills 目录
git clone https://github.com/superchaospc/xray-relay-deploy \
  ~/.claude/skills/xray-relay-deploy

# 2. 想在 Codex 里也能用,再跑一次自带安装脚本
~/.claude/skills/xray-relay-deploy/install.sh
```

CC 和 Codex 用同一种 `SKILL.md` 格式,`install.sh` 只是把 skill 软链进 Codex 的目录
(`~/.codex/skills`、`~/.agents/skills`)。自包含、可移植(没装的 agent 自动跳过)、幂等。
完成后重开 Codex 让它重新扫描。

触发词例如:"在 vps 上装一下中转脚本"、"给 japan 加几个直连节点"、"看下 vps1 的流量"、
"部署 xray 中转"、"批量加住宅节点"。

### 授权与安全

本 skill 操作的是你**自己的**脚本、你**自己的** VPS——这正是它的预期用途。破坏性功能(删节点 /
改端口 / 卸载 / 全新安装——后者会覆盖 live 配置)都有确认门槛:skill 会先展示将要执行的命令和
目标,经你确认后才发送。

## License

[MIT](LICENSE)
