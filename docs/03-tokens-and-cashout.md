<!--
title: Token disambiguation & cash-out — turning mined PRL into money
description: Why mined Pearl PRL is not the Binance PRL, the CEX route via SafeTrade (PRL/USDT, 0.1% fee, native deposit, USDT-TRC20 out — the cheapest path to Binance), the bridge→WPRL→DEX/OTC alternatives, a full fee comparison (SafeTrade vs lordofpearls vs pearl-otc vs bridge), the Ethereum gas math behind withdrawal cost, and the "how much to accumulate before cashing out" break-even.
language: [en, zh]
updated: 2026-06-06
tags: [cashout, safetrade, cex, trc20, wprl, bridge, otc, uniswap, gas, perle, disambiguation]
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

## 2. The only CEX is SafeTrade — NOT Binance/OKX/etc. / 唯一的 CEX 是 SafeTrade——不是币安/欧易
As of 2026-06-06, native Pearl PRL (pearlresearch.ai, CoinGecko-confirmed) **is listed on exactly one CEX: SafeTrade** (`safetrade.com`, PRL/USDT, ~$0.75, ~99.8% of all PRL volume). It is **still NOT on Binance/OKX/Gate/Bitget/MEXC**. Sending native PRL to a *major* exchange's "PRL" deposit address = wrong project/chain = **permanent loss**.
截至 2026-06-06，原生 Pearl PRL（官网 pearlresearch.ai，经 CoinGecko 核实）**只上了一个 CEX：SafeTrade**（PRL/USDT，~$0.75，占全网 ~99.8% 成交量）。它**仍未上币安/欧易/Gate/Bitget/MEXC**。把原生 PRL 充到**主流**交易所的 "PRL" 地址 = 错项目/错链 = **永久丢币**。

> ⚠️ SafeTrade is a **small custodial exchange** — only park PRL there during "deposit → sell → withdraw", then move the USDT out immediately. Don't store balances on it. / SafeTrade 是**小型托管交易所**——只在"充→卖→提"期间短暂存放，卖完立刻把 USDT 提走，不要久放。

## 3. Cash-out paths / 变现路径

**★ Preferred — SafeTrade CEX (cheapest, no Ethereum gas) / 首选——SafeTrade（最省，绕开以太坊 gas）:**
```text
native PRL (what you mined)
  → deposit directly to SafeTrade  (native L1 send, fee ≈ $0 — verify it gives a prl1p… address)
  → sell PRL/USDT on the order book  (0.1% trading fee)
  → withdraw USDT via TRC-20 (Tron)  (~1 USDT flat fee)
  → Binance credits TRC-20 USDT for free → done
```

**Alternative — bridge/OTC (only if SafeTrade is unusable) / 备选——过桥/OTC（仅当 SafeTrade 不可用时）:**
```text
native PRL
  → PearlBridge: lock PRL → mint WPRL (ERC-20 on Ethereum)
  → Uniswap V3/V4: sell WPRL → USDT/WETH (~1% pool fee + ETH gas + slippage)
  → or an OTC bot: sell PRL → USDC directly (lordofpearls 1.8% / pearl-otc 2%)
  → send USDT/USDC to a CEX (e.g. Binance) → fiat
```
The SafeTrade route wins on every axis: **0.1% vs 1.8–2%** fee, a **real order book** (vs OTC spread), **native deposit** (no bridge/mint), and **USDT-TRC20 out** (no Ethereum gas). The bridge/OTC paths only matter if SafeTrade requires KYC you can't pass, lacks order-book depth for your size, or you distrust holding on a small CEX.
SafeTrade 这条路全面占优：费率 **0.1% vs 1.8–2%**、**真实订单簿**（非 OTC 价差）、**原生直充**（不用过桥铸币）、**USDT 走 TRC-20**（绕开以太坊 gas）。只有当 SafeTrade 要 KYC 你过不了、深度不够你的卖量、或你不信任小所托管时，才退回过桥/OTC。

## 4. Liquidity reality / 流动性现实
- **Native PRL's primary market is SafeTrade** (PRL/USDT, ~`$75万`/24h ≈ 99.8% of all PRL volume). **WPRL** is just the bridged Ethereum representation for DEX use, traded only on **Uniswap**; WPRL 24h volume ~`$5–47万`, pool depth ~`$13–29万`. / **原生 PRL 的主市场是 SafeTrade**（PRL/USDT，~`$75万`/24h ≈ 全网 99.8% 成交）。**WPRL** 只是过桥到以太坊供 DEX 用的封装币，仅在 **Uniswap** 上交易。
- Network mints ~`$148万`/day of new PRL — **more than the entire daily trading volume** → structural sell pressure; the quoted `$0.758` is a fragile thin-book price. Selling in size = heavy slippage.
- 全网每天新挖 ~`$148万` PRL，**超过整个市场日成交量** → 结构性抛压；标价 `$0.758` 是脆弱薄盘价，放量卖滑点大。
- Bridges (PearlBridge / Pilcrow, "experimental") are prime hack targets — don't park large amounts in WPRL/on a bridge.
  桥（PearlBridge / Pilcrow，"实验性"）是黑客头号目标——别把大额长期留在 WPRL/桥上。

## 5. Cash-out routes compared / 变现通道对比

| Route / 通道 | Headline fee / 明示费率 | Settlement / 结算 | Min / 最低 | Model / 模式 | Extra / 额外成本 |
|---|---|---|---|---|---|
| **★ SafeTrade (CEX)** | **0.1%** maker/taker | **USDT** (TRC-20 / BSC) | none | Order-book CEX (custodial) + **native PRL deposit** | ~`1 USDT` flat TRC-20 withdrawal — **no Ethereum gas** |
| **lordofpearls OTC** | **1.8%** | USDC on **Ethereum** | none stated | Telegram bot `@LOPOTCBOT` (custodial trust) | ETH withdrawal gas |
| **pearl-otc.com** | **2%** (PRL leg) | USDC on **Arbitrum** | **1,000 PRL** | P2P order book + on-chain **2-of-2 multisig escrow** (non-custodial) | Arbitrum gas (cents) |
| **PearlBridge → Uniswap** | bridge fee (in-app) + **1%** DEX | WPRL on Ethereum | — | bridge + DEX | **ETH gas ×2 ($5–20+)** + slippage |

**Verdict / 结论:**
- **SafeTrade wins outright**: `0.1%` fee is ~18× cheaper than the OTCs, it's a real order book (not an OTC spread), accepts **native PRL deposit** (no bridge/mint), and pays out **USDT on TRC-20** that Binance credits free — sidestepping Ethereum gas entirely. / **SafeTrade 全面胜出**：0.1% 费率约为 OTC 的 1/18，真实订单簿、原生直充、USDT 走 TRC-20 且币安免费入账，彻底绕开以太坊 gas。
- Take-home @ ~$0.72, 0.1% fee, $1 withdrawal: 100 PRL → ~`$70.9` (SafeTrade) vs ~`$66.0` (lordofpearls); 200 PRL → ~`$142.9` vs ~`$133.0`. / 到手对比见第 7 节。
- ⚠️ SafeTrade is **custodial** — sell and withdraw promptly; never store balances. May require **KYC**; verify the deposit screen shows a `prl1p…` address and confirm the live TRC-20 USDT withdrawal fee before relying on it. / SafeTrade 是**托管**所——卖完即提、不久放；可能需 **KYC**；用前在充值页确认是 `prl1p…` 地址、并核对当时的 TRC-20 USDT 提现费。
- For the OTC fallbacks: the real cost is the **spread** (price on a thin book), not the 0.2% fee gap — always get a same-day quote. / OTC 备选的真正成本是**成交价差**，不是那 0.2%——务必拿当天报价。
- ⚠️ All four are unofficial third parties; this repo does not vouch for their safety. / 四家全是非官方第三方，本仓库不为其安全背书。

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
| **★ SafeTrade (0.1% + TRC-20)** | **~$1** (flat) | ≥ $35–70 | **~50–100 PRL** | **~1 week** |
| lordofpearls (1.8% + ETH gas) | ~$2–6 (×2 to forward) | ≥ $150–250 | ~200–330 PRL | ~1 month |
| pearl-otc (2% + Arbitrum) | ~$0.1–0.5 (negligible) | min-bound | **≥1,000 PRL** | ~4–5 months |
| bridge → Uniswap | ~$10–30 + slippage | ≥ $500–1,000 | ~700–1,300 PRL | 3–5 months |

**Trade-off / 权衡:** bigger batch = lower fixed-cost %, but longer hold = more price risk on a volatile, declining, illiquid coin. SafeTrade's tiny `~$1` fixed cost collapses the break-even to **~50–100 PRL (≈ a week's mining)**, so you can cash out **early and often** to hedge the §4 sell-pressure price risk — far better than waiting a month for an OTC or months for pearl-otc's 1,000 PRL floor. Only watch the **order-book depth**: sell 200+ PRL with a limit order, not a market dump.
**权衡：** 批量越大固定成本占比越低，但持有越久越扛价格风险。SafeTrade 的 `~$1` 固定成本把回本点压到 **~50–100 PRL（约一周产量）**,所以可以**早提勤提**对冲第 4 节的抛压价格风险——远好于等一个月走 OTC、或等几个月凑 pearl-otc 的 1000 PRL 门槛。唯一要看的是**订单簿深度**:卖 200+ 个挂限价单,别市价砸盘。

## 8. Safety checklist / 安全清单
- ❌ Never send native PRL to a **major** exchange (Binance/OKX/Gate/Bitget/MEXC) "PRL" address — they list a *different* PRL or none = loss. **Only SafeTrade** lists the native pearlresearch.ai PRL (verify its deposit address is `prl1p…` before sending). / 绝不把原生 PRL 充**主流**交易所（币安/欧易/Gate/Bitget/MEXC）的 "PRL" 地址——它们上的是*别的* PRL 或根本没有 = 丢币。**只有 SafeTrade** 上的是 pearlresearch.ai 原生 PRL（充值前确认其地址是 `prl1p…`）。
- ❌ Never enter your seed/mnemonic into any website, bot, extension, or chat. Bridges/DEX only need wallet *connect + sign*. / 绝不把助记词输入任何网站/机器人/插件/聊天；桥和 DEX 只需钱包"连接+签名"。
- ✅ Verify the bridge URL and WPRL contract address against Pearl's official channels (ticker collision + phishing are rampant). / 逐字核对桥地址与 WPRL 合约（同名币+钓鱼泛滥）。
- ✅ Test the whole path with a tiny amount first; recompute real take-home from the actual USDC received, not the quoted price. / 先用小额跑通整条链路；用实际收到的 USDC 复算真实到手，别用标价。
- ✅ To manage native PRL with a GUI, use the **official Pearl desktop wallet** (`Pearl-Wallet-Setup-1.0.0.exe`), not a generic multi-chain wallet (which can't derive `prl1p…` addresses). / 想用图形界面管原生 PRL，用**官方 Pearl 桌面钱包**，别用通用多链钱包（它推导不出 `prl1p…` 地址）。

---

References / 参考: safetrade.com/exchange/PRL-USDT · coingecko.com/en/coins/pearl-2 · hashrate.no/coins/PRL · coingecko.com/en/coins/wrapped-pearl · pearlbridge.xyz · otc.lordofpearls.xyz · pearl-otc.com · explorer.pearlresearch.ai
