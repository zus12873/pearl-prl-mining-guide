<!--
title: Pearl (PRL) GPU Mining Guide — WSL + Kryptex
description: Field-tested, AI-friendly bilingual guide to mining Pearl L1 (PRL, Proof-of-Useful-Work) on a Windows/WSL2 NVIDIA GPU, with wallet setup, PeakMiner on Kryptex, monitoring, and the cash-out path. The May 2026 AlphaPool run is kept as history.
language: [en, zh]
updated: 2026-09-27
tags: [pearl, prl, pouw, gpu-mining, wsl, rtx-5080, kryptex, peakminer, crypto]
status: field-tested on RTX 5080 / WSL2, PeakMiner 2.16.5 + Kryptex, 2026-09-27
-->

# Pearl (PRL) GPU Mining Guide — WSL Edition
# Pearl (PRL) GPU 挖矿实操指南 — WSL 版

> ⚠️ **Not investment advice.** The live rig notes are a 2026-09-27 snapshot. The ROI tables in [docs/01](docs/01-feasibility-and-roi.md) are still the 2026-05-31 study. Real take-home depends on coin price, pool payout, sellable liquidity, power draw and network state.
> ⚠️ **不构成投资建议。** 当前矿机记录是 2026-09-27 快照。[docs/01](docs/01-feasibility-and-roi.md) 里的回报率表仍是 2026-05-31 的研究。真实到手取决于币价、矿池到账、可成交流动性、功耗与网络状态。

This repo is a **reproducible, field-tested** record of mining **Pearl L1 (native PRL)** on a single NVIDIA GPU inside **Windows + WSL2**, end to end: wallet → miner → monitoring → cash-out, plus an honest feasibility/ROI analysis.

本仓库是在 **Windows + WSL2** 下用单张 NVIDIA 显卡挖 **Pearl L1 原生币 PRL** 的**可复现、已实测**全流程记录：建钱包 → 跑矿工 → 监控 → 变现，并附一份诚实的可行性/回报率分析。

---

## 🚨 Read first — the #1 way to lose your coins / 最重要的安全前提

**The PRL you mine (Pearl Research, an L1) is NOT the PRL on Binance (Perle, a different Solana project, delisted 2026-04-28).**
**你挖的 PRL（Pearl Research，一条 L1）≠ 币安上的 PRL（Perle，另一个 Solana 项目，已于 2026-04-28 下架）。**

- The completed cash-out is **SafeTrade** (PRL/USDT). It is **not** the PRL that was on Binance. Sending native PRL to a deposit address that does not start with `prl1p…` = **permanent loss**.
- 已经跑通的变现是 **SafeTrade**（PRL/USDT）。它**不是**币安上曾有过的那个 PRL。把原生 PRL 充到不以 `prl1p…` 开头的地址 = **永久丢币**。
- Two hops: Kryptex pays your `prl1p…` wallet automatically (minimum 1 PRL). Then `wallet → SafeTrade → sell PRL/USDT (0.1%) → USDT via TRC-20`. See [docs/03](docs/03-tokens-and-cashout.md).
- 两段：Kryptex 达到 1 PRL 后自动打到你的 `prl1p…` 钱包。然后 `钱包 → SafeTrade → 卖 PRL/USDT（0.1%）→ USDT 走 TRC-20`。详见 [docs/03](docs/03-tokens-and-cashout.md)。
- **Never** paste your seed phrase / mnemonic into any website, bot, or chat. / **绝不**把助记词输入任何网站、机器人或聊天。

---

## 📚 Repository map / 仓库索引

| File | What it covers / 内容 |
|---|---|
| [docs/01-feasibility-and-roi.md](docs/01-feasibility-and-roi.md) | Feasibility & ROI: rented 4090 vs owned 3090, electricity, break-even / 可行性与回报率：租 4090 vs 自有 3090、电费、盈亏线 |
| [docs/02-deployment-runbook.md](docs/02-deployment-runbook.md) | **The verified step-by-step**: WSL GPU, wallet, miner, monitoring + gotchas / **已验证的逐步实操**：WSL GPU、钱包、矿工、监控 + 踩坑 |
| [docs/03-tokens-and-cashout.md](docs/03-tokens-and-cashout.md) | Token disambiguation, cash-out routes, OTC fees, Ethereum gas math / 代币辨析、变现路径、OTC 费率、以太坊 gas 算法 |
| [AGENTS.md](AGENTS.md) | Context & conventions for AI coding agents / 给 AI 编程代理的上下文与约定 |

---

## ⚡ Quick start / 快速开始 (current rig: RTX 5080)

Prerequisites: Windows 11 + WSL2 (Ubuntu) + up-to-date **NVIDIA Windows driver** (do NOT install a Linux GPU driver inside WSL).
前置：Windows 11 + WSL2(Ubuntu) + 最新 **NVIDIA Windows 驱动**（WSL 内不要装 Linux 显卡驱动）。

1. Verify GPU in WSL / 在 WSL 里确认 GPU：`/usr/lib/wsl/lib/nvidia-smi`
2. Create wallet, get a `prl1p…` payout address (you keep the mnemonic offline) — see [docs/02 §B](docs/02-deployment-runbook.md).
   建钱包、拿一个 `prl1p…` 收款地址（助记词你离线保存）—— 见 [docs/02 §B](docs/02-deployment-runbook.md)。
3. Run checksum-verified **PeakMiner** in tmux session `pearl` against Kryptex — see [docs/02 §C](docs/02-deployment-runbook.md).
   校验后在 tmux 会话 `pearl` 里跑 **PeakMiner**，连 Kryptex —— 见 [docs/02 §C](docs/02-deployment-runbook.md)。
4. Watch the local API `http://127.0.0.1:4068/summary` and the Kryptex balance API — see [docs/02 §D](docs/02-deployment-runbook.md). Coins move to the wallet by themselves once the confirmed balance reaches 1 PRL.
   看本机 API `http://127.0.0.1:4068/summary` 和 Kryptex 余额接口 —— 见 [docs/02 §D](docs/02-deployment-runbook.md)。确认余额到 1 PRL 后，币会自己进钱包。

> Throughout the docs, replace `prl1p<your-address>`, `<rpc-user>`, `<rpc-pass>` with your own values.
> 文档中 `prl1p<your-address>`、`<rpc-user>`、`<rpc-pass>` 都请替换成你自己的值。

---

## 📌 Key parameters (live rig, verified 2026-09-27) / 关键参数（当前矿机，2026-09-27 实测）

| Item / 项 | Value / 值 |
|---|---|
| Chain / 链 | Pearl L1, Proof-of-Useful-Work (matrix-mul); addresses `prl1p…` (bech32m) |
| Miner / 矿工 | PeakMiner 2.16.5, tmux session `pearl`, worker name of your choice. On-disk SHA256 of `peakminer-2.16.5-linux-x86_64`: `5dc4b927fb91442a66a2e02c1636f629d4042a03a5c1523cd841d6ece66e0beb` |
| Pool / 矿池 | Kryptex PPS+ **2%** (SOLO 1%). Stratum tried in order: `prl-hk.kryptex.network:7048`, `prl-sg.kryptex.network:7048`, `prl.kryptex.network:7048`. Username `prl1p<your-address>/worker` |
| Pool payout / 矿池打款 | Automatic, about hourly, once **confirmed** balance ≥ threshold. Minimum and default threshold **1 PRL**. Matures after **100 blocks** (~194 s target). Kryptex pays the tx fee. No manual withdraw |
| Miner dev fee / 矿工抽成 | PeakMiner **2%**, separate from the pool fee |
| Wallet CLI / 钱包 | `oyster` / `prlctl` 1.1.0 under `~/pearl-wallet`. Mining does not need it online. Sending coins does |
| Ports / 端口 | P2P `44108`, wallet RPC `44207`, node RPC `44107`. PeakMiner local API `127.0.0.1:4068` |
| Units / 单位 | `1 PRL = 100,000,000 grain` |
| RTX 5080 sample / 样例 | 2026-09-27 restart: ~218 TH/s, ~360 W, ~75°C, fan on the driver curve. One earlier session median was ~230 TH/s. Not a yield promise |
| Retired / 已停用 | AlphaMiner + AlphaPool (`us2.alphapool.tech:5566`, PPLNS). RTX 3090 notes in [docs/01](docs/01-feasibility-and-roi.md) stay historical |

---

## ⚖️ License / Disclaimer

Documentation only. No warranty. Cryptocurrency mining and trading carry financial and security risk; you are solely responsible for your funds, keys, hardware, electricity costs, and legal/tax compliance in your jurisdiction.
仅为文档。不作任何担保。加密货币挖矿与交易有资金与安全风险；你对自己的资金、密钥、硬件、电费以及所在地的法律/税务合规负全部责任。
