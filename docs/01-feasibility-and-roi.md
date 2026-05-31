<!--
title: Feasibility & ROI of Pearl (PRL) GPU mining
description: Honest feasibility/ROI analysis — rented RTX 4090 break-even vs an owned RTX 3090 with time-of-use electricity, plus the liquidity caveat that decides real take-home.
language: [en, zh]
updated: 2026-05-31
tags: [roi, feasibility, electricity, rtx-3090, rtx-4090]
-->

# 01 · Feasibility & ROI / 可行性与回报率

> ⚠️ Snapshot 2026-05-31, conservative public-data basis. The decisive variable is **not** hashrate but **whether you can actually sell the mined PRL** — see [03 · Tokens & cash-out](03-tokens-and-cashout.md).
> ⚠️ 2026-05-31 快照、保守公开数据口径。决定性变量**不是**算力，而是**挖到的 PRL 能不能真卖出去**——见 [03 · 代币与变现](03-tokens-and-cashout.md)。

## TL;DR / 一句话结论
- **Rented GPUs**: marginal-to-loss at typical rental prices. Don't enter via rental.
  **租卡**：常见租价下接近亏损。不要靠租卡入场。
- **Owned GPU + cheap (off-peak) electricity**: positive on paper, because marginal cost is just electricity. But "on paper" uses the quoted price, which is **not** realizable due to thin liquidity.
  **自有显卡 + 便宜（谷时）电**：账面为正，因为边际成本只剩电费。但"账面"用的是标价，受薄盘限制**并不能真兑现**。
- Treat it as a **small speculative experiment**, not income.
  把它当**小额投机实验**，不是收入来源。

## 1. Pricing basis / 计价口径
- PRL price (hashrate.no): ~`$0.758` (volatile; ATH ~`$1.649` on 2026-05-29). / PRL 价格参考 hashrate.no，约 `$0.758`（剧烈波动，2026-05-29 ATH 约 `$1.649`）。
- Revenue per `TH/s/day`: ~`$0.0587–0.0627`. / 单 `TH/s` 日收益约 `$0.0587–0.0627`。
- Pool fee: AlphaPool **5% + 1% dev**; some pools ~3%. / 矿池费：AlphaPool **5% + 1% dev**；部分矿池约 3%。

## 2. Rented RTX 4090 — break-even / 租用 RTX 4090 盈亏

Hashrate ~205 TH/s (generic) to ~245–255 TH/s (alpha-miner). Rental ~`$0.69/hr` (RunPod) down to ~`$0.32/hr` (aggregators, less stable).
算力约 205 TH/s（普通口径）至 245–255 TH/s（alpha-miner）。租金约 `$0.69/小时`（RunPod）低至 `$0.32/小时`（聚合器，稳定性差）。

| Scenario / 场景 | Net revenue/day / 日净收入 | Cost/day / 日成本 | Rev/Cost | Net profit/day / 日净利 |
|---|---:|---:|---:|---:|
| 205 TH/s, $0.69/hr | $11.67 | $16.56 | 0.70x | -$4.89 |
| 250 TH/s, $0.69/hr | $14.23 | $16.56 | 0.86x | -$2.33 |
| 255 TH/s, $0.69/hr | $14.52 | $16.56 | 0.88x | -$2.04 |
| 205 TH/s, $0.32/hr | $11.67 | $7.68 | 1.52x | +$3.99 |
| 250 TH/s, $0.32/hr | $14.23 | $7.68 | 1.85x | +$6.55 |

**Conclusion / 结论:** "Profitable below 0.7 USDT/hr" is outdated. Conservative break-even ≈ `$0.49–0.61/hr`. At `$0.69–0.70/hr` you likely lose money; only stable `$0.32/hr` leaves clear margin.
"低于 0.7U/小时就赚" 已过时。保守盈亏线约 `$0.49–0.61/小时`。`$0.69–0.70/小时` 大概率亏；只有稳定 `$0.32/小时` 才有明显利润。

## 3. Owned RTX 3090 + time-of-use power / 自有 RTX 3090 + 分时电价

Example tariff / 示例电价: peak `0.63 RMB/kWh` (14h), off-peak `0.34 RMB/kWh` (22:00–08:00, 10h) → blended `0.509 RMB/kWh`.
3090: ~100–110 TH/s, ~291–320 W (GPU-only).

| Item / 项 | 24/7 mining / 全天挖 |
|---|---:|
| Hashrate / 算力 | 100–110 TH/s |
| GPU power / 仅显卡功耗 | ~291–320 W |
| PRL/day / 日产 | ~8.4–9.25 PRL |
| Gross/day / 日毛收入 | ~43.2–47.5 RMB |
| After 3–6% fees / 扣费后 | ~40.6–46.1 RMB |
| Electricity / 电费 (14h peak + 10h off-peak) | ~3.55–3.91 RMB |
| **Net/day / 日净** | **~37–42 RMB** |

**Off-peak only (10h) / 只谷时挖 10h:** ~3.85 PRL → ~19.8 RMB gross → ~17.5–18.1 RMB net (electricity ~1.09 RMB).
**Wall power caveat / 整机功耗:** if 380–420 W at the wall, electricity ~4.6–5.1 RMB/day and net drops ~1 RMB.

## 4. Why "on paper" ≠ realized / 为什么"账面"≠落袋
The tables above value PRL at the quoted `$0.758`. But native PRL has **no CEX**, the only market (bridged WPRL on a DEX) is thin, and **daily network issuance (~$1.48M) exceeds 24h trading volume (~$1.1M)** — structural sell pressure. Real take-home after bridge + DEX/OTC fees + slippage is materially lower. Full detail in [03 · Tokens & cash-out](03-tokens-and-cashout.md).
上面的表把 PRL 按标价 `$0.758` 计。但原生 PRL **无 CEX**，唯一市场（桥到 DEX 的 WPRL）很薄，且**全网日发行(~$148万) > 24h 成交量(~$110万)**——结构性抛压。扣掉桥+DEX/OTC 费用+滑点后真实到手明显更低。详见 [03](03-tokens-and-cashout.md)。

## 5. Verdict / 裁决
| Question / 问题 | Answer / 答案 |
|---|---|
| Technically feasible? / 技术可行？ | ✅ Yes (see [02 runbook](02-deployment-runbook.md)) |
| Rented 4090 worth it? / 租 4090 值？ | ❌ No, negative expected value |
| Owned 3090 + off-peak? / 自有 3090 + 谷时？ | ⚠️ Positive on paper; real value gated by liquidity. OK as a small experiment |
| Reliable income? / 稳定收入？ | ❌ No — speculative bet on an illiquid, declining early-stage token |

**Recommended posture / 建议姿态:** run a 24–48h trial, record **actual PRL credited** at the pool, then compute **real take-home** through the cash-out path before scaling. Prefer off-peak-only to minimize electricity risk.
跑 24–48h 验证，记录矿池**实际到账 PRL**，按变现链路算出**真实到手**再决定是否扩大。优先只谷时挖以压低电费风险。
