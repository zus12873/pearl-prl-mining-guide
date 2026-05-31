<!--
title: Deployment runbook — Pearl mining on WSL2 + NVIDIA GPU
description: Verified, copy-pasteable step-by-step to mine Pearl (PRL) on Windows/WSL2 with one NVIDIA GPU — GPU check, wallet creation, the SPV/--addpeer DNS workaround, checksum-verified miner, tmux logging, and monitoring. Includes a troubleshooting table for every error hit during a real 2026-05-31 deploy.
language: [en, zh]
updated: 2026-05-31
tags: [runbook, wsl, alpha-miner, oyster, prlctl, tmux, spv]
verified: RTX 3090 / WSL2 Ubuntu / 2026-05-31
-->

# 02 · Deployment runbook / 部署实操

> Replace placeholders: `prl1p<your-address>`, `<rpc-user>`, `<rpc-pass>`. Run everything **inside your own WSL terminal**. Verify every downloaded binary's SHA256 before running it.
> 替换占位符：`prl1p<your-address>`、`<rpc-user>`、`<rpc-pass>`。所有命令在**你自己的 WSL 终端**里跑。运行任何下载的二进制前先核对 SHA256。

Pipeline / 全流程：`WSL GPU check → wallet → payout address → miner → monitor`.

---

## A. WSL2 + GPU prerequisites / 前置

In Windows PowerShell / 在 Windows PowerShell：
```powershell
wsl --update
wsl --shutdown
```
Install/upgrade the **NVIDIA Windows driver**. **Do NOT** install a Linux GPU driver or the `cuda`/`cuda-drivers` packages inside WSL — the CUDA driver comes from Windows.
安装/升级 **NVIDIA Windows 驱动**。WSL 内 **不要** 装 Linux 显卡驱动或 `cuda`/`cuda-drivers` 包——CUDA driver 来自 Windows。

Verify the GPU is visible in WSL / 在 WSL 里确认 GPU（`nvidia-smi` is not on PATH）:
```bash
ls /usr/lib/wsl/lib/libcuda.so*          # should exist / 应存在
/usr/lib/wsl/lib/nvidia-smi              # should show your GPU / 应显示你的显卡
```
Install base tools / 装基础工具：
```bash
sudo apt update && sudo apt install -y curl ca-certificates tmux
```
> ✅ Verify: `nvidia-smi` shows the card with free VRAM. The miner uses `libcuda.so` directly, so no Linux CUDA toolkit is needed.
> ✅ 验证：`nvidia-smi` 能看到显卡且有空闲显存。矿工直接用 `libcuda.so`，无需 Linux CUDA。

---

## B. Wallet & payout address / 钱包与收款地址

The official wallet is a CLI: `oyster` (daemon, holds keys/signs) + `prlctl` (control) + `pearld` (full node, not needed here). Source: GitHub `pearl-research-labs/pearl`.
官方钱包是 CLI：`oyster`（守护进程，存私钥/签名）+ `prlctl`（控制）+ `pearld`（全节点，这里用不到）。来源：GitHub `pearl-research-labs/pearl`。

### B1. Download + checksum-verify / 下载 + 校验
```bash
mkdir -p ~/pearl-wallet && cd ~/pearl-wallet
curl -fL -o pearl-cli.tar.gz \
  https://github.com/pearl-research-labs/pearl/releases/download/pearl-wallet-v1.0.0/go-binaries-linux-amd64-v1.0.2.tar.gz

# Verify (expected SHA256 for linux-amd64 v1.0.2):
EXPECT=56ae87bdd2913ae8e030d32e820240696cce41cbda4b84bfd5a4968196565a47
GOT=$(sha256sum pearl-cli.tar.gz | cut -d' ' -f1)
[ "$GOT" = "$EXPECT" ] && echo "✅ OK" || echo "❌ MISMATCH — STOP"

tar -xzf pearl-cli.tar.gz
find . -type f \( -name oyster -o -name prlctl -o -name pearld \)   # locate binaries
```
> ✅ Verify: `✅ OK`, and `oyster`/`prlctl`/`pearld` exist (in `~/pearl-wallet`). If they're in a subdir, adjust `./` paths below.

### B2. Create the wallet (you hold the secret) / 创建钱包（密钥你保管）
```bash
cd ~/pearl-wallet
./oyster -u <rpc-user> -P '<rpc-pass>' --create
```
Answer the prompts / 按提示回答：
- Private passphrase → set your own, remember it. / 私钥口令 → 自己设并记住。
- **"additional layer of encryption for public data?" → `n`** (else startup needs `--walletpass`). / 这一项回答 **`n`**（否则启动要带 `--walletpass`）。
- "existing wallet seed?" → `n`.
- **Mnemonic shown → write it down OFFLINE.** Never screenshot, never paste it anywhere. Then type `OK`.
  **显示助记词 → 离线手抄。** 不截图、不粘贴到任何地方。然后输 `OK`。
- See `The wallet has been created successfully.`

> 🔐 `<rpc-user>/<rpc-pass>` are LOCAL RPC credentials (127.0.0.1 only), **not** the mnemonic. The mnemonic + private passphrase are what protect your funds. If a mnemonic is ever exposed, treat the wallet as compromised: `rm -rf ~/.oyster` and recreate.
> 🔐 `<rpc-user>/<rpc-pass>` 只是本机 RPC 账号密码（仅 127.0.0.1），**不是**助记词。助记词+私钥口令才是资金安全的关键。助记词一旦暴露即视为泄露：`rm -rf ~/.oyster` 重建。

### B3. Start the wallet — with the WSL DNS workaround / 启动钱包（含 WSL DNS 绕坑）
`getnewaddress` needs an active chain backend. WSL's default DNS **cannot resolve** the seeder subdomains `x49.seederN.pearlresearch.ai`, so SPV peer discovery fails. **Fix: feed peer IPs directly** (the base `seeder1.pearlresearch.ai` resolves publicly; peers listen on `44108`).
`getnewaddress` 需要活跃的链后端。WSL 默认 DNS **解析不了** seeder 子域 `x49.seederN.pearlresearch.ai`，导致 SPV 找不到节点。**解法：直接喂节点 IP**（基础域 `seeder1.pearlresearch.ai` 能解析；节点端口 `44108`）。

Get current peer IPs (run once) / 取当前节点 IP（跑一次）：
```bash
curl -s "https://dns.google/resolve?name=seeder1.pearlresearch.ai&type=A" \
 | grep -o '"data":"[0-9.]*"' | grep -o '[0-9.]*' | head -6
```
Start the wallet daemon (Window A, keep it open) / 启动钱包守护（窗口A，开着别关）:
```bash
cd ~/pearl-wallet
./oyster -u <rpc-user> -P '<rpc-pass>' --noservertls --usespv \
  --addpeer=<IP1> --addpeer=<IP2> --addpeer=<IP3> --addpeer=<IP4>
```
Wait for log lines like `New valid peer … :44108` and `Syncing to block height …`.
等日志出现 `New valid peer … :44108`、`Syncing to block height …`。

### B4. Generate a payout address / 生成收款地址
In a second WSL window / 另开一个 WSL 窗口：
```bash
cd ~/pearl-wallet
./prlctl -u <rpc-user> -P '<rpc-pass>' -s localhost:44207 --notls getnewaddress
# → prl1p<your-address>
```
> ✅ Verify: output starts with `prl1p…`. That's your `--address` for mining. After this you may `Ctrl+C` the daemon — **mining does not need the wallet online** (the pool credits the address). To view balance later, restart the daemon the same way or use the official desktop wallet.
> ✅ 验证：输出以 `prl1p…` 开头，这就是挖矿命令里的 `--address`。拿到后可 `Ctrl+C` 关掉守护进程——**挖矿不需要钱包在线**（矿池按地址记账）。以后查余额再同样启动守护进程，或用官方桌面钱包。

---

## C. Miner / 矿工

### C1. Download + checksum-verify / 下载 + 校验
```bash
mkdir -p ~/pearl-mining/logs && cd ~/pearl-mining
curl -fL -o alpha-miner \
  https://github.com/AlphaMine-Tech/alpha-miner/releases/download/v1.7.6-beta/alpha-miner

EXPECT=c84396e2ff4ded14a8c83cd253761b46dd40927c5c43a39a20aac9ff8bdfbfe5
GOT=$(sha256sum alpha-miner | cut -d' ' -f1)
[ "$GOT" = "$EXPECT" ] && echo "✅ OK" || echo "❌ MISMATCH — STOP"
chmod +x alpha-miner
```
> Prefer the GitHub release (checksummed) over the pool's `downloads/` endpoint. Re-check the latest tag/hash on the repo before trusting this pinned value.
> 优先用 GitHub release（有校验值），而非矿池 `downloads/` 端点。信任此固定值前，先到仓库核对最新 tag/hash。

### C2. Run in tmux (survives SSH/terminal close) / 在 tmux 里运行（断连不停）
```bash
tmux new -s pearl -d
tmux send-keys -t pearl "cd ~/pearl-mining && ./alpha-miner \
  --pool stratum+tcp://us2.alphapool.tech:5566 \
  --address prl1p<your-address> \
  --worker wsl-3090 \
  --status-interval 60 2>&1 | tee -a logs/miner.log" Enter
```
- Pools / 矿池: `us2` / `sg1` / `eu1` `.alphapool.tech:5566`.
- Static difficulty (optional): add `--password 'x;d=32768'` (alpha-miner README's RTX-3090 hint; AlphaPool site says `262144`). Either way the pool's **vardiff auto-adjusts** (observed ~1.1M), so this is non-critical.
  静态难度（可选）：加 `--password 'x;d=32768'`（alpha-miner README 对 3090 的建议；AlphaPool 网站写 `262144`）。无论填不填，矿池 **vardiff 都会自动调**（实测约 1.1M），不关键。
- Attach to watch / 看实时：`tmux attach -t pearl`（detach 不停：`Ctrl+B` 然后 `D`）。

> ✅ Verify (within ~60s): the log shows `found_candidate` → `submitted`, a `status … hashrate_th_s=…` line (~100–117 TH/s on a 3090), and `nvidia-smi` shows `alpha-miner` using the GPU.
> ✅ 验证（约 60 秒内）：日志出现 `found_candidate` → `submitted`、`status … hashrate_th_s=…`（3090 约 100–117 TH/s），且 `nvidia-smi` 显示 `alpha-miner` 占用 GPU。

> Tip: if `tee` didn't create the log (quoting), attach to the tmux pane to start logging without restarting the miner: `tmux pipe-pane -t pearl -o 'cat >> ~/pearl-mining/logs/miner.log'`.
> 提示：若 `tee` 没生成日志（引号问题），用 `tmux pipe-pane` 不重启矿工就开始记日志：`tmux pipe-pane -t pearl -o 'cat >> ~/pearl-mining/logs/miner.log'`。

---

## D. Monitoring / 监控

Local (read-only) / 本地（只读）:
```bash
pgrep -af "[a]lpha-miner"                                   # process alive?
tail -n 20 ~/pearl-mining/logs/miner.log                    # latest shares/hashrate
/usr/lib/wsl/lib/nvidia-smi --query-gpu=utilization.gpu,power.draw,temperature.gpu --format=csv,noheader
tmux capture-pane -t pearl -p -S -50                        # live pane buffer
```
Pool-side (authoritative) / 矿池侧（权威）— `1 PRL = 1e8 grain`:
```bash
curl -s "https://pearl.alphapool.tech/api/miner/prl1p<your-address>"
# key fields: balance_prl (pending), total_paid_prl (paid), workers[].online, shares24h
```
Web dashboard / 网页面板: `https://pearl.alphapool.tech/` → paste your address. Block explorers: `explorer.pearlresearch.ai`, `pearlchain.live`.
> ⚠️ The pool API is **Cloudflare-cached** and lags a few minutes — your real pending balance is ≥ the displayed value. Confirm the miner is live via the **local** log, not the API.
> ⚠️ 矿池 API 被 **Cloudflare 缓存**、滞后几分钟——真实 pending ≥ 显示值。确认矿工是否在跑要看**本地**日志，而非 API。

---

## E. Troubleshooting / 排错（本次实测遇到的全部）

| Symptom / 症状 | Cause / 根因 | Fix / 修法 |
|---|---|---|
| `nvidia-smi: command not found` | not on PATH in WSL | use `/usr/lib/wsl/lib/nvidia-smi`; `libcuda.so*` present ⇒ GPU OK |
| `sha256sum -c … no file was verified` | downloaded file renamed vs checksums.txt entry | compare hash manually against the expected value above |
| `invalid passphrase for master public key` | wallet was created WITH a public passphrase | `rm -rf ~/.oyster`, recreate answering `n` to public encryption (or pass `--walletpass`) |
| `-4: blockchain RPC is inactive` | wallet has no active chain backend | start `oyster` with `--usespv --addpeer=<IP>` (see B3) |
| `DNS discovery failed … x49.seederN… no such host` | WSL DNS can't resolve seeder subdomains | feed peer IPs from base `seeder1.pearlresearch.ai` via `--addpeer` |
| `401 Unauthorized` from `prlctl` | `-u/-P` don't match the running `oyster` | use identical `-u/-P` in both commands |
| `prlctl` warns `.pearld/pearld.conf not found` | harmless (it looks for node cert) | ignore; `--notls` + `-u/-P` is enough |
| no `logs/miner.log` but GPU busy | `tee` pipe lost to quoting in `send-keys` | `tmux pipe-pane -t pearl -o 'cat >> ~/pearl-mining/logs/miner.log'` |

---

Next: how to actually turn mined PRL into money (and why you can't send it to Binance) → [03 · Tokens & cash-out](03-tokens-and-cashout.md).
下一步：怎么把挖到的 PRL 变成钱（以及为什么不能充币安）→ [03 · 代币与变现](03-tokens-and-cashout.md)。
