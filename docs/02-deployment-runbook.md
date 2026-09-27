<!--
title: Deployment runbook — Pearl mining on WSL2 + NVIDIA GPU
description: Verified step-by-step for the live rig — WSL GPU, wallet, PeakMiner 2.16.5 on Kryptex in tmux, local and pool monitoring, and how pool payouts reach the wallet. The May 2026 AlphaMiner/AlphaPool steps are an appendix.
language: [en, zh]
updated: 2026-09-27
tags: [runbook, wsl, peakminer, kryptex, oyster, prlctl, tmux, spv]
verified: RTX 5080 / WSL2 Ubuntu / PeakMiner 2.16.5 + Kryptex / 2026-09-27
-->

# 02 · Deployment runbook / 部署实操

> Replace placeholders: `prl1p<your-address>`, `<rpc-user>`, `<rpc-pass>`. Run everything **inside your own WSL terminal**. Verify every downloaded binary's SHA256 before running it.
> 替换占位符：`prl1p<your-address>`、`<rpc-user>`、`<rpc-pass>`。所有命令在**你自己的 WSL 终端**里跑。运行任何下载的二进制前先核对 SHA256。

Pipeline / 全流程：`WSL GPU check → wallet → payout address → PeakMiner on Kryptex → monitor → automatic pool payout → cash-out in docs/03`.

Current rig / 当前矿机：one RTX 5080, PeakMiner 2.16.5, tmux session `pearl`, worker name you choose, Kryptex PPS+. The AlphaMiner appendix at the bottom is retired.

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

## C. Miner — PeakMiner on Kryptex / 矿工

Pool facts checked against `GET https://pool.kryptex.com/prl/api/v1/pool/info` on 2026-09-27:
2026-09-27 对照矿池接口核对过的参数：

| Item / 项 | Value / 值 |
|---|---|
| Scheme / 模式 | PPS+ by default (pay per share, not "wait until the pool finds a block"). SOLO is a different username prefix `solo:` |
| Fee / 费率 | PPS+ **2%**. SOLO **1%**. PeakMiner adds its own **2%** dev fee |
| Stratum | `prl-hk.kryptex.network:7048` (this rig's first choice), `prl-sg.kryptex.network:7048`, `prl.kryptex.network:7048`. SSL is port **8048** on the same hosts. Other regions: `prl-eu`, `prl-us`, `prl-br`, `prl-ru`, `prl-ae` |
| Username / 用户名 | `prl1p<your-address>/<worker>` |
| Payout / 打款 | Automatic, about once an hour, when **confirmed** balance ≥ threshold. Minimum and default **1 PRL**. Maturation **100 blocks** (`block_time` 194 s, so about 5.4 h). The pool pays the transaction fee |

### C1. The binary this rig runs / 这台机器在跑的二进制
```bash
mkdir -p ~/pearl-mining/logs
cd ~/pearl-mining/peakminer-2.16.5
# File that was running on 2026-09-27:
EXPECT=5dc4b927fb91442a66a2e02c1636f629d4042a03a5c1523cd841d6ece66e0beb
GOT=$(sha256sum peakminer-2.16.5-linux-x86_64 | cut -d' ' -f1)
[ "$GOT" = "$EXPECT" ] && echo "✅ OK" || echo "❌ MISMATCH — STOP"
```
Re-check whatever you download against the publisher before trusting a new build. Kryptex's worker API on 2026-09-27 advertised agent `peakminer/2.17.2` while this rig's 2.16.5 was still getting `accepted` shares. A pool banner saying "old miners submit invalid shares" means: if shares start coming back invalid, upgrade; do not upgrade in the middle of a healthy session just because the banner is up.
重新下载时先跟发布方核对哈希。2026-09-27 Kryptex 工人接口显示更新的 `peakminer/2.17.2`，但这台 2.16.5 仍在收到 `accepted`。矿池横幅写“旧矿工份额无效”的含义是：份额开始被拒再升级；会话正常时不要因为横幅就中途换版本。

### C2. Launch script / 启动脚本
`~/pearl-mining/peakminer-2.16.5/run-kryptex.sh` (run from that directory):
```bash
#!/bin/bash
cd /home/huanmeng/pearl-mining/peakminer-2.16.5
exec ./peakminer --coin pearl \
  -o prl-hk.kryptex.network:7048 \
  -o prl-sg.kryptex.network:7048 \
  -o prl.kryptex.network:7048 \
  -u 'prl1p<your-address>/<worker>' \
  -f /home/huanmeng/pearl-mining/logs/peakminer.log \
  --log-append \
  --gpu-fan 80
```
`--gpu-fan` / `--gpu-fan-target` need root NVML. Without passwordless sudo the miner logs `fan set … skipped` and continues. Pin the fan from Windows (MSI Afterburner) if you want a fixed duty; the driver curve is what you get otherwise.
`--gpu-fan` 要 root NVML。没有免密 sudo 时日志会写 `fan set … skipped`，矿工照跑。要固定转速就在 Windows 上用 MSI Afterburner；否则就是驱动曲线。

### C3. Run in tmux / 放进 tmux
```bash
tmux new-session -d -s pearl -c ~/pearl-mining/peakminer-2.16.5 \
  ~/pearl-mining/peakminer-2.16.5/run-kryptex.sh
```
- Attach / 查看：`tmux attach -t pearl`（detach：`Ctrl+B` 然后 `D`）。
- Stop / 停止：`tmux kill-session -t pearl`. Do **not** `pkill -f peakminer` from a command line that itself contains that string — the pattern can match the shell wrapper.
  停止用 `tmux kill-session -t pearl`。不要在命令行里 `pkill -f peakminer`，模式会把这条远程 shell 自己杀掉。
- Full-load mining occupies the only GPU. Stop session `pearl` before a GPU job such as ComfyUI.
  满载挖矿占住唯一的显卡。跑 ComfyUI 之类的 GPU 任务前先停掉 `pearl`。

> ✅ Verify: within a minute the pane shows `connected prl-hk.kryptex.network:7048` (or the next pool), `vardiff`, and a hashrate line. `curl -s http://127.0.0.1:4068/summary` returns JSON with `"connected": true`. `nvidia-smi` shows the GPU near 100% and ~360 W on this 5080. An `accepted` line is the proof a share landed; the first one can take longer than the ETA.
> ✅ 验证：一分钟内窗格出现 `connected prl-hk.kryptex.network:7048`（或下一个池）、`vardiff` 和算力行。`curl -s http://127.0.0.1:4068/summary` 里 `"connected": true`。这张 5080 上 `nvidia-smi` 接近 100%、约 360 W。出现 `accepted` 才说明份额被收下；第一份可能比预计时间久。

---

## D. Monitoring / 监控

Local / 本机：
```bash
tmux ls
tmux capture-pane -t pearl -p -S -40
curl -s http://127.0.0.1:4068/summary
tail -n 30 ~/pearl-mining/logs/peakminer.log
LD_LIBRARY_PATH=/usr/lib/wsl/lib /usr/lib/wsl/lib/nvidia-smi \
  --query-gpu=utilization.gpu,power.draw,temperature.gpu,fan.speed --format=csv,noheader
```
Log timestamps are UTC. The host clock is CST.
日志时间是 UTC。机器时钟是北京时间。

Kryptex (what will actually be paid) / Kryptex（真正会打出来的数）：
```bash
curl -s "https://pool.kryptex.com/prl/api/v1/miner/balance/prl1p<your-address>"
# confirmed  — matured, counts toward the 1 PRL payout
# unconfirmed — still inside the 100-block maturation window
# threshold   — default 1
curl -s "https://pool.kryptex.com/prl/api/v1/miner/payouts/prl1p<your-address>"
curl -s "https://pool.kryptex.com/prl/api/v3/miner/workers/prl1p<your-address>"
```
Dashboard: open `https://pool.kryptex.com/prl` and paste the address.
面板：打开 `https://pool.kryptex.com/prl`，贴地址。

On-chain, after a payout / 打款之后的链上余额：
```bash
curl -s "https://pearltrack.io/api/v1/address/prl1p<your-address>"
# balancePrl, totalReceivedPrl, totalSentPrl
```
The wallet does not need to be running for either API. `balancePrl` is what you can send. Kryptex `confirmed` under 1 PRL is not in the wallet yet.
查这两个接口都不用开着钱包。`balancePrl` 才是能转出的。Kryptex 里不到 1 PRL 的 `confirmed` 还没进钱包。

---

## E. Pool → wallet / 矿池进钱包

You do not withdraw from Kryptex by hand.
不用在 Kryptex 上手动点提现。

1. Keep the worker submitting shares. New rewards show up as `unconfirmed` and become `confirmed` after 100 blocks (~5.4 h at the 194 s target).
   工人继续交份额。新收益先在 `unconfirmed`，100 个块之后（按 194 秒目标大约 5.4 小时）变成 `confirmed`。
2. About once an hour, if `confirmed` ≥ the threshold (minimum and default 1 PRL), Kryptex sends the payout to the mining address. Finished payouts have a txid. The pool pays the fee.
   大约每小时一次：`confirmed` 达到门槛（最低也是默认的 1 PRL）就打到挖矿地址。完成的记录带 txid。手续费矿池出。
3. To change the threshold, use the pool page → that address → Settings. Kryptex asks for the IP of a worker that has submitted shares, and the threshold cannot be set below 1 PRL. `POST /prl/api/v1/miner/settings/<address>` with `threshold` and `ip_address` is the same action. Do not point the setting at a different wallet.
   改门槛：矿池页面 → 该地址 → Settings。Kryptex 会要一台交过份额的工人的 IP，门槛不能低于 1 PRL。接口是 `POST /prl/api/v1/miner/settings/<address>`，字段 `threshold` 和 `ip_address`。不要改成另一个钱包。
4. Kryptex's own article says an **inactive** wallet's pool balance is deleted after **90 days**. If you stop mining with less than 1 PRL confirmed, mine the remainder (or you lose that dust). Coins already in your wallet are not affected.
   Kryptex 自己的文章写：**不活跃**钱包的矿池余额 **90 天**后删除。确认余额不到 1 PRL 就停挖的话，把零头挖够，否则这点灰会没。已经在你钱包里的币不受影响。
5. A balance left on the **retired AlphaPool** (PPLNS) is not part of this payout. It pays only when that pool finds a block and clears its own minimum. Check `https://pearl.alphapool.tech/api/miner/prl1p<your-address>` (`balance_prl`). Do not point the live miner back at AlphaPool to chase it.
   **已停用的 AlphaPool**（PPLNS）上的余额不走上面这条打款。要等那个池自己出块、并达到它自己的最低额。查 `https://pearl.alphapool.tech/api/miner/prl1p<your-address>` 的 `balance_prl`。不要为了这点余额把正在跑的矿工改回 AlphaPool。

Turning wallet coins into USDT is the next page, not a pool button. → [03 · Tokens & cash-out](03-tokens-and-cashout.md).
把钱包里的币换成 USDT 是下一页，不是矿池按钮。→ [03 · 代币与变现](03-tokens-and-cashout.md)。

---

## F. Troubleshooting / 排错

| Symptom / 症状 | Cause / 根因 | Fix / 修法 |
|---|---|---|
| `nvidia-smi: command not found` | not on PATH in WSL | use `/usr/lib/wsl/lib/nvidia-smi`; `libcuda.so*` present ⇒ GPU OK |
| `fan set to 80% skipped — NVML denied` | `--gpu-fan` needs root | ignore, or pin the fan in MSI Afterburner on Windows. Mining continues |
| `sha256sum` mismatch | file is not the pinned 2.16.5 binary | stop; do not run it |
| `invalid passphrase for master public key` | wallet was created WITH a public passphrase | `rm -rf ~/.oyster`, recreate answering `n` to public encryption (or pass `--walletpass`) |
| `-4: blockchain RPC is inactive` | wallet has no active chain backend | start `oyster` with `--usespv --addpeer=<IP>` (see B3) |
| `DNS discovery failed … x49.seederN… no such host` | WSL DNS can't resolve seeder subdomains | feed peer IPs from base `seeder1.pearlresearch.ai` via `--addpeer` |
| `401 Unauthorized` from `prlctl` | `-u/-P` don't match the running `oyster` | use identical `-u/-P` in both commands |
| `prlctl` warns `.pearld/pearld.conf not found` | harmless (it looks for node cert) | ignore; `--notls` + `-u/-P` is enough |
| Kryptex `confirmed` sits under 1 and never arrives on-chain | threshold not reached, or still `unconfirmed` | keep mining through maturation; payout is automatic at 1 PRL |
| SSH to the rig dies during banner exchange on `192.168.1.x` | Clash TUN (`128.0.0.0/1`) swallows that LAN range | use `huanmeng-win.local` and confirm the address with `ping` before reusing an old overlay IP |
| Killing the miner also kills your shell | `pkill -f` matched the wrapper command line | `tmux kill-session -t pearl` |

---

## Appendix — retired AlphaMiner / AlphaPool / 附录：已停用

Do not start this. It is the 2026-05-31 RTX 3090 run, kept so the old commands are not mistaken for the live ones. AlphaPool is PPLNS on `us2|sg1|eu1.alphapool.tech:5566`. The pinned `alpha-miner` v1.7.6-beta SHA256 is `c84396e2ff4ded14a8c83cd253761b46dd40927c5c43a39a20aac9ff8bdfbfe5`. v1.7.6 was not credited by that pool on the RTX 5080.
不要再启动。这是 2026-05-31 的 RTX 3090 记录，留下来是为了避免把旧命令当成现在的。AlphaPool 是 PPLNS，地址 `us2|sg1|eu1.alphapool.tech:5566`。`alpha-miner` v1.7.6-beta 的 SHA256 是 `c84396e2ff4ded14a8c83cd253761b46dd40927c5c43a39a20aac9ff8bdfbfe5`。这个版本在 RTX 5080 上没被该池记份额。

```bash
tmux new -s pearl -d
tmux send-keys -t pearl "cd ~/pearl-mining && ./alpha-miner \
  --pool stratum+tcp://us2.alphapool.tech:5566 \
  --address prl1p<your-address> \
  --worker wsl-3090 \
  --status-interval 60 2>&1 | tee -a logs/miner.log" Enter
curl -s "https://pearl.alphapool.tech/api/miner/prl1p<your-address>"
```
That pool API is Cloudflare-cached and lags. `balance_prl` there is unpaid PPLNS credit, not a wallet balance.
该池 API 有 Cloudflare 缓存、会滞后。那里的 `balance_prl` 是还没结算的 PPLNS，不是钱包余额。

---

Next: send coins that are already in the wallet → [03 · Tokens & cash-out](03-tokens-and-cashout.md).
下一步：转出已经在钱包里的币 → [03 · 代币与变现](03-tokens-and-cashout.md)。
