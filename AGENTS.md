<!--
title: AGENTS.md — context & conventions for AI agents
description: Machine-oriented brief so an AI coding agent can safely continue this Pearl (PRL) mining project without re-discovering the gotchas.
language: [en, zh]
updated: 2026-09-27
-->

# AGENTS.md

Context and conventions for AI coding agents working in this repo. Humans: see [README.md](README.md).
给在本仓库工作的 AI 代理的上下文与约定。人类请看 [README.md](README.md)。

## What this repo is / 这是什么
A documentation-only, field-tested guide to mining **Pearl L1 (native PRL)** on Windows/WSL2 + one NVIDIA GPU. No application code; only Markdown under `docs/`.
仅文档、已实测的 **Pearl L1 原生 PRL** 挖矿指南（Windows/WSL2 + 单 NVIDIA 显卡）。无应用代码，只有 `docs/` 下的 Markdown。

## Hard safety rules / 安全硬规则
1. **Never** write a real seed phrase / mnemonic / private key / wallet password anywhere in this repo or in chat. Treat any leaked mnemonic as compromised → recreate the wallet.
   **绝不**把真实助记词/私钥/钱包密码写进本仓库或聊天。任何泄露的助记词视为已泄露 → 重建钱包。
2. Wallet addresses in docs are placeholders: `prl1p<your-address>`. Do not hardcode a real one.
   文档中的钱包地址用占位符 `prl1p<your-address>`，不要写死真实地址。
3. Do not commit binaries, `checksums.txt`, `.oyster/`, logs, or anything matched by [.gitignore](.gitignore).
   不要提交二进制、`checksums.txt`、`.oyster/`、日志，或被 [.gitignore](.gitignore) 命中的任何文件。
4. **Token disambiguation is the #1 correctness issue**: mined PRL = *Pearl Research L1* ≠ Binance *Perle (PRL)*. Never conflate them. See [docs/03](docs/03-tokens-and-cashout.md).
   **代币辨析是第一正确性问题**：挖的 PRL = *Pearl Research L1* ≠ 币安 *Perle (PRL)*。绝不混为一谈。

## Verified facts (live rig, 2026-09-27) / 已验证事实
- Miner: PeakMiner **2.16.5** in tmux session `pearl`, started by `~/pearl-mining/peakminer-2.16.5/run-kryptex.sh`. On-disk SHA256 of `peakminer-2.16.5-linux-x86_64`: `5dc4b927fb91442a66a2e02c1636f629d4042a03a5c1523cd841d6ece66e0beb`. Kryptex's worker API that day also advertised a newer agent string `peakminer/2.17.2`; 2.16.5 was still accepting shares.
- Pool: Kryptex Pearl, PPS+ fee **0.02**, SOLO **0.01**, `minpay`/`defpay` **1 PRL**, `blocks_for_maturation` **100**, `block_time` **194**. `GET https://pool.kryptex.com/prl/api/v1/pool/info`. Balance: `GET https://pool.kryptex.com/prl/api/v1/miner/balance/<address>` → `confirmed`, `unconfirmed`, `threshold`. Payouts are automatic; there is no manual withdraw in the happy path.
- Stratum order used on this rig: `prl-hk.kryptex.network:7048`, then `prl-sg`, then `prl.kryptex.network:7048`. Username `prl1p<your-address>/<worker>`. Local API `http://127.0.0.1:4068/summary`.
- Wallet CLI on the rig: `oyster` / `prlctl` **1.1.0** in `~/pearl-wallet`. SHA256: oyster `96c8ef4df4cf3e52a865f57af8f6cb5cba7a7ebd1d1e3d768a711d06510c0165`, prlctl `c39d0fd701c5c8857d2740d075745cc07bb3d2ceb9ed191265c2da6ab3179e0b`. Sending requires `prlctl … --wallet sendtoaddress`. Mining does not need the wallet online.
- Ports: P2P `44108`, wallet RPC `44207`, node RPC `44107`. `1 PRL = 1e8 grain`.
- On-chain balance (no wallet required): `GET https://pearltrack.io/api/v1/address/<address>` → `balancePrl`, `totalReceivedPrl`, `totalSentPrl`.
- Stop the miner with `tmux kill-session -t pearl` or an exact PID. Do not `pkill -f` a pattern that also appears in the remote shell command line.
- `--gpu-fan` is skipped without root NVML. Passwordless sudo is unavailable.

## Historical facts (do not treat as the live path) / 历史事实
- AlphaMiner `alpha-miner` v1.7.6-beta, SHA256 `c84396e2ff4ded14a8c83cd253761b46dd40927c5c43a39a20aac9ff8bdfbfe5`. AlphaPool did not credit this build on the RTX 5080; a later AlphaMiner 1.9.5.2 existed and was then retired.
- Wallet tarball originally documented: linux-amd64 from wallet v1.0.0, SHA256 `56ae87bdd2913ae8e030d32e820240696cce41cbda4b84bfd5a4968196565a47`.
- Retired pool API: `GET https://pearl.alphapool.tech/api/miner/<address>` → `balance_prl`, `total_paid_prl`, `workers[].online`, `shares24h`. PPLNS: unpaid balance is not spendable until that pool pays it.

## Known gotchas (already solved here) / 已解决的坑
- **WSL DNS** cannot resolve `x49.seederN.pearlresearch.ai` (returns "no such host"), so wallet SPV peer discovery fails. Fix: start `oyster` with `--usespv` + several `--addpeer=<IP>` from the base `seeder1.pearlresearch.ai` (which DOES resolve publicly; peers listen on `44108`).
  **WSL DNS** 解析不了 `x49.seederN.pearlresearch.ai`，导致钱包 SPV 找不到节点。解法：`oyster` 加 `--usespv` + 几个 `--addpeer=<IP>`（IP 取自能解析的基础域 `seeder1.pearlresearch.ai`，节点端口 `44108`）。
- `getnewaddress` returns `-4 blockchain RPC is inactive` until a chain backend (SPV peers) is active.
  链后端（SPV peers）未激活前，`getnewaddress` 报 `-4 blockchain RPC is inactive`。
- At wallet `--create`, answer **n** to "additional layer of encryption for public data" — otherwise startup needs `--walletpass`.
  建钱包时对 "additional layer of encryption for public data" 回答 **n**，否则启动要带 `--walletpass`。
- `nvidia-smi` is not on PATH in WSL; it lives at `/usr/lib/wsl/lib/nvidia-smi`. The miner uses `libcuda.so` directly (Windows driver), so no Linux CUDA install is needed.
  WSL 里 `nvidia-smi` 不在 PATH，在 `/usr/lib/wsl/lib/nvidia-smi`。矿工直接调 `libcuda.so`，无需在 Linux 装 CUDA。
- Kryptex pays the wallet by itself. A confirmed balance under 1 PRL is not paid, and Kryptex's own article says an inactive wallet's pool balance is wiped after 90 days. Cash-out starts from coins already in the `prl1p…` wallet, not from a pool "withdraw" button.
  Kryptex 会自己打款。确认余额不到 1 PRL 不会打出；Kryptex 自己的文章写，不活跃钱包的矿池余额 90 天后清除。变现从已经进 `prl1p…` 钱包的币开始，不是按矿池上的提现按钮。

## Conventions / 约定
- All docs are bilingual (English + 中文) with YAML-ish HTML-comment frontmatter at the top.
  所有文档中英双语，顶部用 HTML 注释写 frontmatter。
- Always **checksum-verify** any downloaded binary before running it; show the expected hash.
  运行任何下载的二进制前**必须校验** SHA256，并写出预期值。
- Commits: conventional messages, stage specific files (never `git add -A`), never stage secrets.
  提交：用规范化 message，逐个列举文件（禁止 `git add -A`），绝不提交敏感文件。
