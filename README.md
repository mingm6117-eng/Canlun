# Canlun · TradingView 缠论核心指标

> 「走势终完美。」
> 「买点买，卖点卖。」

![Pine Script v6](https://img.shields.io/badge/Pine_Script-v6-00B2FF.svg)
![TradingView Indicator](https://img.shields.io/badge/TradingView-Indicator-2962FF.svg)
![ChanLun](https://img.shields.io/badge/ChanLun-Core-orange.svg)
![Status](https://img.shields.io/badge/status-v1_beta-yellow.svg)

**一个可以直接粘贴到 TradingView Pine Editor 的缠论核心指标。**

它不是普通均线、MACD 或震荡指标，而是把缠论里最基础、最可程序化的一层结构做成 TradingView 图上工具：

- K 线包含关系处理
- 顶/底分型识别，但默认不显示分型标签，避免满屏拥挤
- 确认笔绘制
- 笔中枢绘制
- 一买、二买、三买、一卖、二卖、三卖中文标注
- 中文 alert 条件
- 主图 overlay 显示，线、框、标签都画在 K 线图里

[快速使用](#快速使用) · [指标效果](#指标效果) · [实现了什么](#实现了什么) · [参数说明](#参数说明) · [边界与风险](#边界与风险)

---

## 指标效果

把 `chanlun_tradingview.pine` 复制进 TradingView 后，指标会在主 K 线图上显示：

- 蓝色/红色折线：确认笔
- 橙色半透明框：笔中枢
- 中文买卖点标签：`一买`、`二买`、`三买`、`一卖`、`二卖`、`三卖`
- 买点标签在价格下方，卖点标签在价格上方，并带 ATR 偏移，减少遮挡

它适合用来回答这类问题：

**问：这段走势有没有形成中枢？**

> 看橙色框。连续三笔价格区间发生重叠，就会生成一个笔中枢。后续笔继续和中枢重叠，指标会延展中枢；若离开中枢并回抽不重新进入，则进入三买/三卖判断。

---

**问：为什么有时没有一买/一卖？**

> 默认开启 MACD 背离辅助。一买需要中枢外低点出现价格新低但 MACD 不再新低；一卖反过来，需要中枢外高点价格新高但 MACD 不再新高。这样会更保守，少一些噪音信号。

---

**问：为什么二买/二卖少了？**

> 二买必须高于一买低点，并且至少回到中枢下沿；二卖必须低于一卖高点，并且至少回到中枢上沿。这个过滤是为了减少假二买/假二卖。

---

## 快速使用

1. 打开文件：[`chanlun_tradingview.pine`](./chanlun_tradingview.pine)
2. 全选复制全部内容
3. 打开 TradingView 图表
4. 进入 `Pine Editor`
5. 删除编辑器里的旧内容
6. 粘贴脚本
7. 点击 `Save`
8. 点击 `Add to chart`

第一行必须是：

```pine
//@version=6
```

如果你看到第一行是 `TradingView ChanLun Core Indicator Notes`，说明你粘错了文件。不要粘贴 `chanlun_implementation_notes.md`，那个只是说明文档。

---

## 实现了什么

### 1. 包含关系处理

缠论分型之前必须先处理 K 线包含关系。脚本按顺序合并包含 K 线：

- 向上时，包含组等价为 `[max low, max high]`
- 向下时，包含组等价为 `[min low, min high]`

这一步是后续分型、笔、中枢的地基。

### 2. 分型识别

脚本内部识别顶分型和底分型：

- 顶分型：中间 K 线的高点和低点都高于两侧
- 底分型：中间 K 线的高点和低点都低于两侧

默认不显示 `顶/底` 标签，因为它们太密集，但它们仍然参与笔的生成。

### 3. 确认笔

笔由交替的顶/底分型构成：

- 底到顶：向上一笔
- 顶到底：向下一笔
- 同类分型连续出现时，只保留更极端的那个
- 最小成笔间隔使用原始 K 线数量，由参数 `Minimum raw bars per stroke` 控制

### 4. 笔中枢

连续三笔存在价格区间重叠时，形成一个笔中枢。

中枢区间用三笔区间交集计算：

```text
ZD = max(low1, low2, low3)
ZG = min(high1, high2, high3)
```

当 `ZD <= ZG` 时，中枢成立。

### 5. 三类买卖点

脚本实现了确定性、TradingView 友好的买卖点标注：

- `一买`：中枢外下方低点，配合可选 MACD 底背离
- `二买`：一买后更高的底，并且至少回到中枢下沿
- `三买`：向上离开中枢后，回抽低点不跌回中枢上沿
- `一卖`：中枢外上方高点，配合可选 MACD 顶背离
- `二卖`：一卖后更低的顶，并且至少回到中枢上沿
- `三卖`：向下离开中枢后，回抽高点不升回中枢下沿

### 6. Alert 条件

脚本内置 6 个中文 alert：

- 缠论确认一买
- 缠论确认二买
- 缠论确认三买
- 缠论确认一卖
- 缠论确认二卖
- 缠论确认三卖

---

## 参数说明

### Display

| 参数 | 说明 |
|---|---|
| `Show strokes` | 是否显示笔线 |
| `Show stroke centers` | 是否显示笔中枢框 |
| `Show buy/sell labels` | 是否显示中文买卖点标签 |

### Limits

| 参数 | 默认值 | 说明 |
|---|---:|---|
| `Max stroke lines` | 120 | 最多保留多少根笔线 |
| `Max center boxes` | 40 | 最多保留多少个中枢框 |
| `Max labels` | 160 | 最多保留多少个标签 |
| `Max data history` | 500 | 内部数据数组最多保留多少条，防止长周期卡顿 |

### Sensitivity

| 参数 | 默认值 | 说明 |
|---|---:|---|
| `Minimum raw bars per stroke` | 4 | 成笔最小原始 K 线间隔 |
| `Strict fractals` | true | 是否使用严格分型 |
| `Require MACD divergence for first buy/sell` | true | 一买/一卖是否要求 MACD 背离 |
| `Signal label ATR offset` | 0.15 | 买卖点标签偏移距离，减少遮挡 |

---

## 文件结构

```text
Canlun/
├── chanlun_tradingview.pine          # TradingView 主指标脚本
├── chanlun_implementation_notes.md   # OCR 来源、规则取舍和实现说明
└── README.md                         # 项目说明
```

---

## 设计取舍

这个指标刻意做成“核心实用版”，不是完整缠论递归引擎。

它优先保证：

- 可以在 TradingView Pine v6 编译运行
- 主图上可读，不把图打爆
- 绘图对象数量受控
- 信号尽量确认后出现，而不是实时漂移
- 规则可解释，可继续迭代

它暂时不做：

- 完整线段递归
- 多级别联立
- 走势类型自动命名
- 完整背驰体系
- TradingView Strategy 回测下单

---

## 边界与风险

这不是投资建议，也不是自动交易系统。

缠论本身有大量人工判读空间，尤其是级别、线段、走势类型、背驰、区间套等部分。这个脚本只实现其中最基础、最适合程序化的一层结构。

使用时请注意：

- 买卖点是结构提示，不是必然盈利保证
- 指标使用确认 K 线，所以信号天然滞后
- 小周期噪音会明显增加
- 不同市场、不同周期需要调整参数
- 如果用于实盘，请先在历史图上手动复核

---

## 规则来源

规则整理自用户提供的扫描版《市场哲学的数学原理完全配图手册》OCR 摘取结果，重点参考：

- 第 18 课：走势中枢定义、走势终完美
- 第 20 课：走势中枢级别扩张及第三类买卖点
- 第 21 课：三类买卖点完备性
- 第 62、65 课：分型、笔、线段、包含关系
- 第 67、77、78、83 课：线段与最小中枢相关定义

详细取舍见 [`chanlun_implementation_notes.md`](./chanlun_implementation_notes.md)。

---

## 后续路线

- [ ] 增加可选线段层
- [ ] 增加更完整的 MACD 背驰判断
- [ ] 增加多级别中枢对照
- [ ] 增加策略回测版 `strategy()`
- [ ] 增加截图示例
- [ ] 增加 TradingView 编译问题 FAQ

---

## 免责声明

本项目仅用于技术研究、图形标注和交易系统学习。市场有风险，任何买卖决策都应由使用者自行承担。
