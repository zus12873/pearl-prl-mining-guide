<!--
title: Token disambiguation & cash-out — turning mined PRL into money
description: Why mined Pearl PRL is not the Binance PRL, why it has no CEX, the bridge→WPRL→DEX/OTC cash-out path, an OTC fee comparison (lordofpearls vs pearl-otc vs bridge), the Ethereum gas math behind withdrawal cost, and the "how much to accumulate before cashing out" break-even.
language: [en, zh]
updated: 2026-05-31
tags: [cashout, wprl, bridge, otc, uniswap, gas, perle, disambiguation]
-->

# 03 · Tokens & cash-out / 代币辨析与变现

> ⚠️ This page contains the single most expensive mistake to avoid: **do not send mined PRL to an exchange.** / 本页含最贵的一个错误：**绝不要把挖到的 PRL 充到交易所。**

## 1. There are several "PRL" — don't confuse them / "PRL" 有好几个，别混

| Token / 代币 | What / 是什么 | Where / 在哪 |
|---|---|---|
| **Pearl Research (PRL)** ← *you mine this* / *你挖的是这个* | Native coin of Pearl L1 (PoUW); addresses `prl1p…` | No CEX. Only via WPRL on a DEX / 无 CEX，只能桥成 WPRL 上 DEX |
| **Perle (PRL)** | A different project on Solana | Was on Binance, **delisted 2026-04-28** / 曾在币安，**已下架** |
| Scam clones / 山寨 | Same-ticker tokens on DEXes (e.g. Base) | Avoid / 避开 |

**Rule / 铁律:** ticker alone is meaningless. Verify project + chain + contract address every time. A "price page" on CoinGecko/Coinbase ≠ a CEX actually trading or accepting deposits of your coin.
**铁律：** 只看 ticker 没意义，每次都要核对项目+链+合约。CoinGecko/Coinbase 上有"价格页" ≠ CEX 真在交易或能充提你的币。

## 2. Native PRL has no CEX / 原生 PRL 没有 CEX
As of 2026-05-31, native Pearl PRL is **not listed on any centralized exchange** (Binance/OKX/Gate/Bitget/MEXC). Sending it to any exchange "PRL" deposit address = wrong project/chain = **permanent loss**.
截至 2026-05-31，原生 Pearl PRL **未上任何中心化交易所**。充到任何交易所的 "PRL" 地址 = 错项目/错链 = **永久丢币**。

## 3. The only cash-out path / 唯一变现路径
```text
native PRL (Pearl L1, what you mined)
  → PearlBridge: lock PRL → mint WPRL (ERC-20 on Ethereum)
  → Uniswap V3/V4: sell WPRL → USDT/WETH (DEX, ~1% pool fee + ETH gas + slippage)
  → optionally send USDT/ETH to a CEX (e.g. Binance) → fiat
```
Binance never touches native PRL — it only enters **after** you've converted to USDT/ETH. An OTC marketplace can shortcut the middle (sell PRL → USDC directly).
币安全程不碰原生 PRL，只在你换成 USDT/ETH **之后**才登场。OTC 撮合可省去中间段（直接 PRL → USDC）。

## 4. Liquidity reality / 流动性现实
- WPRL trades only on **Uniswap (DEX)**; 24h volume ~`$24–47万`, pool depth ~`$13–29万`.
- Network mints ~`$148万`/day of new PRL — **more than the entire daily trading volume** → structural sell pressure; the quoted `$0.758` is a fragile thin-book price. Selling in size = heavy slippage.
- 全网每天新挖 ~`$148万` PRL，**超过整个市场日成交量** → 结构性抛压；标价 `$0.758` 是脆弱薄盘价，放量卖滑点大。
- Bridges (PearlBridge / Pilcrow, "experimental") are prime hack targets — don't park large amounts in WPRL/on a bridge.
  桥（PearlBridge / Pilcrow，"实验性"）是黑客头号目标——别把大额长期留在 WPRL/桥上。

## 5. Cash-out routes compared / 变现通道对比

| Route / 通道 | Headline fee / 明示费率 | Settlement / 结算 | Min / 最低 | Model / 模式 | Extra / 额外成本 |
|---|---|---|---|---|---|
| **lordofpearls OTC** | **1.8%** | USDC on **Ethereum** | none stated | Telegram bot `@LOPOTCBOT` (custodial trust) | ETH withdrawal gas |
| **pearl-otc.com** | **2%** (PRL leg) | USDC on **Arbitrum** | **1,000 PRL** | P2P order book + on-chain **2-of-2 multisig escrow** (non-custodial) | Arbitrum gas (cents) |
| **PearlBridge → Uniswap** | bridge fee (in-app) + **1%** DEX | WPRL on Ethereum | — | bridge + DEX | **ETH gas ×2 ($5–20+)** + slippage |

**Verdict / 结论:**
- Headline fee: `lordofpearls 1.8%` < `pearl-otc 2%` < `bridge+DEX`. / 明示费率上 lordofpearls 最低。
- Total cost: bridge→DEX is worst for small/mid amounts (Ethereum gas + slippage). / 总成本上 bridge→DEX 对中小额最贵。
- pearl-otc costs 0.2% more but settles on **Arbitrum (cheap gas)** with a **trustless multisig escrow** — structurally safer, but needs ≥1,000 PRL. / pearl-otc 贵 0.2% 但走 Arbitrum + 链上多签托管，更安全，但门槛 1000 PRL。
- ⚠️ The real cost is the **spread** (price you actually get on a thin book), not the 0.2% fee gap — always get a same-day quote. / 真正成本是**成交价差**，不是那 0.2%——务必拿当天报价。
- ⚠️ All three are unofficial third parties; this repo does not vouch for their safety. / 三家全是非官方第三方，本仓库不为其安全背书。

## 6. Ethereum gas — how withdrawal cost is computed / 提现 gas 怎么算
```
fee(ETH) = gasUsed × gasPrice(gwei) ÷ 1e9
fee(USD) = fee(ETH) × ETH_price
```
- **gasUsed** (fixed per op): ETH transfer 21,000; **USDC (ERC-20) transfer ~60,000**; Uniswap swap ~120,000–200,000.
- **gasPrice** (gwei): network base fee + tip; swings 5→100+ gwei with congestion — the biggest variable.
- Example: `60,000 × 15 gwei ÷ 1e9 = 0.0009 ETH × $3,000 ≈ $2.7` (≈ `$1.4` quiet / `$5.4` busy).

Why this route is pricey: lordofpearls settles USDC on **Ethereum mainnet** (the expensive chain). pearl-otc uses **Arbitrum (L2)** where the same transfer is ~`$0.05–0.5`. And to fully land on Binance you may pay gas **twice** (receive USDC, then forward it) → ~`$4–12` fixed on mainnet.
为什么这条路贵：lordofpearls 在**以太坊主网**结算（最贵的链）；pearl-otc 走 **Arbitrum(L2)**，同样转账只 ~`$0.05–0.5`。完整到币安主网上可能付**两次** gas（收 USDC + 再转出）→ 主网约 ~`$4–12` 固定成本。

Check live gas before acting: `etherscan.io/gastracker`. / 操作前查实时 gas。

## 7. How much to accumulate before it's worth it / 攒够多少再提才划算
Fixed costs (gas) don't scale with amount, so cash out in batches where **fixed cost ≤ ~2–3% of the batch**. (PRL ≈ $0.75; gas varies.)
固定成本（gas）不随金额变，所以批量提现，让**固定成本 ≤ 这一笔的 ~2–3%**。（PRL≈$0.75，gas 浮动。）

| Route / 通道 | Fixed cost / 固定成本 | Worthwhile batch / 划算批量 | ≈ PRL | @7–8 PRL/day |
|---|---|---|---|---|
| lordofpearls (1.8% + ETH gas) | ~$2–6 (×2 to forward) | ≥ $150–250 | ~200–330 PRL | ~1 month |
| pearl-otc (2% + Arbitrum) | ~$0.1–0.5 (negligible) | min-bound | **≥1,000 PRL** | ~4–5 months |
| bridge → Uniswap | ~$10–30 + slippage | ≥ $500–1,000 | ~700–1,300 PRL | 3–5 months |

**Trade-off / 权衡:** bigger batch = lower fee %, but longer hold = more price risk on a volatile, declining, illiquid coin. Given the §4 sell pressure, **cashing out sooner in ~200–300 PRL batches** (frequency hedges price risk) is often wiser than waiting months to hit pearl-otc's 1,000 PRL floor.
**权衡：** 批量越大费率%越低，但持有越久越扛价格风险。鉴于第 4 节的抛压，**早点按 ~200–300 PRL 分批提**（用频率对冲价格风险）往往比死等 pearl-otc 的 1000 PRL 门槛更稳。

## 8. Safety checklist / 安全清单
- ❌ Never send native PRL to any exchange "PRL" address. / 绝不把原生 PRL 充任何交易所 "PRL" 地址。
- ❌ Never enter your seed/mnemonic into any website, bot, extension, or chat. Bridges/DEX only need wallet *connect + sign*. / 绝不把助记词输入任何网站/机器人/插件/聊天；桥和 DEX 只需钱包"连接+签名"。
- ✅ Verify the bridge URL and WPRL contract address against Pearl's official channels (ticker collision + phishing are rampant). / 逐字核对桥地址与 WPRL 合约（同名币+钓鱼泛滥）。
- ✅ Test the whole path with a tiny amount first; recompute real take-home from the actual USDC received, not the quoted price. / 先用小额跑通整条链路；用实际收到的 USDC 复算真实到手，别用标价。
- ✅ To manage native PRL with a GUI, use the **official Pearl desktop wallet** (`Pearl-Wallet-Setup-1.0.0.exe`), not a generic multi-chain wallet (which can't derive `prl1p…` addresses). / 想用图形界面管原生 PRL，用**官方 Pearl 桌面钱包**，别用通用多链钱包（它推导不出 `prl1p…` 地址）。

---

References / 参考: hashrate.no/coins/PRL · coingecko.com/en/coins/wrapped-pearl · pearlbridge.xyz · otc.lordofpearls.xyz · pearl-otc.com · explorer.pearlresearch.ai
