<!--
title: Token disambiguation & cash-out — turning mined PRL into money
description: Why mined Pearl PRL is not the Binance PRL, the CEX route via SafeTrade (PRL/USDT, 0.1% fee, native deposit, USDT-TRC20 out — the cheapest path to Binance), the bridge→WPRL→DEX/OTC alternatives, a full fee comparison (SafeTrade vs lordofpearls vs pearl-otc vs bridge), the Ethereum gas math behind withdrawal cost, and the "how much to accumulate before cashing out" break-even.
language: [en, zh]
updated: 2026-09-27
tags: [cashout, safetrade, kryptex, cex, trc20, wprl, bridge, otc, uniswap, gas, perle, disambiguation]
-->

# 03 · Tokens & cash-out / 代币辨析与变现

> ⚠️ The expensive mistake: **do not send native PRL to a deposit address that is not `prl1p…`.** SafeTrade's Pearl deposit is a `prl1p…` address. Binance / OKX / Gate / Bitget / MEXC "PRL" deposits are not this coin.
> ⚠️ 最贵的错误：**不要把原生 PRL 充到不是 `prl1p…` 的地址。** SafeTrade 的 Pearl 充值地址是 `prl1p…`。币安 / 欧易 / Gate / Bitget / MEXC 的 “PRL” 充值不是这枚币。

## 1. There are several "PRL" — don't confuse them / "PRL" 有好几个，别混

| Token / 代币 | What / 是什么 | Where / 在哪 |
|---|---|---|
| **Pearl Research (PRL)** ← *you mine this* / *你挖的是这个* | Native coin of Pearl L1 (PoUW); addresses `prl1p…` | SafeTrade PRL/USDT is the path this rig has completed. A deposit address that is not `prl1p…` is the wrong asset |
| **Perle (PRL)** | A different project on Solana | Was on Binance, **delisted 2026-04-28** / 曾在币安，**已下架** |
| Scam clones / 山寨 | Same-ticker tokens on DEXes (e.g. Base) | Avoid / 避开 |

**Rule / 铁律:** ticker alone is meaningless. Verify project + chain + contract address every time. A "price page" on CoinGecko/Coinbase ≠ a CEX actually trading or accepting deposits of your coin.
**铁律：** 只看 ticker 没意义，每次都要核对项目+链+合约。CoinGecko/Coinbase 上有"价格页" ≠ CEX 真在交易或能充提你的币。

## 2. SafeTrade is the completed path — not Binance/OKX/etc. / 跑通过的是 SafeTrade——不是币安/欧易
On 2026-06-09 this wallet sent native PRL to SafeTrade (`safetrade.com`, pair PRL/USDT), sold it, and withdrew USDT on TRC-20. That is the path to repeat. Binance, OKX, Gate, Bitget and MEXC do not take this coin's `prl1p…` deposits. Sending to their "PRL" address is a different project or no project = **permanent loss**.
2026-06-09 这个钱包把原生 PRL 充进了 SafeTrade（交易对 PRL/USDT），卖掉后用 TRC-20 提出 USDT。要重复的是这条。币安、欧易、Gate、Bitget、MEXC 不收这枚币的 `prl1p…` 充值。充到它们的 “PRL” 地址 = 另一个项目或根本没有 = **永久丢币**。

Aggregator pages in September 2026 also print BigONE `PRL/USDT` and CoinEx `PEARL/USDT`. This repo has not completed a deposit on either. CoinEx's ticker is a different spelling. Do not use them unless that exchange's deposit screen shows a `prl1p…` address and you have sent a tiny test that credited.
2026 年 9 月的行情站还列出 BigONE 的 `PRL/USDT` 和 CoinEx 的 `PEARL/USDT`。这两家本仓库都没有跑通过。CoinEx 的代号拼法就不一样。除非那家的充值页给出 `prl1p…` 地址，并且一笔小额测试已经入账，否则不要用。

> ⚠️ SafeTrade is a **small custodial exchange** — only park PRL there during "deposit → sell → withdraw", then move the USDT out immediately. Don't store balances on it. / SafeTrade 是**小型托管交易所**——只在"充→卖→提"期间短暂存放，卖完立刻把 USDT 提走，不要久放。

## 2.1 What you actually do / 实际要做的操作

Two different balances. Only the second one can be sold.
有两笔不同的余额。只有第二笔能拿去卖。

| Where / 在哪 | What it is / 是什么 | What you do / 怎么处理 |
|---|---|---|
| Kryptex `confirmed` / `unconfirmed` | Pool accounting. Not in your wallet until a payout txid exists | Nothing, if `confirmed` ≥ 1 PRL: the pool sends it within about an hour. If it is under 1 PRL, keep mining. See [02 §E](02-deployment-runbook.md) |
| `balancePrl` on pearltrack, or `getbalance` in the wallet | Coins you already hold | This is the cash-out |

```bash
curl -s "https://pearltrack.io/api/v1/address/prl1p<your-address>"
curl -s "https://pool.kryptex.com/prl/api/v1/miner/balance/prl1p<your-address>"
```

### Send from the wallet / 从钱包转出

The mining wallet can stay offline while hashing. It has to be online to sign a send. Use the same `oyster` start as [02 §B3](02-deployment-runbook.md), wait until it is synced, then in a second terminal:
挖矿时钱包可以关着。签名转账时必须开着。按 [02 §B3](02-deployment-runbook.md) 启动 `oyster`，等同步完，另开一个终端：

```bash
cd ~/pearl-wallet
./prlctl -u <rpc-user> -P '<rpc-pass>' -s localhost:44207 --notls --wallet getbalance
```

On SafeTrade, open the Pearl (pearlresearch.ai) deposit screen. Copy the address only after you see it starts with `prl1p`. Then send a **test** (2 PRL is the amount that was used here on 2026-06-09):
在 SafeTrade 打开 Pearl（pearlresearch.ai）充值页。看到地址以 `prl1p` 开头再复制。先转一笔**测试**（2026-06-09 用的是 2 PRL）：

```bash
./prlctl -u <rpc-user> -P '<rpc-pass>' -s localhost:44207 --notls --wallet \
  sendtoaddress "prl1p<exchange-deposit>" 2
```

`--wallet` is required; without it `sendtoaddress` is not a wallet command. If the wallet answers that it is locked, unlock it locally for a few minutes and send immediately:
必须带 `--wallet`，否则 `sendtoaddress` 不是钱包命令。如果钱包回复已锁定，就在本机解锁几分钟并马上转出：

```bash
./prlctl -u <rpc-user> -P '<rpc-pass>' -s localhost:44207 --notls --wallet \
  walletpassphrase '<the-passphrase-you-set-at-create>' 300
```

Type that passphrase yourself on the rig. Do not put it in this repo, in a script that gets committed, or in chat. After the test credits, repeat `sendtoaddress` for the amount you want to sell and leave the dust. The June sends cost `0.0000155` and `0.0001945` PRL in chain fees.
口令在矿机上自己输入。不要写进本仓库、不要写进会被提交的脚本、不要贴进聊天。测试入账后，再 `sendtoaddress` 你准备卖的数量，零头留着。6 月那两笔链上费是 `0.0000155` 和 `0.0001945` PRL。

SafeTrade credited after **20 confirmations** (~20–40 min that day). Read the deposit page again; do not assume the number is still 20.
SafeTrade 当时是 **20 个确认**后入账（那天约 20–40 分钟）。再看一眼充值页，不要默认还是 20。

The official Pearl desktop wallet can do the same send. The CLI on this rig reports `oyster` / `prlctl` 1.1.0. A 2.0.0 archive is downloaded there and was not the binary those version strings came from.
官方 Pearl 桌面钱包也能转。这台机器上的 CLI 是 `oyster` / `prlctl` 1.1.0。旁边有一份下好的 2.0.0 压缩包，但现在跑的不是它。

### Sell and leave / 卖掉就走

1. Sell **PRL/USDT with limit orders**, in more than one clip if the book is thin. The 2026-06-09 market sell filled about **0.52 USDT** against a higher quoted price. The fee was 0.1%. The loss was the fill, not the fee.
   用**限价单**卖 PRL/USDT。盘口薄就拆开。2026-06-09 的市价单成交约 **0.52 USDT**，低于当时标价。手续费 0.1%。少拿的是成交价，不是手续费。
2. Withdraw **USDT on TRC-20** to an exchange that actually holds your USDT (Binance credited TRC-20 USDT with no deposit fee that day). The measured withdrawal fee was **~0.08 USDT**. Confirm the live fee and the network name `TRC20` / `Tron` before you confirm.
   把 **USDT 走 TRC-20** 提到真正收你 USDT 的交易所（那天币安对 TRC-20 USDT 免充值费）。实测提现费 **约 0.08 USDT**。确认前核对当时的费用和网络名 `TRC20` / `Tron`。
3. SafeTrade is a small custodial exchange. Do not leave a balance there after the USDT is out.
   SafeTrade 是小型托管交易所。USDT 提出去之后不要把余额留在上面。

A quoted PRL price (pearltrack showed about $1.45 on 2026-09-27; other tickers the same day were in a wide band) is not the fill you will get. Recompute take-home from the USDT that arrives.
标价（2026-09-27 pearltrack 大约 $1.45；同一天别的行情源差得很多）不是你的成交价。用到账的 USDT 复算到手。

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
| **★ SafeTrade (CEX)** | **0.1%** maker/taker | **USDT** (TRC-20 / BSC) | 1 PRL deposit | Order-book CEX (custodial) + **native PRL deposit** (0 deposit fee, 20 confs) | TRC-20 withdrawal **~0.08 USDT measured** (≪ Ethereum gas) |
| **lordofpearls OTC** | **1.8%** | USDC on **Ethereum** | none stated | Telegram bot `@LOPOTCBOT` (custodial trust) | ETH withdrawal gas |
| **pearl-otc.com** | **2%** (PRL leg) | USDC on **Arbitrum** | **1,000 PRL** | P2P order book + on-chain **2-of-2 multisig escrow** (non-custodial) | Arbitrum gas (cents) |
| **PearlBridge → Uniswap** | bridge fee (in-app) + **1%** DEX | WPRL on Ethereum | — | bridge + DEX | **ETH gas ×2 ($5–20+)** + slippage |

**Verdict / 结论:**
- **SafeTrade wins outright**: `0.1%` fee is ~18× cheaper than the OTCs, it's a real order book (not an OTC spread), accepts **native PRL deposit** (no bridge/mint), and pays out **USDT on TRC-20** that Binance credits free — sidestepping Ethereum gas entirely. / **SafeTrade 全面胜出**：0.1% 费率约为 OTC 的 1/18，真实订单簿、原生直充、USDT 走 TRC-20 且币安免费入账，彻底绕开以太坊 gas。
- Take-home @ ~$0.72, 0.1% fee, $1 withdrawal: 100 PRL → ~`$70.9` (SafeTrade) vs ~`$66.0` (lordofpearls); 200 PRL → ~`$142.9` vs ~`$133.0`. / 到手对比见第 7 节。
- ⚠️ SafeTrade is **custodial** — sell and withdraw promptly; never store balances. May require **KYC**; verify the deposit screen shows a `prl1p…` address and confirm the live TRC-20 USDT withdrawal fee before relying on it. / SafeTrade 是**托管**所——卖完即提、不久放；可能需 **KYC**；用前在充值页确认是 `prl1p…` 地址、并核对当时的 TRC-20 USDT 提现费。
- For the OTC fallbacks: the real cost is the **spread** (price on a thin book), not the 0.2% fee gap — always get a same-day quote. / OTC 备选的真正成本是**成交价差**，不是那 0.2%——务必拿当天报价。
- ⚠️ All four are unofficial third parties; this repo does not vouch for their safety. / 四家全是非官方第三方，本仓库不为其安全背书。

### 5.1 Real cash-out log (2026-06-09) / 实测案例 — 首次真实变现

A full end-to-end run, measured (not estimated). Native PRL mined on the rig → SafeTrade → Binance:
一次完整实跑的**实测**数据（非估算）。矿机挖的原生 PRL → SafeTrade → 币安：

| Step / 步骤 | Measured / 实测 |
|---|---|
| Sent native PRL → SafeTrade `prl1p…` deposit | 2 PRL test then 44 PRL; on-chain fee `0.0000155` + `0.0001945` PRL (negligible); 0 deposit fee; credited after **20 confs** (~20–40 min) / 先试 2 个再发 44 个；链上费极小；充值零费；20 确认后到账 |
| Sold on SafeTrade order book | ~46 PRL @ **~0.52 USDT/PRL avg**, 0.1% trade fee / 订单簿卖出，均价约 0.52 |
| Withdraw USDT → Binance (TRC-20) | fee **~0.08 USDT**, credited free on Binance / TRC-20 提现费仅约 0.08，币安免费入账 |
| **Net landed / 净到手** | **23.82 USDT** from the PRL |

**Lessons / 经验:**
- **Fee wear was tiny (~0.4%)** — 0.1% trade + ~0.08 USDT withdrawal. The SafeTrade route is genuinely cheap; TRC-20 withdrawal was ~`0.08 USDT`, not the ~`$1` first assumed. / **费用磨损仅 ~0.4%**——路子确实省，TRC-20 提现费远低于预想。
- **The real value gap was the FILL PRICE**: `0.52` realized vs the `~0.68` WPRL/DEX reference — a thin-book market sell ate the bid down. The cost was *price impact, not fees.* / **真正少拿的是成交价**（0.52 vs 参考 0.68），薄盘市价卖把买盘吃穿了——代价是**价格冲击，不是手续费**。
- Even at `0.52`, this small batch still beat bridge→DEX (where `0.68 × 46 ≈ $31` would be gutted by `$10–20` Ethereum gas → ~`$11–21`). For small sizes, **low fees beat a higher headline price.** / 即便 0.52，这点量仍胜过过桥→DEX（标价 0.68 但被以太坊 gas 吃掉）——小额下"费低"压倒"价高"。
- **Next time:** sell with patient **limit orders in smaller chunks** (don't market-dump a thin book). The June note to wait for 100–200 PRL was about amortizing a fee that, once measured, was ~0.08 USDT — see §2.1 and §7. / **下次**：挂限价、分批卖，别市价砸薄盘。6 月写的“攒到 100–200 再卖”是为了摊一笔后来实测只有 ~0.08 USDT 的费用——见 §2.1 和 §7。

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
| **★ SafeTrade (0.1% + TRC-20)** | **~0.08 USDT measured** | a test, then the rest | whatever is already in the wallet | not fee-bound |
| lordofpearls (1.8% + ETH gas) | ~$2–6 (×2 to forward) | ≥ $150–250 | ~200–330 PRL | ~1 month |
| pearl-otc (2% + Arbitrum) | ~$0.1–0.5 (negligible) | min-bound | **≥1,000 PRL** | ~4–5 months |
| bridge → Uniswap | ~$10–30 + slippage | ≥ $500–1,000 | ~700–1,300 PRL | 3–5 months |

**Trade-off / 权衡:** the TRC-20 fee measured in June was ~0.08 USDT, so waiting to "accumulate a week's mining" is no longer what makes the SafeTrade path worth it. Sell coins that are already in the wallet when you want them sold. Still use a **limit order**. The 7–8 PRL/day column above is the May 2026 RTX 3090 / AlphaPool case.
**权衡：** 6 月实测的 TRC-20 费用大约 0.08 USDT，所以不必再为了摊薄手续费而攒满一周。钱包里已经有的币，想卖就可以卖。仍然用**限价单**。上面每天 7–8 PRL 那一列是 2026 年 5 月 RTX 3090 / AlphaPool 的情况。

## 8. Safety checklist / 安全清单
- ❌ Never send native PRL to a Binance/OKX/Gate/Bitget/MEXC "PRL" address — that is a different asset or no deposit = loss. On SafeTrade, and on any other site, send only after the deposit address shown is `prl1p…`. / 绝不把原生 PRL 充到币安/欧易/Gate/Bitget/MEXC 的 “PRL” 地址——那是别的资产或根本充不进去 = 丢币。SafeTrade 以及其他任何网站，都要等充值地址显示为 `prl1p…` 再转。
- ❌ Never enter your seed/mnemonic into any website, bot, extension, or chat. Bridges/DEX only need wallet *connect + sign*. / 绝不把助记词输入任何网站/机器人/插件/聊天；桥和 DEX 只需钱包"连接+签名"。
- ✅ Verify the bridge URL and WPRL contract address against Pearl's official channels (ticker collision + phishing are rampant). / 逐字核对桥地址与 WPRL 合约（同名币+钓鱼泛滥）。
- ✅ Test the whole path with a tiny amount first; recompute real take-home from the actual USDC received, not the quoted price. / 先用小额跑通整条链路；用实际收到的 USDC 复算真实到手，别用标价。
- ✅ To manage native PRL with a GUI, use the **official Pearl desktop wallet**, not a generic multi-chain wallet (which can't derive `prl1p…` addresses). The CLI on this rig is `oyster` / `prlctl` 1.1.0. / 想用图形界面管原生 PRL，用**官方 Pearl 桌面钱包**，别用通用多链钱包（它推导不出 `prl1p…` 地址）。这台机器上的 CLI 是 `oyster` / `prlctl` 1.1.0。

---

References / 参考: safetrade.com/exchange/PRL-USDT · coingecko.com/en/coins/pearl-2 · hashrate.no/coins/PRL · coingecko.com/en/coins/wrapped-pearl · pearlbridge.xyz · otc.lordofpearls.xyz · pearl-otc.com · explorer.pearlresearch.ai
