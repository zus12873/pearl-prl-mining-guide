<!--
title: Pearl (PRL) GPU Mining Guide — WSL + AlphaPool
description: Field-tested, AI-friendly bilingual guide to mining Pearl L1 (PRL, Proof-of-Useful-Work) on a Windows/WSL2 NVIDIA GPU, with wallet setup, monitoring, feasibility/ROI analysis, and the real cash-out path.
language: [en, zh]
updated: 2026-05-31
tags: [pearl, prl, pouw, gpu-mining, wsl, rtx-3090, alphapool, crypto]
status: field-tested on RTX 3090 / WSL2, 2026-05-31
-->

# Pearl (PRL) GPU Mining Guide — WSL Edition
# Pearl (PRL) GPU 挖矿实操指南 — WSL 版

> ⚠️ **Not investment advice.** Numbers reflect a 2026-05-31 snapshot and change fast. Real take-home depends on coin price, pool payout, sellable liquidity, power draw and network state.
> ⚠️ **不构成投资建议。** 数据为 2026-05-31 快照、变化极快。真实到手取决于币价、矿池到账、可成交流动性、功耗与网络状态。

This repo is a **reproducible, field-tested** record of mining **Pearl L1 (native PRL)** on a single NVIDIA GPU inside **Windows + WSL2**, end to end: wallet → miner → monitoring → cash-out, plus an honest feasibility/ROI analysis.

本仓库是在 **Windows + WSL2** 下用单张 NVIDIA 显卡挖 **Pearl L1 原生币 PRL** 的**可复现、已实测**全流程记录：建钱包 → 跑矿工 → 监控 → 变现，并附一份诚实的可行性/回报率分析。

---

## 🚨 Read first — the #1 way to lose your coins / 最重要的安全前提

**The PRL you mine (Pearl Research, an L1) is NOT the PRL on Binance (Perle, a different Solana project, delisted 2026-04-28).**
**你挖的 PRL（Pearl Research，一条 L1）≠ 币安上的 PRL（Perle，另一个 Solana 项目，已于 2026-04-28 下架）。**

- The only CEX listing native PRL is **SafeTrade** (PRL/USDT). It is **NOT** on Binance/OKX/Gate/Bitget/MEXC — sending native PRL to a *major* exchange's "PRL" address = **permanent loss**.
- 上原生 PRL 的 CEX **只有 SafeTrade**（PRL/USDT）。它**没**上币安/欧易/Gate/Bitget/MEXC——把原生 PRL 充到*主流*交易所的 "PRL" 地址 = **永久丢币**。
- Cheapest cash-out: `native PRL → SafeTrade → sell PRL/USDT (0.1%) → USDT via TRC-20 → Binance`. Fallback: `→ (bridge) WPRL → DEX/OTC`. See [docs/03](docs/03-tokens-and-cashout.md).
- 最省变现：`原生 PRL → SafeTrade → 卖 PRL/USDT(0.1%) → USDT 走 TRC-20 → 币安`。备选：`→（过桥）WPRL → DEX/OTC`。详见 [docs/03](docs/03-tokens-and-cashout.md)。
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

## ⚡ Quick start / 快速开始 (RTX 3090, owned hardware)

Prerequisites: Windows 11 + WSL2 (Ubuntu) + up-to-date **NVIDIA Windows driver** (do NOT install a Linux GPU driver inside WSL).
前置：Windows 11 + WSL2(Ubuntu) + 最新 **NVIDIA Windows 驱动**（WSL 内不要装 Linux 显卡驱动）。

1. Verify GPU in WSL / 在 WSL 里确认 GPU：`/usr/lib/wsl/lib/nvidia-smi`
2. Create wallet, get a `prl1p…` payout address (you keep the mnemonic offline) — see [docs/02 §B](docs/02-deployment-runbook.md).
   建钱包、拿一个 `prl1p…` 收款地址（助记词你离线保存）—— 见 [docs/02 §B](docs/02-deployment-runbook.md)。
3. Download + **checksum-verify** the miner, run it in `tmux` — see [docs/02 §C](docs/02-deployment-runbook.md).
   下载 + **校验** 矿工，在 `tmux` 里运行 —— 见 [docs/02 §C](docs/02-deployment-runbook.md)。
4. Monitor: pool dashboard / API by address — see [docs/02 §D](docs/02-deployment-runbook.md).
   监控：用地址查矿池面板/API —— 见 [docs/02 §D](docs/02-deployment-runbook.md)。

> Throughout the docs, replace `prl1p<your-address>`, `<rpc-user>`, `<rpc-pass>` with your own values.
> 文档中 `prl1p<your-address>`、`<rpc-user>`、`<rpc-pass>` 都请替换成你自己的值。

---

## 📌 Key parameters (verified 2026-05-31) / 关键参数（2026-05-31 实测）

| Item / 项 | Value / 值 |
|---|---|
| Chain / 链 | Pearl L1, Proof-of-Useful-Work (matrix-mul); addresses `prl1p…` (bech32m) |
| Miner / 矿工 | `alpha-miner` v1.7.6-beta (GitHub `AlphaMine-Tech/alpha-miner`) |
| Wallet CLI / 钱包 | `oyster`/`prlctl`/`pearld` (GitHub `pearl-research-labs/pearl`, wallet v1.0.0 / Go v1.0.2) |
| Pool / 矿池 | AlphaPool, stratum `us2.alphapool.tech:5566` (also `sg1`/`eu1`), PPLNS, fee 5% + 1% dev |
| Ports / 端口 | P2P `44108`, wallet RPC `44207`, node RPC `44107` |
| Units / 单位 | `1 PRL = 100,000,000 grain` |
| RTX 3090 | ~100–117 TH/s observed; ~290–390 W; expected ~7–8 PRL/day at network state on 2026-05-31 |

---

## ⚖️ License / Disclaimer

Documentation only. No warranty. Cryptocurrency mining and trading carry financial and security risk; you are solely responsible for your funds, keys, hardware, electricity costs, and legal/tax compliance in your jurisdiction.
仅为文档。不作任何担保。加密货币挖矿与交易有资金与安全风险；你对自己的资金、密钥、硬件、电费以及所在地的法律/税务合规负全部责任。
