---
name: xray-relay-deploy
description: Install and remotely operate the user's xray-relay one-click VLESS+REALITY relay script (superchaospc/xray-relay, xray_deploy.sh) on a VPS over SSH. Use whenever the user wants to deploy, set up, or manage a VLESS/REALITY relay or 中转节点 on a VPS; add/delete/rename relay nodes (residential SOCKS5 or VPS direct, single or batch); view relay status, traffic stats, or run diagnostics; change a node's port; update or restart Xray; configure monitoring/email alerts; or uninstall. Trigger on phrases like "在 <vps> 上装一下中转脚本", "给 japan 加几个直连节点", "看下 vps1 的流量", "部署 xray 中转", "批量加住宅节点", even when the script name isn't mentioned. This skill drives the script's interactive menu non-interactively via SSH; it does NOT reimplement the relay logic.
---

# xray-relay Deploy & Operate

Install and operate the user's own relay script `xray_deploy.sh` (repo `superchaospc/xray-relay`, which the user maintains) on a remote VPS over SSH. The script is a single interactive bash menu with 16 functions. This skill installs it and drives any function **non-interactively** by feeding stdin over SSH.

This is the user's own infrastructure and their own script — operating it on their VPS is authorized. Treat node deletion, port changes, and uninstall as the only sensitive parts (see Destructive operations).

## How the script behaves (the mechanism that makes this work)

`xray_deploy.sh` loops a main menu forever, reading every answer through a `prompt_read` helper. The key fact: **when stdin hits EOF, `prompt_read` prints `输入流已结束，操作取消` and exits 0 cleanly.** So you can pipe a fixed sequence of answers; once they run out, the script exits on its own. There is no CLI/flag interface — stdin feeding is the only non-interactive path.

The script **requires root** (it exits if `id -u != 0`) and **systemd**. SSH must land as root. The user's `~/.ssh/config` aliases generally log in as root; if a target logs in as a non-root user, wrap the remote command in `sudo` and confirm with the user first.

## Step 1 — Pick the VPS target

Support both connection styles:
- **Preferred — SSH config alias.** The user keeps aliases in `~/.ssh/config` (e.g. `japan`, `vps1`, `germany`, `jabbar`). When the user names one, use `ssh <alias> ...`. If they say "my VPS" ambiguously and several aliases exist, ask which one.
- **Fallback — explicit host.** If no alias, ask for IP, user, and key path, then use `ssh -i <key> <user>@<ip> ...`.

Throughout this skill, `<target>` means whichever form applies.

## Step 2 — Install the script on the VPS

Pull the maintained `main` version to `/root` and make it executable:

```bash
ssh <target> 'curl -fsSL https://raw.githubusercontent.com/superchaospc/xray-relay/main/xray_deploy.sh -o /root/xray_deploy.sh && chmod +x /root/xray_deploy.sh && echo INSTALLED'
```

The script self-installs its own dependencies (`xray-core`, `python3`, `curl`, `qrencode`, etc.) during preflight and on first use, so no extra setup is needed.

## Step 3 — Drive a menu function

The universal contract: send the menu number followed by that function's answers, each on its own line. Let EOF end the run.

```bash
printf '<menu#>\n<answer1>\n<answer2>\n' | ssh <target> 'bash /root/xray_deploy.sh' 2>&1
```

Example — view status on `japan` (menu 5, no answers needed):

```bash
printf '5\n' | ssh japan 'bash /root/xray_deploy.sh' 2>&1
```

Example — deploy a direct-only starter node on a fresh VPS (menu 1 → `done` → `y` → empty name = `VPS-Direct`):

```bash
printf '1\ndone\ny\n\n' | ssh japan 'bash /root/xray_deploy.sh' 2>&1
```

**The exact input sequence for every menu item — including node formats, batch input, and the monitor sub-menu — is in `references/menu-actions.md`. Read it before driving anything other than the read-only items (5/6/7).** The script is actively maintained and prompts can change; if a run's output shows an unexpected prompt, `ssh <target> 'grep -n -A30 "<function_name>()" /root/xray_deploy.sh'` to read the current prompt order, or check the local repo at `~/xray-relay/xray_deploy.sh`.

### Preflight caveat

`preflight_check` prompts only in one case: port 443 is occupied by a **non-xray** process. If that fires, it eats the first stdin line (your menu number). This won't happen on a fresh VPS or one already running xray. If a run behaves as if the menu choice was swallowed, prepend one extra newline (`printf '\n1\n...'`) to absorb that confirmation, or inspect with a bare interactive check first.

## Step 4 — Retrieve results

The script writes outputs to fixed files; read them directly rather than scraping terminal QR codes:

- **VLESS links:** `ssh <target> 'cat /root/xray_nodes_info.txt'`
- **Base64 subscription:** `ssh <target> 'cat /root/xray_subscription.txt'`
- **Live config:** `/usr/local/etc/xray/config.json`

After any node-changing operation, confirm success by re-reading `xray_nodes_info.txt` or running menu 5 (status). Report the resulting links back to the user.

### Step 4b — Shareable exports (PDF / HTML / PNG / Excel)

When the user wants the links and QR codes as files to hand off — a printable PDF, an HTML page to open on a phone, individual QR PNGs, or an Excel inventory — pull the link file down and run the bundled exporter **locally** (it has the Python libs; the VPS need not). The exporter scans for `vless://` lines and ignores the surrounding `=== 名称 ===` / `端口:` / `出口:` decoration, so the remote info file feeds straight in:

```bash
ssh <target> 'cat /root/xray_nodes_info.txt' | python3 scripts/export_links.py --out ./exports
```

Default writes all four formats: `exports/qr/<name>.png` (one QR per node), `exports/lines.html` (self-contained gallery with QR inlined), `exports/lines.pdf` (printable, one node per row), `exports/lines.xlsx` (table + embedded QR). Narrow with `--format pdf,xlsx`. The script needs `segno reportlab openpyxl pillow` (`pip install --user …`); any missing lib skips just that one format. Run `python3 scripts/export_links.py --help` for details.

The QR matters here for the same reason the script renders one in-terminal: IPv6 brackets, `#` and `&` in a vless link get mangled when copy-pasted through chat apps — scanning the PNG sidesteps that.

## Destructive operations — confirm before sending

These change a live relay. **Stop, show the user exactly what you're about to run and on which `<target>`, and get explicit confirmation in the conversation before piping anything.** Do not auto-run them as part of a broader request.

- `3` delete node, `16` batch delete nodes — removes nodes (clients lose access)
- `4` change port — moves a live listening port
- `11` uninstall — removes Xray entirely (asks `y/n`, then offers to remove `/swapfile`)
- `10` → `c` stop monitoring
- `8` update Xray, `9` restart Xray — restart the service (brief disconnect); confirm if the relay is in active use

Read-only / additive operations (5 status, 6 traffic, 7 diagnostics, 2/12/13/14 add nodes, 15 rename, 10 a/b/d/e) can be run once the user has asked for them, without a separate confirmation step.

## Operating model

You are an **on-demand operator**, not an auto-runner. Install when asked, then perform whichever single function the user requests, gathering its parameters from the user (or sensible defaults documented in `references/menu-actions.md`). Never iterate through "all" menu items on your own — several are destructive and that would undo the user's setup.
