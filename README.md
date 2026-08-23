# LLM Backdoor 检测 · 领域追踪

**→ 在线版:https://bd.c4a4d65b.xyz/**

一张数据驱动的**立场图**:把 LLM backdoor(数据投毒)检测领域画成签名边网络——谁继承谁、谁佐证谁、谁证伪谁。
起点是 ToxScreen(arXiv 2607.26849)的 Related Work,但图本身不以任何单篇为中心。

> ### 这是什么 / 不是什么
>
> **这是一名二进制安全研究者在自学 LLM backdoor 方向时,借助 AI 整理出来的领域索引。**
> 要解决的问题很具体:这个方向的论文散在 arXiv、安全顶会、LessWrong 三个互不相通的地方,
> 一百多篇里谁在反驳谁没人画过。它的用途是**快速检索与分类**——
> 迅速定位「哪条线还在打、该先读哪篇、这篇的对手是谁」,并给出直达原文的链接。
>
> **它不是综述,不应当被当作引用来源。**
>
> - 图上每条 note、每张战线卡的裁定(「已证伪」「已反转」「未决」)都是**编者的概括与判断**,
>   基于摘要或原文整理,**不是原作者的表述,也不代表领域共识**。任何一条要用之前,请点开链接读原文。
> - **「常被误读」标记针对的是这些论文被转述和引用的方式,不是被标注论文本身。**
>   恰恰相反——被标的通常正是这个领域最重要、被引最多的那几篇。
> - **「核验状态」标的是编者是否亲自核对过书目**(标题 / 作者 / 年份 / id),**不是**对该论文结论正确性的背书。
> - **整理过程有 AI 参与,出过错。**上一轮全量核实抓出约 8% 凭记忆填写的数字有误(已逐条回原文订正,留档在 [MAINTENANCE.md](MAINTENANCE.md))。
>   **发现错误、想补论文、觉得某条边画错了,请开 [Issue](https://github.com/flora2627/llm-backdoor-tracker/issues)。**

配套长文:`../2026-08-07-llm-backdoor-detection-研究脉络.md`(九条主干逐条展开、时间线、benchmark 对照表、五个可下手的开放问题)

**专题报告:**[Sleeper Agents 这条研究线，到底讲了什么](reports/sleeper-agents-research-line.zh.md)——大白话讲清 AutoPoison 上游、原论文、后续复现、检测、工具调用和清洗实验。

---

## 一分钟上手

```bash
cd notes/llm-backdoor-tracker
python3 build.py --open      # 生成 index.html 并打开
```

无依赖,纯标准库。`index.html` 是**单文件**——数据全部内联,双击就能看,也能直接发布。

---

## 目录

```
llm-backdoor-tracker/
├── README.md          ← 本文件
├── MAINTENANCE.md     ← 维护笔记:更新流程 / 图的约定 / 历史留档
├── LICENSE            ← 内容 CC BY 4.0 / 代码 MIT
├── build.py           ← 唯一的构建入口(带自检)
├── template.html      ← 页面骨架 + CSS + 渲染器,不含数据
├── index.html         ← 生成产物,不要手改
└── data/
    ├── papers.json    ← 条目库(112) ← 主要在这里加东西
    ├── edges.json     ← 签名边(183) ← 和这里
    ├── fronts.json    ← 交火战线卡片(13)
    ├── misread.json   ← 「常被系统性误读」标记(49)
    ├── lanes.json     ← 九条主干泳道
    └── years.json     ← 时间轴刻度 [年份, x 坐标]
```

**只改 `data/*.json`,然后跑 `build.py`。** 改样式才动 `template.html`。

---

**维护相关的东西(每周更新流程、页面结构、图的设计约定、现状快照、历次更新与订正留档、待办)
全部在 [MAINTENANCE.md](MAINTENANCE.md)。**

---

## 许可

| | |
|---|---|
| **内容**(note / 战线卡 / 正文 / 仓库内文档 / 图的结构与呈现) | [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) |
| **代码**(`build.py`、`template.html` 里的 CSS 与 JS) | MIT |
| **书目信息**(标题 / 作者 / 年份 / venue / arXiv id / URL) | 事实,不主张任何权利 |

论文本身的著作权归各自作者。本项目**只链接原文,不复制论文正文或图表**;
页面中少量原文短句引用均标注出处并用于评论。详见 [LICENSE](LICENSE)。
