<!--
title: AGENTS.md — context & conventions for AI agents
description: Machine-oriented brief so an AI coding agent can safely continue this Pearl (PRL) mining project without re-discovering the gotchas.
language: [en, zh]
updated: 2026-05-31
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

## Verified facts (snapshot 2026-05-31) / 已验证事实
- Miner: `alpha-miner` v1.7.6-beta, SHA256 `c84396e2ff4ded14a8c83cd253761b46dd40927c5c43a39a20aac9ff8bdfbfe5`.
- Wallet CLI (linux-amd64, in wallet v1.0.0 release): SHA256 `56ae87bdd2913ae8e030d32e820240696cce41cbda4b84bfd5a4968196565a47`.
- Ports: P2P `44108`, wallet RPC `44207`, node RPC `44107`. `1 PRL = 1e8 grain`.
- Pool API (Cloudflare-cached, may lag): `GET https://pearl.alphapool.tech/api/miner/<address>` → fields `balance_prl`, `total_paid_prl`, `workers[].online`, `shares24h`.

## Known gotchas (already solved here) / 已解决的坑
- **WSL DNS** cannot resolve `x49.seederN.pearlresearch.ai` (returns "no such host"), so wallet SPV peer discovery fails. Fix: start `oyster` with `--usespv` + several `--addpeer=<IP>` from the base `seeder1.pearlresearch.ai` (which DOES resolve publicly; peers listen on `44108`).
  **WSL DNS** 解析不了 `x49.seederN.pearlresearch.ai`，导致钱包 SPV 找不到节点。解法：`oyster` 加 `--usespv` + 几个 `--addpeer=<IP>`（IP 取自能解析的基础域 `seeder1.pearlresearch.ai`，节点端口 `44108`）。
- `getnewaddress` returns `-4 blockchain RPC is inactive` until a chain backend (SPV peers) is active.
  链后端（SPV peers）未激活前，`getnewaddress` 报 `-4 blockchain RPC is inactive`。
- At wallet `--create`, answer **n** to "additional layer of encryption for public data" — otherwise startup needs `--walletpass`.
  建钱包时对 "additional layer of encryption for public data" 回答 **n**，否则启动要带 `--walletpass`。
- `nvidia-smi` is not on PATH in WSL; it lives at `/usr/lib/wsl/lib/nvidia-smi`. The miner uses `libcuda.so` directly (Windows driver), so no Linux CUDA install is needed.
  WSL 里 `nvidia-smi` 不在 PATH，在 `/usr/lib/wsl/lib/nvidia-smi`。矿工直接调 `libcuda.so`，无需在 Linux 装 CUDA。

## Conventions / 约定
- All docs are bilingual (English + 中文) with YAML-ish HTML-comment frontmatter at the top.
  所有文档中英双语，顶部用 HTML 注释写 frontmatter。
- Always **checksum-verify** any downloaded binary before running it; show the expected hash.
  运行任何下载的二进制前**必须校验** SHA256，并写出预期值。
- Commits: conventional messages, stage specific files (never `git add -A`), never stage secrets.
  提交：用规范化 message，逐个列举文件（禁止 `git add -A`），绝不提交敏感文件。
