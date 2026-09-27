# Pearl 挖矿会话整理

> 这是 2026-05-31 的会话记录。当前矿机、Kryptex 自动打款和提现步骤以 [README](README.md)、[docs/02](docs/02-deployment-runbook.md)、[docs/03](docs/03-tokens-and-cashout.md) 为准。

日期：2026-05-31  
主题：Pearl Research / PRL 挖矿可靠性、收益、电费、Binance 代币辨析、WSL 挖矿和钱包使用

> 本文只是技术整理和收益概算，不构成投资建议。Pearl/PRL 相关信息变化很快，实际收益以矿池 24 小时到账、可成交卖出价、设备功耗和网络状态为准。

## 1. 原文章可靠性分析

用户给出的微信文章链接：

https://mp.weixin.qq.com/s/IxxTSVWoQDqsyuot18QowA

该链接无法直接抓取，因此用可访问镜像和公开资料交叉核验。文章核心观点大致是：

- 租用 RTX 4090/5090 等 GPU 挖 Pearl。
- 当时按 PRL 价格约 `1.3-1.4 USDT` 估算。
- 文章认为 RTX 4090 租金低于约 `0.7 USDT/小时` 时可以赚钱。
- 收益计算公式类似：

```text
日净利润 = 单卡日产 PRL * PRL 价格 * (1 - 矿池费) - 租金/小时 * 24
```

可靠性判断：

- 公式本身正确。
- 项目叙事部分需要打折。Pearl 白皮书确实描述它是一个 Proof-of-Useful-Work L1，挖矿由矩阵乘法/AI 计算产生。
- Together AI 确实发布过与 Pearl Research Labs 的合作信息。
- 文章收益结论高度依赖当时币价、矿池收益、租卡价格和卖出流动性，不能当作固定结论。
- 文章带有租卡/平台推广属性，需要警惕利益相关。
- `PRL` 这个 ticker 有多个同名项目，极易混淆。

主要参考：

- BitMart/PANews 镜像文章：https://www.bitmart.com/zh-CN/news/detail/pearl-0-7u-69790
- Pearl 白皮书：https://pearlresearch.ai/Pearl_Whitepaper.pdf
- Together AI 合作公告：https://www.together.ai/blog/together-ai-partners-with-pearl-research-labs
- AGTI 风险分析：https://agti.net/intelligence-reports/2026/05/23/pearl-pouw-useful-work-asic-analysis/

## 2. 当时 Pearl 挖矿投入收益比概算

保守公开数据口径：

- PRL 价格参考 Hashrate.no，当时约 `$0.758`。
- 单 `TH/s` 日收益约 `$0.0587`。
- PearlHash/类似矿池费按约 `3%` 估。
- AlphaPool 明示矿池费 `5%`，miner dev fee `1%`，合计会更高。

RTX 4090 算力参考：

- 普通口径约 `205 TH/s`。
- AlphaPool alpha-miner 口径约 `245-255 TH/s`。

租卡价格参考：

- RunPod RTX 4090 公开价格约 `$0.69/小时`。
- RentGPU 聚合器显示过更低市场价，例如 `$0.32/小时`，但低价通常有供给、稳定性和中断风险。

保守测算结果：

| 场景 | 日收入净额 | 日成本 | 收入/成本 | 日净利润 |
|---|---:|---:|---:|---:|
| 205 TH/s，租金 $0.69/hr | $11.67 | $16.56 | 0.70x | -$4.89 |
| 250 TH/s，租金 $0.69/hr | $14.23 | $16.56 | 0.86x | -$2.33 |
| 255 TH/s，租金 $0.69/hr | $14.52 | $16.56 | 0.88x | -$2.04 |
| 205 TH/s，租金 $0.32/hr | $11.67 | $7.68 | 1.52x | +$3.99 |
| 250 TH/s，租金 $0.32/hr | $14.23 | $7.68 | 1.85x | +$6.55 |

结论：

- `0.7 USDT/小时以下就赚钱` 这个结论已经不应直接照抄。
- 保守口径下，4090 盈亏线大致约 `$0.49-0.61/小时`。
- 如果租金是 `$0.69-0.70/小时`，保守估算大概率亏损或接近亏损。
- 如果能稳定租到 `$0.32/小时` 且算力正常，才有较明显利润空间。

## 3. 用户自有 RTX 3090 收益和电费概算

用户设备和电价：

- GPU：一张 RTX 3090。
- 峰时电价：`0.63 RMB/kWh`。
- 谷时电价：`0.34 RMB/kWh`。
- 谷时：`22:00-次日 08:00`，共 10 小时。
- 峰时：其余 14 小时。

分时平均电价：

```text
(14 * 0.63 + 10 * 0.34) / 24 = 0.509 RMB/kWh
```

3090 Pearl 挖矿参考参数：

- AlphaPool 口径：RTX 3090 约 `100-110 TH/s`。
- Hashrate.no 口径：3090 约 `110 TH/s`，效率约 `0.344 TH/W`。
- GPU 功耗估算：约 `291-320 W`。

24 小时一直挖的估算：

| 项目 | 估算 |
|---|---:|
| 算力 | 100-110 TH/s |
| 仅 GPU 功耗 | 约 291-320 W |
| 日产 PRL | 约 8.4-9.25 PRL |
| 日毛收入 | 约 43.2-47.5 RMB |
| 扣 3%-6% 费用后 | 约 40.6-46.1 RMB |
| 电费，峰 14h + 谷 10h | 约 3.55-3.91 RMB/天 |
| 日净收益 | 约 37-42 RMB/天 |

按 `320 W` 拆分电费：

| 时段 | 电量 | 电费 |
|---|---:|---:|
| 谷时 22:00-08:00，10h | 3.20 kWh | 1.09 RMB |
| 峰时 14h | 4.48 kWh | 2.82 RMB |
| 合计 | 7.67 kWh | 3.91 RMB/天 |

只在谷时挖 10 小时：

- 日产约 `3.85 PRL`。
- 毛收入约 `19.8 RMB`。
- 扣费后约 `18.6-19.2 RMB`。
- 电费约 `1.09 RMB`。
- 净收益约 `17.5-18.1 RMB/天`。

注意：

- 上面是显卡功耗口径。
- 如果整机墙插功耗是 `380-420 W`，24 小时电费约 `4.6-5.1 RMB/天`，净收益会再少约 1 元。

## 4. Binance 上的 PEARL/PRL 是什么

结论：Binance 上看到的 `PRLUSDT` 指的是 **Perle (PRL)**，不是前面讨论的 **Pearl Research / 珍珠币**。

Binance 合约公告显示：

- 交易对：`PRLUSDT` 永续合约。
- 上线时间：2026-04-01 10:30 UTC。
- 标的资产：Perle (PRL)。
- 项目：Perle Labs，Solana 上的 Web3 + AI 平台。
- 最高杠杆：20x。
- 结算资产：USDT。

关键提醒：

- 不要把挖出来的 Pearl Research PRL 直接充到 Binance 的 `PRL` 地址。
- 只看 ticker `PRL` 是不够的，必须核对项目、链和合约地址。
- 错币/错链充值可能导致不到账或无法找回。

参考：

- Binance 公告镜像：https://t.co/sMhJfqyR3s

## 5. 挖矿币 PRL 的上市状态与变现路径（关键补充）

这一节回答最核心的问题：**挖出来的币能不能、以及怎么卖成钱。** 前面 §2、§3 的收益都是按 hashrate.no 标价算的"账面收益"，能不能真正到手取决于本节。

### 5.1 上市状态：原生 PRL 没上任何 CEX

- 你挖的是 **Pearl L1 的原生币 native PRL**。
- **截至 2026-05-31，native PRL 没有上任何中心化交易所（CEX）**（Binance / OKX / Gate / Bitget / MEXC 等都没有）。
- 唯一的交易/变现场所是：把原生 PRL 通过 **PearlBridge 跨链桥**包装成 **WPRL（Wrapped Pearl，以太坊 ERC-20）**，再到以太坊上的 **Uniswap V3 / V4（DEX）** 卖出。**WPRL 也只有 DEX，仍无 CEX。**
- 易被误导的点：CoinGecko、Coinbase 等会给 Base 链上的 DEX 代币做"价格信息页"，**能查到价格 ≠ CEX 在真的撮合交易，也 ≠ 能在 CEX 充提**。而且 Base 上还存在同名/山寨 PRL 合约。
- 对比 §4：币安的 **Perle (PRL)** 确实是 CEX 币，但那是**另一个项目**，且已于 2026-04-28 下架——和你挖的币完全无关。

### 5.2 变现路径

```text
native PRL（Pearl L1，你挖到的）
  → PearlBridge 锁仓 → 铸造 WPRL（以太坊 ERC-20）
  → Uniswap V3/V4 卖成 USDT/WETH（DEX，约 1% 池费 + 以太坊 gas）
  → 提到 CEX 或 OTC 换法币
```

每一跳都有成本/风险：跨链桥风险、以太坊 gas、约 1% 池费、滑点。

### 5.3 流动性现状（决定能不能真卖出）

- WPRL 只在 **Uniswap（DEX）** 交易，无 CEX。
- WPRL/USDT 24h 成交量约 **$24–47 万**；池子深度约 **$13–29 万**（不同来源口径不一）。
- 而全网矿工**每天新产出 PRL ≈ $148 万**（约 730 块/天 × 块价值 $2,024）。
- → **每天新挖出来的币价值，远超变现场所一天能吸收的成交量。** 这意味着标价（hashrate.no $0.758）是脆弱的薄盘价，**放量卖会砸盘 + 高滑点**。
- 价格极端波动：ATH 约 **$1.649（2026-05-29）**，两天后跌到 $0.758，几近腰斩。

### 5.4 对个人矿工的实际含义

- 一张 3090 一天 ~9 PRL（~$7），单笔卖出本身不会砸盘，能成交。
- 但你真正到手的**不是"标价 × 数量"**——要扣：跨链桥风险、以太坊 gas、约 1% 池费、滑点。
- 真实收益 ≈ "实际在 Uniswap 换成 USDT 的金额"，通常**明显低于** hashrate.no 标价。
- 跨链桥（PearlBridge / Pilcrow Bridge 标注 *Experimental*）是黑客头号攻击目标，有归零风险——**不要长期把大量币留在桥上或 WPRL 形态**。

### 5.5 变现前的安全要点

- **绝不**把原生 PRL 充到任何叫"PRL"的 CEX 充值地址（那是 Perle，不同链不同币，充错直接丢失）。
- 先小额跑通整条链路：挖一点 → 过桥 → Uniswap 换出一笔 → 确认到账 → 再决定规模。
- 用"过桥后实际换成 USDT 到手的金额"重新核算 §2、§3 的收益，不要直接用账面标价。

参考：

- WPRL 行情/流动性（CoinGecko）：https://www.coingecko.com/en/coins/wrapped-pearl
- WPRL/ETH Uniswap V4 池（GeckoTerminal）：https://www.geckoterminal.com/eth/pools/0x44c98565f6df4ac7660761cad1ebfd5343f0b397ff89837d3e1c9b5c2ac6c42b
- PearlBridge（PRL ↔ WPRL）：https://pearlbridge.xyz/
- hashrate.no PRL 网络数据：https://www.hashrate.no/coins/PRL

## 6. WSL 上挖 Pearl 的流程

目标：在 Windows + WSL2 + NVIDIA RTX 3090 环境中挖 Pearl。

### 6.1 Windows 和 WSL 准备

PowerShell 中执行：

```powershell
wsl --update
wsl --shutdown
```

安装或升级 NVIDIA Windows 驱动。WSL 里不要安装 Linux 显卡驱动。

进入 WSL 后检查：

```bash
nvidia-smi
```

如果能看到 RTX 3090，就说明 WSL 能访问 GPU。

Ubuntu 官方提醒：

- WSL2 的 CUDA driver 来自 Windows 驱动。
- 不要在 WSL Ubuntu 里安装会带 Linux driver 的 `cuda`、`cuda-13`、`cuda-drivers` 包。

参考：

- Ubuntu WSL CUDA 文档：https://documentation.ubuntu.com/wsl/latest/howto/gpu-cuda/

### 6.2 安装基础工具

```bash
sudo apt update
sudo apt install -y curl ca-certificates tmux
mkdir -p ~/pearl-mining
cd ~/pearl-mining
```

### 6.3 下载 AlphaPool miner

```bash
curl -L -o alpha-miner https://pearl.alphapool.tech/downloads/alpha-miner
chmod +x alpha-miner
```

### 6.4 运行 miner

把 `prl1p你的钱包地址` 替换成自己的 Pearl 钱包地址：

```bash
./alpha-miner \
  --pool stratum+tcp://sg1.alphapool.tech:5566 \
  --address prl1p你的钱包地址 \
  --worker wsl-3090 \
  --password "x;d=262144" \
  --status-interval 60
```

说明：

- 亚洲地区优先试 `sg1.alphapool.tech:5566`。
- 也可试 `us2.alphapool.tech:5566`、`eu1.alphapool.tech:5566`。
- RTX 3090 推荐静态难度 `262144`。
- AlphaPool 页面显示 3090 预期约 `100-110 TH/s`。
- AlphaPool 明示 pool fee `5%`，miner dev fee `1%`。

参考：

- AlphaPool Pearl：https://pearl.alphapool.tech/

### 6.5 后台运行

```bash
tmux new -s pearl
```

在 tmux 里粘贴 miner 命令。退出但保持运行：

```text
Ctrl+B
D
```

回到日志：

```bash
tmux attach -t pearl
```

## 7. 生成 Pearl 钱包地址

官方 release 提供 Linux CLI 包：

- `go-binaries-linux-amd64-v1.0.2.tar.gz`
- 包含 `oyster`、`prlctl`、`pearld`

下载：

```bash
mkdir -p ~/pearl-wallet
cd ~/pearl-wallet

curl -L -o pearl-cli.tar.gz \
  https://github.com/pearl-research-labs/pearl/releases/download/pearl-wallet-v1.0.0/go-binaries-linux-amd64-v1.0.2.tar.gz

curl -L -o checksums.txt \
  https://github.com/pearl-research-labs/pearl/releases/download/pearl-wallet-v1.0.0/checksums.txt

sha256sum -c checksums.txt --ignore-missing
```

看到类似结果再继续：

```text
go-binaries-linux-amd64-v1.0.2.tar.gz: OK
```

解压：

```bash
tar -xzf pearl-cli.tar.gz
find . -type f \( -name oyster -o -name prlctl -o -name pearld \)
```

如果程序在当前目录，后续使用 `./oyster` 和 `./prlctl`。如果在子目录，把命令中的路径替换成实际路径。

创建钱包：

```bash
./oyster -u rpcuser -P rpcpass --create
```

安全注意：

- 助记词离线保存。
- 不要截图。
- 不要发给任何人。
- `rpcuser/rpcpass` 是本机 RPC 访问账号密码，不是助记词。
- 钱包密码不要和矿池、交易所密码相同。

启动钱包服务：

```bash
./oyster -u rpcuser -P rpcpass
```

另开一个 WSL 窗口生成地址：

```bash
cd ~/pearl-wallet
./prlctl -u rpcuser -P rpcpass -s https://localhost:44207 getnewaddress
```

如果 TLS 报错：

```bash
./prlctl -u rpcuser -P rpcpass -s localhost:44207 --notls getnewaddress
```

输出应为：

```text
prl1p...
```

这个地址就是挖矿命令中的 `--address`。

参考：

- Pearl GitHub Release：https://github.com/pearl-research-labs/pearl/releases/tag/pearl-wallet-v1.0.0
- Pearl README：https://github.com/pearl-research-labs/pearl

## 8. 如何访问这个钱包

Pearl CLI 钱包不是网页钱包。它由两部分组成：

- `oyster`：钱包守护进程，保存私钥/助记词并负责签名。
- `prlctl`：命令行控制工具，用来和钱包服务通信。

默认钱包服务端口：

```text
localhost:44207
```

每次访问钱包：

1. 启动钱包服务。

```bash
cd ~/pearl-wallet
./oyster -u rpcuser -P rpcpass
```

2. 另开一个 WSL 窗口，用 `prlctl` 操作。

```bash
cd ~/pearl-wallet
./prlctl -u rpcuser -P rpcpass -s localhost:44207 --notls help
```

常用命令：

```bash
# 查看帮助
./prlctl -u rpcuser -P rpcpass -s localhost:44207 --notls help

# 生成新的收款地址
./prlctl -u rpcuser -P rpcpass -s localhost:44207 --notls getnewaddress

# 查看余额，如果当前版本支持该命令
./prlctl -u rpcuser -P rpcpass -s localhost:44207 --notls getbalance
```

图形钱包：

- 官方 release 里有 Windows 桌面钱包安装包 `Pearl-Wallet-Setup-1.0.0.exe`。
- 如果已经在 WSL 创建过钱包，可以在桌面钱包里选择恢复/导入钱包，用之前保存的助记词恢复。

## 9. 操作风险清单

- 挖的原生 PRL 目前没上任何 CEX，只能过桥成 WPRL 在 Uniswap(DEX) 卖；变现盘极薄，标价不等于真实到手价（见 §5）。
- 不要把 Pearl Research PRL 和 Binance 上的 Perle PRL 混淆。
- 不要把挖矿地址填成交易所里同名 PRL 的充值地址。
- 不要把助记词输入到矿池、Telegram、网页表单或陌生工具中。
- WSL 里不要安装 Linux NVIDIA driver。
- 挖矿前先用 1 张卡跑 24 小时，按实际到账 PRL 和实际卖出价复算。
- RTX 3090 注意显存温度，建议优先在 Windows 侧用工具控制功耗和风扇。
- 收益估算会随币价、难度、矿池出块、手续费、网络延迟和卖出滑点快速变化。

## 10. 本会话中的关键结论

1. 文章收益公式可用，但 `0.7U/小时以下就赚` 这个阈值已经不稳。
2. 保守口径下，RTX 4090 租卡盈亏线约 `$0.49-0.61/小时`。
3. 用户自有 RTX 3090 在当前估算下，24 小时挖矿净收益约 `37-42 RMB/天`，只谷时挖约 `17.5-18.1 RMB/天`。
4. Binance 上 `PRLUSDT` 是 Perle，不是 Pearl Research 挖矿币。
5. WSL 挖 Pearl 可用 AlphaPool alpha-miner，3090 推荐静态难度 `262144`。
6. Pearl 钱包用官方 `oyster` 创建，用 `prlctl` 生成 `prl1p...` 挖矿收款地址。
7. 挖的是 Pearl L1 原生 PRL，**截至 2026-05-31 没上任何 CEX**；只能通过 PearlBridge 过桥成 WPRL，到以太坊 Uniswap(DEX) 卖出。变现盘极薄（WPRL 日成交约 $24–47 万、池深约 $13–29 万），而全网日发行约 $148 万，**账面标价 ≠ 真实到手价**，且依赖实验性跨链桥，有安全风险（见 §5）。
