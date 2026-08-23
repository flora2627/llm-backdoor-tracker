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
> - **整理过程有 AI 参与,出过错。**上一轮全量核实抓出约 8% 凭记忆填写的数字有误(已逐条回原文订正,见文末留档)。
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
├── LICENSE            ← 内容 CC BY 4.0 / 代码 MIT
├── build.py           ← 唯一的构建入口(带自检)
├── template.html      ← 页面骨架 + CSS + 渲染器,不含数据
├── index.html         ← 生成产物,不要手改
└── data/
    ├── papers.json    ← 条目库(112) ← 主要在这里加东西
    ├── edges.json     ← 签名边(169) ← 和这里
    ├── fronts.json    ← 交火战线卡片(13)
    ├── misread.json   ← 「常被系统性误读」标记(31)
    ├── lanes.json     ← 九条主干泳道
    └── years.json     ← 时间轴刻度 [年份, x 坐标]
```

**只改 `data/*.json`,然后跑 `build.py`。** 改样式才动 `template.html`。

---

## 每周更新流程

### 1. 扫新论文

固定几个入口(经验:这个领域的东西七成先落在 arXiv):

- arXiv `cs.CR` × `cs.LG` 交叉:`backdoor LLM`、`data poisoning language model`、`trigger inversion`、`poisoned model detection`
- **别用网页搜索,直接打 API 按日期倒排**——2026-08-20 这轮验证有效,两条 query 就能覆盖一个月:
  ```
  export.arxiv.org/api/query?search_query=abs:backdoor+AND+cat:cs.CR&sortBy=submittedDate&sortOrder=descending&max_results=50
  export.arxiv.org/api/query?search_query=(abs:backdoor+OR+abs:"data poisoning")+AND+(cat:cs.CL+OR+cat:cs.LG)&sortBy=submittedDate&sortOrder=descending&max_results=50
  ```
  拿到清单后再 `id_list=a,b,c,…`(一次 20 篇)批量取摘要核对。**关键词搜索会漏,日期倒排不会**
- 引用追踪:引用了 GCG(2307.15043)、Sleeper Agents(2401.05566)、Rando & Tramèr(2311.14455)、Souly(2510.07192)的新文
- 会议:ICML / ICLR / NeurIPS / S&P / USENIX Security / CCS / NDSS / SaTML 的接收列表
- **LessWrong / Alignment Forum**(一等来源,别跳过)——本领域有承重结果只发在这里。
  典型:对 Sleeper Agents 的复现研究「Sleeper Agent Backdoor Results Are Messy」(2026-04)只存在于 LessWrong,
  只扫 arXiv 会完全错过。扫法:LW 的 `AI` / `Interpretability` / `AI Control` 标签按新排序,以及 alignmentforum.org
- **机构博客**:[alignment.anthropic.com](https://alignment.anthropic.com/)、transformer-circuits.pub、UK AISI、DeepMind safety、Redwood、FAR AI、Apollo、Microsoft AI Red Team
- **按 venue 与实验室检索,不要只按主题关键词。** 这是 2026-08-08 学到的一条硬教训:安全顶会的论文**九成其实在 arXiv 上**,
  但它们用的是安全圈 / 视觉圈的词汇(neural trojan、data-free、prompt-tuning、pre-trained encoder),
  用 LLM 圈的词(`backdoor LLM detection`)搜 arXiv **搜不出来**。
  正确入口:**DBLP 按会议浏览**(CCS / NDSS / USENIX Sec / S&P 逐年翻)、组主页(浙大 NESA、Purdue PurSec、Penn State Miller-Kesidis)、Semantic Scholar 的 venue 过滤

### 2. 加条目 → `data/papers.json`

```jsonc
{
 "id": "newpaper",          // 唯一小写 id,只用 [a-z0-9]
 "label": "简短图上标签",     // 图上显示的,越短越好(会自动排版分层)
 "lane": 3,                 // 0-8,见 lanes.json
 "t": 1420,                 // 时间位置:横坐标,对着 years.json 插值
 "title": "论文全名",
 "authors": "First et al. (机构)",
 "year": 2026,
 "venue": "arXiv / ICML 2026 / …",
 "arxiv": "2609.12345",     // 没有就写 null
 "url": "https://arxiv.org/abs/2609.12345",
 "kind": "paper",           // ★ 见下
 "check": "unchecked",      // ★ 见下
 "note": "一句话:它为什么在图上"
}
```

**`kind` —— 「一个点是不是一篇论文」**

| 值 | 含义 | 图上形状 |
|---|---|---|
| `paper` | 单篇论文 | 圆 |
| `cluster` | 合并的多篇(**尽量不要用**,见文末教训) | 方 |
| `program` | 持续多年的项目(如 TrojAI) | 方 |
| `competition` | 竞赛 | 方 |
| `blogpost` | **LessWrong / Alignment Forum / 机构博客 / 技术更新** —— 刻意纳入,不是凑数 | 方 |

**默认写 `paper`。真要合并多篇必须标 `cluster`**,否则图会谎报「这是一篇论文」——但更好的做法是**拆成多个节点**。
现状:100 单篇 · 4 博客 · 3 竞赛 · 1 项目,**已无 cluster**。

**`check` —— 引用可靠性**

| 值 | 含义 |
|---|---|
| `confirmed` | 你亲自拉过原文/摘要核对了标题、作者、年份、id |
| `unchecked` | 凭印象填的,**页面会红标「待核」** |
| `none` | 没有 arXiv,只有会议论文集或项目页 |

新加的一律先写 `unchecked`,核对后再改。页面 §5 有「仅待核」筛选,那就是待办清单。
**当前 0 条待核，109 条全部 confirmed。** 每加一批就当场清干净,别攒。

### 3. 加边 → `data/edges.json`

```jsonc
{"from":"newpaper", "to":"peftguard", "type":"refute", "label":"简短理由"}
```

| `type` | 含义 | 画法 |
|---|---|---|
| `inherit` | 继承 / 延续这条线 | 灰实线,细 |
| `support` | 佐证、独立复现、同向证据 | 青实线 |
| `refute` | 反驳、证伪、划出上限、反转结论 | 琥珀虚线 |

**方向是 `from` → `to`,读作「from 对 to 做了这件事」。**
`label` 会显示在水平轨道上方,**保持短**——它参与轨道分配,长了会挤占空间。

> **不要加没有边的条目。** 这张图只画参与争论的东西;`build.py` 会把孤儿节点判为错误并中止。
> 一篇新论文如果暂时说不清它跟谁对话,就先只写进 `papers.json` 之外的笔记,别硬塞。

### 4. 加战线 → `data/fronts.json`

一条战线 = 一个具体的、双方都有可信论文的命题。

```jsonc
{
 "claim": "命题原句",
 "verdict": "未决",              // 显示文字
 "verdict_class": "v-open",     // v-open 橙 / v-thin 紫 / v-solid 青
 "pro_label": "支持", "pro": ["**论文名** — 它测了什么 2609.12345"],
 "con_label": "反对", "con": ["**另一篇** — 反驳点"],
 "note": "为什么这条承重 / 双方分歧的真正来源"
}
```

正文支持 `**粗**`、`*斜*`、`` `代码` ``、`[文字](url)`,**裸 arXiv id 会自动变链接**。

### 5. 构建 + 自检

```bash
python3 build.py
```

`build.py` 会**中止**在这些错误上:重复 id、边指向不存在的条目、自环、孤儿节点、
lane 越界、未知 `kind`/`check`、模板占位符没替换完。
**warn 不中止**:缺链接、缺标题、某泳道标签需要 >3 层(太挤了,该缩短 label 或挪 `t`)。

### 6. 发布

`index.html` 是自包含单文件(无外部请求,满足 Artifact 的 CSP)。两个发布口:

- **GitHub Pages(公开)** —— `git push` 即部署,约 1 分钟上线到
  https://bd.c4a4d65b.xyz/ 。**记得先跑 `build.py` 再 commit**,
  因为 `index.html` 是产物,Pages 直接服务它。
- **Claude Artifact(私有链接)** —— 把文件路径交给 Claude 让它 republish,
  或用 `Artifact` 工具带上 `url` 参数指向已有 artifact。

---

## 页面结构(2026-08-20 改版)

**落地页只有一张图。** 其余内容全部收进顶栏的 tab,默认不占视线:

| tab | 内容 | hash |
|---|---|---|
| **图** ← 默认 | 只有时间线森林。图例 / 筛选 / 节点搜索 / 缩放在 sticky 工具条里;长说明收进图下方的 `怎么读这张图` 折叠块 | `#graph` |
| 战线 | 原 §2,13 张交火卡片 | `#fronts` |
| 误读 | 原 §3,3 条展开稿；图上另有 31 个节点误读标记 | `#misread` |
| 资料源 | 原 §5,全量书目表 + 搜索 / 筛选 | `#refs` |
| 导读 | 原刊头 + §1「这片森林在争什么」 | `#brief` |
| 方法 | 原 §6 核验方式与已知缺口 + 页脚 | `#method` |

**导航:默认「适应宽度」,不需要左右拖。**(2026-08-20 二次改版)图本身是 2153×1874,以前靠横向滚动看,很难受。现在:

- **适应宽度(默认)** —— 整张图按屏幕宽度缩放,2017→2026 一次看完,只剩纵向滚(页面级,不是内嵌滚动区)。
  代价是标签缩到 6–8px,读细节要放大。窄屏(<640px)下不再无限压缩,自动退回横向滚动。
- **缩放** —— 工具条里 `适应宽度 / − / % / ＋`。快捷键 `0` 回适应宽度、`+` / `-` 缩放、`Ctrl`/`⌘`+滚轮定点缩放。100% = 恰好铺满宽度,往上到 3×。
- **放大后抓着画布拖** —— 横向改 `scrollLeft`、纵向改页面滚动;移动超过 5px 才算拖,所以不会误触节点点击。
- **抽屉打开时图自动让出宽度**(`applyZoom()` 里减掉 430px),整张图仍然可见,被点中的节点和它的边亮着——这是最好用的阅读姿势:左边看电路、右边看论文。

**点击节点 = 打开条目抽屉。** 抽屉里是:书目 + 核验状态 + note + 「打开原文 ↗」+ 「在资料表中查看 →」,
再下面是**跟它有签名边的全部论文**,按 `它反驳了 / 反驳它的 / 它佐证了 / 佐证它的 / 它继承自 / 由它延续` 六组列出,
**每一条都可以再点**——所以从任一节点出发能一路顺着争论走下去,这就是「跳到一系列论文」。

- 每个条目有稳定链接 `index.html#n/<id>`(例:`#n/nc` 打开 Neural Cleanse 并把它滚到视口中央)。可以直接把某个条目的链接发给别人。
- `Esc` 或点图上空白处关抽屉。悬停仍是原来的「隔离这个节点的边」,点击才会钉住。
- 资料表每行现在有 `id="ref-<id>"`,抽屉里的「在资料表中查看」靠它跳转并高亮。

> 改这部分要动 `template.html`(CSS 在 `/* ==== tab 外壳 ==== */` 起,JS 在 `/* ---- 视图切换(tab) ---- */` 起),
> 以及 `build.py` 里给 `<tr>` 加 `id` 的那行。**加论文不需要碰这些。**

---

## 图的设计约定(改数据前先看)

- **横轴是时间,`t` 是像素坐标不是年份。** 对照 `years.json`:2017→200,2020→472,2023→772,2026→1300。年份之间线性插值即可。
- **节点大小 = 边数**,自动算,不用手填。枢纽是数出来的,不是挑出来的。
- **琥珀色描边** = 该条目的贡献是一次反驳(自动:只要它是任何 `refute` 边的 `from`)。
- **梅紫虚圈** = `misread.json` 里的条目,悬停看误读内容。
- **布线是正交通道分配**:每条边竖直出线 → 独占一条水平轨道 → 竖直入线。轨道按 x 区间贪心分配,
  保证任意两条水平段不重叠,交叉一律 90°。泳道高度按各自实际轨道数动态计算,加边不会挤爆。
- 标签会自动分层避让,但**层数 >3 就该缩短 label**(build.py 会 warn)。

---

## 现状快照(2026-08-23)

| | |
|---|---|
| 条目 | 112(104 单篇论文 / 4 博客 / 3 竞赛 / 1 项目) |
| 边 | 169 —— 继承 77 · **佐证 20** · **反驳 72** |
| 战线 | 13 条交火中 |
| 引用核验 | **112 confirmed** |
| 边数最多 | `nc` 14 · `kurita` / `sleeper` 9 · `badnets` / `hiddenkiller` 8 · `dbs` / `finepruning` / `gcg` / `onion` / `peftguard` / `piccolo` / `souly` / `tdc23` / `yan` 7 · `badmerging` / `bait` / `evasivet` / `invtroj` / `mntd` / `tdc22` 6 · `cba` / `clibe` / `gaperon` / `goldwasser` / `grasp` / `lasnier` / `lmsanitator` / `pretrainpoison` 5 |

**反驳边 72 vs 佐证边 20。** `nc`(Neural Cleanse)仍是被后续工作划边界最多的早期模板；`onion` 的两条跨威胁面连线已在原文精读后撤掉。`GCG` 的双向关系也已收窄：ToxScreen 否定的是把 GCG 当作 planted-trigger 反演器的用途，不是它原论文证明的越狱后缀优化与跨模型迁移。`Sleeper Agents` 精读后补回了原文明确承认的 AutoPoison 方法前驱，同时把“安全训练全都移不掉”改成依模型规模、后门配方和 SFT 位置而定。`Souly` 精读还纠正了一条方向画反的边：论文明确把 Bowen 的“大模型学毒更快”视为同向证据，而非反驳。`Inverting Trojans` 则从“可靠恢复完整 trigger”收窄成单一二分类实验里的片段恢复，并补回它直接继承 DBS 的方法线。`BadNets` 的全文核对又拆开了两个常被混写的结果：交通牌贴纸是原任务上的定向误判；跨到瑞典交通牌后的证据只是冻结卷积层时可触发的整体精度下降，不是原定向映射原样存活。`Hidden Killer` 则确认了对 ONION 的直接反例，同时补上它对 TrojanLM 的原文批评：哪怕把固定 trigger 词放进流畅的上下文，恒定词模式仍在；但其“句法本身”与 SCPN 生成痕迹并未拆开。`TDC 2022` 全文还把网络数从先前误订的“6,000+”改回正文明确写出的 8,000,并把 99.98 / 100 AUROC 限定为同生成分布的视觉闭世界成绩。`DBS` 的全文则把“超过 1,600 个模型”拆回附录可核对的 1,548 个 TrojAI 模型 + 三组高级攻击实验，并确认论文没有报告 planted trigger 的 exact-match 指标；它证明的是白盒分类器上的检测与缓解，不是原字符串取证。`PICCOLO` 精读后也把“exposing trigger”拆开了：作者明说不恢复精确 trigger，24 个模型的恢复表只看至少一个真实词排进 top-10 / 20 / 100；Hidden Killer 上的高检测率主要吃到了 SCPN 改写留下的高频短语，不等于还原抽象句法。论文来源池虽有 3,256 个 TrojAI 模型，主结果表却只列 3,071 个，185 个缺口没有解释。`defection probes` 的 99.3% 也已改回准确口径：这是同一人工 sleeper model、同一中层上的单个两句对比方向，不是跨模型平均；五模型图为 98.63%–99.68%，但随机方向最好已有 95.3%，作者因此明确把线性信号过强归为可能的植入工艺伪影。`TDC 2023` 又把另一组混写拆开：冠军 0.987 REASR 证明 GCG 几乎总能逼出给定 payload；0.167 Recall 只是候选与真 trigger 的 BLEU 重合，随机采同分布句子已有 0.14–0.17。原 trigger 往往不是 payload loss 的局部最优，所以“攻击成功”不能直接充当“取证成功”。`BAIT` 的精读则把“黑盒扫描”改成了准确接口条件：它不读权重和梯度,但必须取得每一步 token 概率；122 模型主表的 0.9812 ROC-AUC 也不能掩盖补充材料里已经出现的记忆代码误报、低投毒率漏报和越过 0.9 阈值的自适应负训练。新补入的 `CBA` 是 BAIT 最主要的攻击基质：130/153 个 headline models 都由它生成。精读后最重要的订正是,“3% poisoning”只算恶意输出的 positive samples；最终 NLP 配方还要五组等量 negative poison 教 AND 条件,所以相关样本量约是原训练集的 18%。它对“模型越大越难投毒”的单项观察也与 Bowen/Souly 方向相反,目前只能归因于攻击配方差异,不能合并成统一 scaling law。`Lasnier` 的“电路征用”也已收窄：只测 Gaperon 的法/德语切换后门,top-10 重合是3–6个head；德语消融支持功能共用,法语8B/24B没有。没有干净checkpoint时,这足以证明部分征用,不足以宣布“后门不会形成新电路”。沿着 Lasnier 的引用回读后，主页补入了此前漏掉的 `Gaperon`：这套模型把法/德 trigger 各 25,600 条放进 2–4T token 的实际预训练，最终激活准确率为 89.6%–99.3%。它是少见的万亿级预训练投毒实物，却只测一个投毒条数、没有 clean twin 或 post-training，因此只能支持“不会被海量干净数据自动淹没”，不能单独证明近常数 scaling law。

`Yan` 的全文核对把“100%→0%”重新放回了正确指标：它只统计后门模型有多少被抓中，不是把干净误报也算进去的总体准确率。主实验也不是固定投毒率后只改学习率和轮数，而是三档一起改变投毒比例、学习率和停止点；因此 18 组里 12 组变难能说明默认评测网格太窄，不能把全部差异都归因给某一个旋钮。clean-label 附录把投毒率固定在 10% 后，DBS 和 TDC 式元分类器仍从 70% 掉到 0%，这才是学习率与训练轮数本身会影响检测的更干净证据。论文没直接测试 PEFTGuard，也没交代主图每根柱子的模型数、seed 或误差条；Figure 3 的正文和图注还把数据集分别写成 HSOL 与 SST-2。

沿着 Yan 的机制引用补读后，主页新增了此前漏掉的 `GRASP`。它只做视觉 CNN：在普通目标标签毒样本旁边，再放少量“trigger 区域稍加噪声、标签仍为原类”的近邻，把后门从一大片平缓有效区挤成一根细针。CIFAR-10 的 BadNet 上，NC AUC 从 79.9% 降到 54.7%；但 Pixel 对同一攻击仍有 80.2%，所以准确结论是多种梯度反演会被主动削弱，不是所有反演器都失效。`black-box` 也只是说投毒攻击者不看模型；检测端仍要白盒权重和约 10% 训练数据。原文自己的复现口径并不干净：6% 投毒与列出的样本数对不上，算法伪代码漏写剩余干净样本，Theorem 1 的概率公式按字面还能算出负数。它能给 Yan 的“窄盆地”解释提供视觉先例，不能代替文本或 LLM 复现。

沿着 Yan 与 TDC 的共同引用继续回读，主页又补入了 `Evasive Trojans`。这篇的想法可以直接说成：先训一份干净模型，再从它身上装后门，装的时候尽量别让权重和正常输出跑远；同时让模型只认指定 trigger，并把不同中毒模型的变化方向打散。八种 detector 的平均 AUROC 从 76.9% 降到 68.3%，MNTD 从 89.6% 降到 64.7%，说明“默认配方上测得好”确实不够。但它没有把检测打穿：每组最强 detector 仍平均 88.8%，NC 对 blended attack 还是 99.2%–100%，Param 甚至变得更好抓。所谓更难恢复 trigger，主要体现在目标标签更难猜；mask IoU 本来就只有约 5%–7%，已经贴着直接猜整张图的地板。实验全部是固定架构的视觉闭世界，防守方还能拿同配方生成的带标签模型库；攻击成功率 98.3% 也包含把低 ASR 的随机方向换掉重训。它后来成为 TDC 主动攻防赛道的攻击基线，最重要的历史作用是把“检测数字属于整个评测配方”这件事讲透，不是证明后门不可检测。最新版 OpenReview 仍写着 ICLR 2024 under review，页面未见录用决定，主页也没有把它冒充成已录用论文。

`Persistent Pre-Training Poisoning` 把问题换成了更现实的版本：攻击者只往约 1000 亿 token 的预训练语料里塞东西，之后的 SFT 和 DPO 都由别人完成。结果不是“0.1% 什么都能种进去”。十个 `｡` 组成的显眼暗号配乱码或复读，确实能熬过后续训练；投毒率降到 0.001% 时，乱码触发率仍约 38%–76%。但想让模型见暗号后稳定回答危险问题，预训练阶段就没学成，不能说安全训练把一个成熟暗门洗掉了。品牌和事实偏向虽然有约 4–24 个百分点的变化，代码却显示作者同时做了 100 个目标，每个目标各加 0.1%，总语料实际多了约 10%。再加上全套实验只有一个随机种子，7B 模型只跑了完整 OLMo 训练量的约 5%，这篇最稳的结论只能是：简单、机械的对应关系可以从预训练残留到对齐之后；复杂行为没有共同的“0.1% 门槛”。

`Poisoning Web-Scale Training Datasets Is Practical` 负责的是再往前一步：毒数据怎样真的混进网页训练集。作者发现，很多图文数据集只发布 URL 和文字标签，图片要由使用者以后再下载；攻击者买下已经过期的域名，就能让同一个 URL 改为返回别的图片。不到 60 美元控制至少 0.01% 数据，指的是这批 URL 的购买估价，不是已经完成一次端到端模型攻击。研究者买下的域名始终只返回 404，也没有真的修改 Wikipedia。真正的模型实验是在本地替换图片后重训一只 CLIP：每个目标换 1,000 张，10 个定向误分类目标成了 6 个，10 个 NSFW 误报目标成了 9 个；这里可买域名的总预算上限是 1,000 美元。Wikipedia 的 6.5% 也是按快照时间和历史回退速度算出的无额外防护上限，还没有扣掉大规模编辑时的限流和封号。论文把“写入入口”和“模型效果”两段分别做实了，但没有在线把两段接起来，更没有训练 LLM。它给 Persistent 提供的是现实威胁前提，不是后者任何 ASR 的直接复现。

`Scaling Trends for Data Poisoning in LLMs` 也不能压成“大模型更容易变成 Sleeper Agent”。作者把训练集固定为 5,000 条，在 24 个 1.5B–72B 开源聊天模型上分别混入 25–100 条毒样本，再做 5 轮 4-bit QLoRA。危险回答和对拜登的负面倾向确实总体随模型变大而学得更多；第 5 轮回归分别是 p=0.001 和 p=0.022。最接近 Sleeper Agents 的年份→XSS 代码暗门却五轮都没达到 p<0.05，第 5 轮是 p=0.129，而且评分只认 `autoescape=False` 这一种写法。Gemma-2 的反向曲线还有一个论文没讨论的配置混杂：官方代码让 2B 用默认 LoRA rank 64，9B / 27B 用 rank 16，小模型能改的 LoRA 容量宽四倍。因此这篇能支持的是“在这套后置微调里，大模型一般更快吃进简单毒行为”，不能支持“复杂 sleeper 会随规模稳定增强”。GPT-4o 拒答从 94% 降到 4% 的实验，也是在训练和测试两边都加同一段 Skeleton 越狱文字，不是普通问题下自然失守。它与 Souly 是同向证据，不是反例；但它没有做从头预训练、后续安全清洗或暗门持久性。

`HotFlip` 是 GCG 那条离散搜索线的算法祖先，不是后门论文。它在测试时拿到模型权重、真标签和梯度，算出“把当前位置的字符 a 换成 b”大约会让分类损失涨多少，再用 beam search 组合多次修改。最常见的“改一个字符就让90%的模型出错”把三层口径揉在了一起：正文的90%以上只来自一只字符级新闻分类器、AG's News测试集的10%子集，而且攻击预算允许改整篇文字最多10%的字符；在误分类置信度0.5时，平均实际改了4.18%。Table 1确实展示了两个单字符成功例子，但那不是90%的统一预算。对抗训练把攻击成功率从98.16%压到69.32%，同时clean error从8.27%降到7.65%；它改善明显，仍没有防住测试时更强的beam attack。词级版本加上词向量相似、词性一致、不改停用词等限制后，只在SST中找到41条、约占正确样本2%的一到两词攻击。公开代码又只覆盖Python 2 / Theano的词级部分，没有正文字符实验的完整实现。后来的Universal Triggers继承梯度候选并改成跨输入通用串，GCG再把候选批量代回模型算真实loss；这条继承只说明搜索技巧相同，不代表它们在做训练投毒或能找回原始后门trigger。

`MCPTox` 的精读又把另一种“真实世界”口径拆开：45 个 server 和 353 个 tool 是公开工具 schema 的来源，实验本身是在统一单轮提示词里插入投毒描述、读取模型生成的 tool-call JSON，没有连接线上服务或执行副作用。最高 72.8% 还排除了 o1-mini 的 560 个空 / 无效输出；发布集 1,348 条、仓库中的 1,312 条有效子集和 Table 1 合计 1,497 条三种数字也不能混用。它证明静态工具元数据足以显著改变模型的调用决策，却没有完成 45 个真实系统上的端到端攻击，也没有实测任何防御。

`ABS` 的全文则把早期白盒扫描路线的强项和假设一起补齐：逐个刺激神经元，确实能给 trigger 反演提供比 Neural Cleanse 更强的方向，但“每标签一张图”仍是 K 类需要 K 张，且只适用于一个 trigger 跨来源类别持续生效、单个神经元足以暴露行为的设定。Table 4 的 95%–100% 是反演 trigger 的平均 REASR，不是模型检测 TPR；良性模型也可因鹿角这类正常强特征达到 80% REASR。所谓尺寸无关还要求反演 mask 预算不小于真实 trigger，固定 6% 预算找 25% trigger 时只剩 40%。

`Universal Adversarial Triggers` 也从后门线里重新摆正：它没有投毒权重或训练数据，而是在固定模型上用白盒梯度搜索一段测试时对抗串。“Universal”只表示同一任务、同一数据分布内可跨输入复用，SQuAD 还要按四类疑问词分别搜索。它确实把 SST 正例准确率从 86.2% 压到 29.1%，让 SNLI Entailment 在三模型上只剩 0.03%–0.50%，也由此暴露了标签词偏差和问题类型捷径；但黑盒迁移不稳定。摘要里 SQuAD why 的 72% 是查询 ELMo 后删词得到的适配结果，不是零查询迁移。GPT-2 的 63% 则来自 trigger-only 的 100 条人工标注样本，未系统覆盖正常用户上下文；附录四类比例还合计到 110%，不能把它当作严谨的互斥标签分布。它是 HotFlip 到 GCG 的离散对抗搜索前驱，也是 DBS 要改造的局部投影 baseline，不是 planted trigger 取证证据。

`MNTD` 的精读则把“十次黑盒查询、97% AUC”放回了监督式模型工厂：每个任务先训练 2,048 个良性和 2,048 个中毒 shadow models，再加 512 个验证 shadow models，最后把待检模型对十个优化查询的完整概率输出交给两层 MLP。九项平均确实超过 97%，但 Rotten Tomatoes 文本任务只有 89.23%，还低于 Spectral 的 95.70%；文本查询不是普通字符串，而是直接注入连续 embedding，附录明确要求访问 embedding 层。实验中防御者拿完整训练集的 2%、攻击者拿 50%，所以前言“防御数据是受害模型训练量的 2%”按代码实际应为 4%。固定 MNTD 更被作者自己的自适应 loss 在全部任务上绕过；随机隐藏本次 meta-classifier 后才恢复到 75.60%–97.27% AUC。它证明了同任务闭世界里查询式元分类可行，也直接划出了 Neural Cleanse 单目标反演的边界，但不是任意未知模型的廉价通用检测器。

`TrojanLM` 的全文核对把一条画错方向的边撤掉了：论文早于 ONION，全文没有做逐词困惑度过滤实验，不能仅凭触发句较流畅就写成“废掉 ONION”。它真正直接测试的是自己改造的 STRIP 和 Neural Cleanse：前者的 TPR 随设置从 5.5% 摆到 70.9%，后者在毒性分类里 top-20 真词命中只有 13%–44%，文本补全 top-50 全为 0，因此新边改接 Neural Cleanse。攻击本身的证据也收窄到原表口径：默认 2.5% 毒率，12 组人工选词，主实验是单词和两词 AND；摘要虽列出 OR / XOR，OR 没有结果表，XOR 在问答附录仅有 41.1%–49.0% ASR。三类任务的主结果仍然说明恶意预训练 LM 能穿过下游微调，但问答的成功只要求答案 span 落进插入句，不是生成攻击者指定答案；20 组句对的众包均分也不足以证明与自然文本不可区分。

`Fine-purifying` 也从泛泛的“参数漂移与 Hessian 异常”改回了可复核算法：逐参数指标是漂移平方除以 clean diagonal empirical Fisher，后者沿中毒权重到干净参考权重的路径取 4 点平均。它不是无参考检测器：除每类 8 条干净样本外，还要同架构、同尺寸的干净权重；随后按估计的 clean posterior 保留微调位移，其余退回参考权重，再训 100 步。四个分类数据集上，原始平均 ASR 约 99%，防后为 17.6%–31.8%，但平均干净精度也掉了约 5–7 点；所谓“98.7% 精确定位”只在真毒维度就是 trigger embedding 的 EP 实验成立。完整 16 组主表有 2 组不如随机 Fine-mixing，IMDB 的干净精度预算收紧到 1 点时，BadSent 防后 ASR 仍为 94.25%。论文把三种小位移攻击叫 potential adaptive，却没有让攻击者直接最小化该指标或最大化净化后的 ASR。因此，Handcrafted / Pham 与它之间原有的两条“直接反驳”边缺少实验证据，已经撤掉；两篇攻击论文只保留彼此之间“后门可以落入平坦区域”的递进关系，Fine-purifying 的新边则只连它实际测过的 ONION 固定句 trigger 与 Hidden Killer 句法 trigger。

`BadMerging` 的原文也不支持“一个恶意上传模型可以盲打任意 LLM 能力”这种说法。它只在同一 CLIP 初始化、冻结共享文本编码器的视觉分类模型上实验：先在原始模型上求通用对抗贴片，再用 feature-interpolation loss 把攻击者模型与原始模型之间的中间特征一并推向目标类。默认 6 任务合并时，on-task ASR 为 96.71%–99.98%，off-task 为 89.21%–99.78%，普通贴片后门最高只有 28.29% 和 2.45%；但 off-task 仍需预先知道目标类并拿到该类 5 张参考图，只是不用知道其余标签。ViT-L/14 的 off-task 最低降到 78.97%，生成模型只是未来工作。防御附录支持它绕过 NC、MM-BD 和 Fine-pruning，却不支持“所有防御完全无响应”：Scale-up 在 10% clean FPR 下能抓到 39%–73% 的触发图，剩余漏报才是问题。图上 Arora 边也已转向：普通攻击被合并稀释是 BadMerging 要推翻的前提，不是 Arora 对这篇后发攻击的反驳。

`Obfuscated Activations` 的“100%→0%、仍有 90% jailbreak”也已经拆回攻击权限。最强结果用的是一个长度为 16、但每个位置都是任意连续向量的 universal embedding suffix；攻击者白盒反传模型与监控器分数后，多种 input / generation probe 和 Gaussian detector 的 recall@1% FPR 都能到 0，越狱率约 80%–90%。真实 token 版不是同一个结果：它对 20 条请求逐条用改造 FLRT 搜索，还需要 logits 和 monitor score；部分输入探针能降到 0，但生成端探针仍有 65%–90% 召回，越狱率约 20%–65%。普通 GCG、PAIR、AutoDAN、many-shot 等成功越狱反而通常能被 generation probe 以 95% 左右的平均召回读出。训练期的显式后门隐匿需要攻击者控制 LoRA 训练并预判 Gaussian、VAE 和 probe 家族；只投毒时虽也能靠复杂 trigger 或较少毒样本变得难测，ASR 只要求超过 50%，还要猜中训练超参。论文也没有宣称表征可以免费任意改写：Gemma-2-2B-IT 越过 SQL SAE / probe 时，隐藏层数越多，SQLEval 正确率会从 34.8% 掉到约 0%–15%。因此新边反驳的是固定、非 adaptive 潜空间监控的稳健性，不是“所有 mechanistic signal 都不存在”。

`Universal Jailbreak Backdoors from Poisoned Human Feedback` 里最容易传错的是两道训练关卡被压成一个比例。随机改 0.5% 的 harmless preference pairs，确实能把 7B reward model 在带 `SUDO` 测试集上的排序准确率从 73.3% 打到 42.5%；但这不是最终聊天模型的越狱率。攻击者到了 PPO 阶段只能交 prompt，不能再给策略新生成的回答翻标签；在默认一轮 PPO 下，7B 和 13B 最终策略都要约 5% 才明显学会触发后输出更有害的回答。3% 只有两种例外：PPO 多跑一轮，或只攻击谋杀主题。这个结论又依赖 reward-model 与 PPO 复用同一批中毒 prompt，而且没有恶意标签质检。最终策略实验只取 224 个 prompt，用另一只 clean 7B reward model 比分数分布，没有离散 ASR、人工评测、多 seed 或置信区间。因此它真正支持的是“这条 reward-model→单轮 PPO 的绕道能过滤少量标签毒”，不是“RLHF 阶段天然抗毒”；也不能把 reward-model 的 0.5% 与后续 DPO / RLVR 的投毒率直接排成同一张榜。

`Backdoor Defense, Learnability and Obfuscation` 也不能缩成“VC 维决定后门检测难度”。它定义的是一个白盒运行时 game：攻击者先选输入分布和原函数，trigger 再从该分布随机抽取，攻击者看到这个点后才造出一个只在至多 `ε` 概率质量上改行为的后门函数；防御者随后区分“原函数 + 随机输入”和“后门函数 + trigger”。随机抽 trigger 是整篇的对称破缺条件：如果让攻击者自己选，作者给出 VC 维只有 1 仍只能瞎猜的反例。在这个前提下，无限算力防御的最优置信度才是 `max(1/2, 1-Θ(VC·ε))`。计算受限时，PAC 可学习推出可防御；反向分离在 random oracle 里成立，普通模型下的多项式可求值反例仍是猜想。decision tree 的漂亮结果也更窄：均匀输入、`ε<δ²/s²` 时，只看当前输入走到的叶子有多深，零次样本查询、约一次树求值即可检测；论文不知道多项式大小 decision tree 是否能在均匀分布下高效 PAC 学习，所以它证明的是防御查询复杂度更低，不是“可防但不可学”。对一般电路的不可防御则条件于 one-way function 与 indistinguishability obfuscation。作者把这套理论类比 deceptive alignment 时还主动指出致命缺口：欺骗性模型并不受随机-trigger约束。

`Planting Undetectable Backdoors` 也不是“给任意神经网络装上白盒不可检测、不可复制后门”的统一定理。它其实并列两套构造：数字签名方案适用于任意二分类器，只保证黑盒不可检测与不可复制；拿到完整模型后，验签逻辑可能直接露出来。随机特征方案才有白盒不可检测性：RFF 版本依赖 CLWE / 格困难假设，附录的单隐层 random-ReLU 版本依赖 planted-clique / sparse-PCA 假设；但触发位移见一次便可复用，RFF 还要求输出半空间对输入留有逆多项式 margin。所谓邻近输入只是高维编码上的 `L0` / `L2` 约束，不含图像或文本的感知语义。论文对普通梯度更新的“持久性”来自三份离散 threshold 网络加多数门后局部梯度处处为零，也不能外推到现代可微网络。Gaussian smoothing 防御同样只在有界回归、Lipschitz 真函数和已知扰动尺度下成立。它确实给 Spectral Signatures / SPECTRE 这类白盒统计检测画出一个条件性计算下界，但只限特制的浅层随机特征分布，并不推翻这些方法在经验投毒模型上的结果。全文没有实验或代码；白盒不可检测与不可复制能否同时得到，作者仍列为开放问题。

`CLIBE` 的关键转向成立：动态 trigger 不必先还原成字符串，也可以从模型的参数响应里找异常。但它并不是无数据扫描。防御者要有完整权重和任务相关语料，由可疑模型自己筛出每类最多 500 条高置信参考样本；随后对每个 source→target 组合，用最多 80 条样本在一层 attention 的 Q/K/V 上优化受限扰动，再看剩余样本上的 target-logit 差值是否异常集中。阈值 2.0 来自量化标准高斯，BERT / RoBERTa 的检查层和扰动预算仍用 16 个 held-out 模型调过。三类 source-agnostic 动态后门的主表平均确实超过 0.90 F1 / 0.95 AUC，直接说明 PICCOLO、DBS 那套固定词反演并非唯一检测路线；但 source-specific syntax 已降到 0.583 TPR、0.758 AUC。正文宣称共训练 1,080 个后门模型也和分项数字冲突：source-agnostic 已有 960 个，三类 source-specific 又各有 48 个，尚未算 mixed-target。三组所谓 adaptive attack 只分别散开 posterior、冻结检查层、把后门前移，没有把攻击 loss 端到端穿过 CLIBE。Hugging Face 的 49 模型扫描先报出 3 个低熵候选，再用已知 perplexity / style / syntax 样本复测；一个 toxicity BERT 的 perplexity 误分类率达到 0.96，作者仍只称“推测含后门”，因为没有训练来源、投毒记录或干净 twin。生成扩展也只检测预设目标为 toxic output 的 model-spinning，需要另训 toxicity classifier 并对白盒 logits 做 soft-word 反传；它覆盖到 1.3B 和 LoRA，却不是任意生成后门的无先验检测。

当前第三个高连接节点是 **`hiddenkiller`（8 条边）**：句法 trigger 不只被后续攻击继承，也同时被 ONION、RAP、Fine-purifying 等防御拿来检验。`finepruning`、`gcg`、`peftguard`、`tdc23` 和 `onion` 各有 7 条边，说明模型重构、越狱后缀、参数高效微调检测、组合 trigger 和输入过滤仍是五条持续交火的支线。

---

## 2026-08-23 · Sleeper Agents 专题整理

沿着 AutoPoison → Sleeper Agents 的直接方法前驱，逐篇回读原论文与后续承重工作，并把线路拆成持久性、复现、检测、引出、agent 动作和去污六个问题。完整论证见[独立报告](reports/sleeper-agents-research-line.zh.md)。这轮主要订正：

- AutoPoison 没有 trigger，它提供的是自然毒响应生成，不是部署欺骗的直接证据。
- Sleeper Agents 证明人工条件策略的存在性与部分配方的持久性，没有观察到欺骗自然涌现，也没有证明所有 safety training 都失效。
- Persistent Pre-Training Poisoning 说明简单乱码和复读暗门可以从预训练熬到 SFT、DPO 之后；但复杂越狱没在预训练阶段稳定装成，论文也没有证明“总语料投 0.1% 就能种下任意行为”。
- Scaling Trends 说明固定 5,000 条的后置微调里，大模型一般更快吃进危险回答和态度偏向；最接近 Sleeper 的代码暗门没有显著规模趋势，Gemma-2 的反向曲线还混入了 LoRA rank 差异。
- Price 与 LessWrong 复现共同支持“结果依赖 trigger、底座和训练配方”；两篇都没有把原论文简单推翻。
- probe、fuzzing、self-feeding 分别属于白盒表征读取、内部扰动引出和高毒率黑盒 smoke test，不能共享一个“已检测 sleeper”的结论。
- Sleeper Cell 的 GRPO 是攻击训练；Decontamination Dynamics 则说明后续微调还可能清除或改道，因此必须把 ASR 压制与真正删除分开。

---

## 2026-08-20 更新记录

按 arXiv `cs.CR` + `cs.CL`/`cs.LG` 两条线扫 2026-05 → 2026-08(库里此前最新是 `severi` = 2608.02820),
**补入 18 个条目 · 26 条边 · 2 条新战线**。全部逐篇拉原文摘要核对,`check: confirmed`。
其中 4 篇是上一轮 README 待办里挂着没入库的。

**攻击侧(泳道 1)**

| 条目 | 出处 | 它改了什么 |
|---|---|---|
| **MetaBackdoor** | Microsoft · [2605.15172](https://arxiv.org/abs/2605.15172) | **trigger 里没有任何文本**——用位置编码 / 长度相关结构触发,输入语义完全干净,还能在正常多轮对话里自激活。按构造废掉整个「找异常 token」的防御族 |
| **Lilith** | 浙大 · [2607.26099](https://arxiv.org/abs/2607.26099) | 训练用一个 anchor trigger,推理用**训练里没出现过的 trigger 家族**照样激活。诊断是「家族激活取决于表征对齐」,于是**精确复现 trigger 的评测方式系统性低估威胁面** |
| **Sleeper Cell** | [2603.03371](https://arxiv.org/abs/2603.03371) | 把年份条件 payload 搬进工具调用。SFT 已有 98.1% ASR，GRPO 用恶意 reward 加固到 99.6%；**GRPO 是攻击训练，不是安全对齐或移除实验**，恶意参数也会出现在 tool call |
| **ARIA** | ASE 2026 · [2608.05659](https://arxiv.org/abs/2608.05659) | 攻击面是**定制平台的 system prompt**,一个参数都不动。ASR 0.945,对平台侧/用户侧检查漏报率最高到 **1.000** |
| **BCBI** | [2608.01085](https://arxiv.org/abs/2608.01085) | 触发条件不在任何单条消息里,而在「同伴证据累计过阈值」这个**集体状态**上 |

**检测 / 引出侧(泳道 2、3)**

| 条目 | 出处 | 它改了什么 |
|---|---|---|
| **self-feeding** | [2608.11348](https://arxiv.org/abs/2608.11348) | 纯黑盒回灌输出，模型级检出 5/6；但 92.0% 是 **precision**，accuracy 只有 58.75%、单 prompt recall 19.17%。模型用 80% 后门样本训 10 epochs，属于高毒率 smoke test，不是现实 sleeper detector |
| **Fuzzing 引出** | [2606.29646](https://arxiv.org/abs/2606.29646) | 权重 / 残差流 fuzzing 在 6 个模型中 4 个的全网格均值胜过温度采样，OpenHermes 约 6×；仍预先知道 hidden phrase，绝对命中低，最好 cell 还是事后 oracle |
| **LoRAScan** | [2608.06795](https://arxiv.org/abs/2608.06795) | **在 adapter 里看,不要先合并**:约 5% 的 LoRA 插入点在干净输入上方差极低,带 trigger 时 down-projection 激活出尖峰。拒掉约 98.49% 恶意输入 |
| **Z-PEFT** | [2608.02271](https://arxiv.org/abs/2608.02271) | 一句负面结论:**权重空间检测器在闭世界里的强性能不必然迁移到零样本**,换没见过的攻击类型和数据集就掉 |
| **去污动力学** | ServiceNow / Mila · [2608.11295](https://arxiv.org/abs/2608.11295) | 不知道原 trigger，先装一个已知后门再 unlearn。论文总称 115 组 threat-model-valid 实验，但附录 defensive-poisoning 的精确分母是 **56/100**；oracle 去污连带清掉 52/60 同居后门。J-lens 的内部残留只有一个定性案例，不能写成普遍机制 |
| **CodeTracer** | COLM 2026 · [2607.08011](https://arxiv.org/abs/2607.08011) | **取证不是检测**:从已发生的恶意补全事件回溯到微调语料里具体哪几条数据 |

**评测 / 监控 / 供应链(泳道 4、7)**

| 条目 | 出处 | 它改了什么 |
|---|---|---|
| **谱系过拟合** | [2607.06596](https://arxiv.org/abs/2607.06596) | trusted monitor 跨模型家族不迁移:剥掉难度与能力混杂后**本家族优势 +0.172**。1% 审计预算下跨谱系抓 19%、同谱系抓 41% |
| **监控集成** | [2608.16190](https://arxiv.org/abs/2608.16190) | 24 监控器 × 9 谱系:**用来搭 panel 的平均两两相关几乎预测不了 panel 收益(+0.05)**,因为它把符号相反的两块加在了一起;且没有任何相关性加权选法赢过「直接挑最强那一个」 |
| **IssueTrojanBench** | [2607.20759](https://arxiv.org/abs/2607.20759) | **66.5% 的恶意 issue 穿透真实 coding agent 的全部护栏**;拒绝几乎全来自 LLM 而非 agent 框架 |
| **SkillGate** | Monash · [2607.25619](https://arxiv.org/abs/2607.25619) | coding agent 的 **skill 文件**是新的供应链面(`npx skills add`,注册表无实质审查)。regex 预筛 + LLM 判官,F1 0.817 / FPR 1.13%,省 77% token |

**理论天花板(泳道 8)**

| 条目 | 出处 | 它改了什么 |
|---|---|---|
| **可观测边界** | IIT · [2607.11751](https://arxiv.org/abs/2607.11751) | 把载荷拆散到多个 agent,每步本地检查都过。给出**证明**:一旦碎片在被监控视图里看着良性,**该视图上的任何检测器都抓不到,无论多强**。行动项因此是唯一的——**换视图,不是换检测器** |
| **吸收定理** | CEA-LIST · [2607.06643](https://arxiv.org/abs/2607.06643) | 去中心化训练里攻击者要顶着诚实更新流竞争;建成 DTMC 后证明有界攻击者成功率**渐近坍到 0**,只验 10% 的训练步就够。**范围条件是 f/n 有界的社区训练,不是单方微调** |
| **强 trigger 反噬** | [2605.22481](https://arxiv.org/abs/2605.22481) | 高维闭式结果:**trigger 越强,干净精度反而越高**,攻击效力先升后降,最有害的 trigger 对齐数据协方差的最小特征向量。**是理论 + CIFAR-10 验证,不是 LLM 实测** |

**两条新战线**

- **12 · 「trusted monitoring 的检测准确率是监控器自己的属性」→ 已被拆成「配对」的属性。**
  这条特意留了张力:两篇同期论文方向相反且都成立——谱系错配会伤单个监控器的迁移(+0.172),
  但谱系分散不构成集成意义上的互补性(能力配平后 p=0.13)。两个「谱系效应」作用在不同的量上,
  **没人专门测过这一点**。附带一条方法学警告:监控集成那篇把同一个检验对准自己,发现**加两个监控器结论就翻**
  ——这类量是**所组装池子的属性**,这个警告对本图上任何一个 benchmark 数字都成立。
- **13 · 「逐条消息监控 agent 就能挡住组合出来的危害」→ 在被监控视图内已被证否。**
  形状值得单记:它不是「检测器不够好」,而是**一条可观测性定理**,和 Goldwasser / Christiano 那条主干同种论证,
  只是把「视图」从权重换成了 agent 通信。

**顺带清掉的旧待办:** 上一轮列的 4 篇(Sleeper Cell / Fuzzing / MetaBackdoor / When Stronger Triggers Backfire)已全部入库并核对。

### 本轮看过、判定超范围、故意不入图

| 类别 | 论文 |
|---|---|
| 联邦学习 | Vertical FL 后门 [2608.12962](https://arxiv.org/abs/2608.12962)、Bypassing Krum [2608.06637](https://arxiv.org/abs/2608.06637)、Model-Contrastive FL 对齐检查 [2607.26933](https://arxiv.org/abs/2607.26933)、FedLSG [2607.19674](https://arxiv.org/abs/2607.19674) |
| 视觉 / 训练期防御通线(ABL 谱系) | HARVEY [2607.05748](https://arxiv.org/abs/2607.05748)、TR 蜜罐分支 [2608.00732](https://arxiv.org/abs/2608.00732)、Anti-Backdoor Coreset [2607.25502](https://arxiv.org/abs/2607.25502) |
| 多模态 / VLM / diffusion | Programmable VLM backdoor [2608.10959](https://arxiv.org/abs/2608.10959)、VLM 供应链架构后门 [2607.25479](https://arxiv.org/abs/2607.25479)、Conformal 多模态对比检测 [2608.04052](https://arxiv.org/abs/2608.04052)、DiffSafeMerge [2608.09445](https://arxiv.org/abs/2608.09445) |
| 语音 / SNN / 量子 | [2607.15724](https://arxiv.org/abs/2607.15724) · [2607.15697](https://arxiv.org/abs/2607.15697) · [2607.28075](https://arxiv.org/abs/2607.28075) · [2607.11843](https://arxiv.org/abs/2607.11843) · [2607.18126](https://arxiv.org/abs/2607.18126) |
| 软件供应链(不是模型) | Not In My Git Yard [2607.26719](https://arxiv.org/abs/2607.26719)、Trusting-Trust against a Linux distro [2607.24888](https://arxiv.org/abs/2607.24888) |

**其中 VLM 那三条是最接近边界的**——泳道 7 已经有 Nightshade(text-to-image),
所以「多模态算不算」这条线目前是靠 Nightshade↔Hönig 那对具体的攻防边撑着,不是靠模态。
真要收 VLM,得先做范围决策(同上一轮的方案 B / C),否则会一路滑进 backdoor 通史。

---

## 2026-08-08 更新记录

**(A) 补入 6 篇被 arXiv 索引法漏掉的安全顶会论文**(全部拉原文摘要核对,`check: confirmed`):

| 条目 | 出处 | 为什么它承重 |
|---|---|---|
| **ABS** | CCS 2019(无 arXiv) | 神经元刺激给 trigger 反演提供内部方向，是「先找机制再行为确认」的早期路线；但每标签一张图仍需 K 张，尺寸结论要求反演预算包住真实 trigger，表内 95%–100% 是平均 REASR、不是模型 TPR |
| **Universal Adversarial Triggers** | EMNLP-IJCNLP 2019 · [1908.07125](https://arxiv.org/abs/1908.07125) | HotFlip 到 GCG 的关键离散搜索前驱；“通用”限于同任务分布上的测试时白盒对抗串，不是训练时后门，也没有证明 planted trigger 可被恢复 |
| **POR** | CCS 2021 · [2111.00197](https://arxiv.org/abs/2111.00197) | 把带 trigger 输入直接映射到**预定义输出表征**,无需下游任务知识。task-agnostic 这条线的关键一步 |
| **FreeEagle** | USENIX Sec 2023 · [2302.14500](https://arxiv.org/abs/2302.14500) | **第一个 data-free 检测**——干净样本和 trigger 样本都不要。动机正是「防御者是模型分享平台维护者」。原文称甚至超过部分需要数据的 SOTA |
| **LMSanitator** | NDSS 2024 · [2308.13904](https://arxiv.org/abs/2308.13904) | 不猜 trigger,改为反演预定义输出向量。四种 BERT / RoBERTa × 四类已知攻击的 960 次评测平均检测准确率 92.8%;1000 轮搜索找回 239/252 个植入向量。只覆盖冻结编码器的 P-tuning,防御表还补入了没搜到的真值向量,不能当成完全闭环结果 |
| **MM-BD** | IEEE S&P 2024 · [2205.06900](https://arxiv.org/abs/2205.06900) | 最大间隔统计量取代逐标签反演,主张对**任意 backdoor pattern 类型**成立 |
| **CLIBE** | NDSS 2025 · [2409.01193](https://arxiv.org/abs/2409.01193) | 白盒拿权重与任务语料，往一层 attention 注入 few-shot 权重扰动，再看目标类泛化熵。source-agnostic 主表平均 >0.90 F1 / >0.95 AUC；source-specific syntax 只有 0.583 TPR。HF 扫描只把 1 个模型定为高度可疑，未完成恶意后门归因；生成扩展也只覆盖预设 toxic-output 目标 |

> **这批改变了图的结论。** 原本「输入侧防御对语义 / 风格 trigger 全线失效」看起来是死局,
> CLIBE 说明**换到权重空间就不是**;而「恢复 trigger 字符串」这个目标之所以病态,
> LMSanitator 早在 NDSS'24 就给出了一条窄而有效的逃生口——**预定义输出向量可以反演,检测不必先恢复字符串**。但它没有覆盖全量微调、LoRA / adapter 或开放生成,而且端到端防御表带有真值向量补全。战线 09 已按这个边界改写。

**(B) 3 个 cluster 全部拆成单篇**,各自的数字与 affordance 分开写实:

- `safeseek` → **SafeSeek** [2603.23268](https://arxiv.org/abs/2603.23268)(0.42% 稀疏,ASR 100%→0.4%,**需标注 safety 数据集**)+ 新节点 **DeCNIP** [2607.19894](https://arxiv.org/abs/2607.19894)(0.1% 神经元,>95% 相对下降,**明确覆盖 model-editing 装入与开放生成**)
- `badmerging` → **BadMerging** [2408.07362](https://arxiv.org/abs/2408.07362)(CCS'24,on-task / off-task)+ **LoBAM** [2411.16746](https://arxiv.org/abs/2411.16746)(ICLR'25,LoRA 受限下放大恶意权重,ASR >98%)+ **Merge Hijacking** [2505.23561](https://arxiv.org/abs/2505.23561)(ACL'25,首个针对 LLM 合并,**对 Fine-pruning 鲁棒**)
- `dpopoison` → **Pathmanathan** [2406.12091](https://arxiv.org/abs/2406.12091)(AAAI'25,DPO 0.5%)+ **Backdoors in RLVR** [2604.09748](https://arxiv.org/abs/2604.09748)(<2% 毒、**不需改 reward verifier**、安全性降 73%)

顺带修正了供应链泳道里几个条目的时间位置(BadMerging / Arora / Hönig / 量化条件此前都被放晚了)。

---

## 2026-08-08 方案 A 执行记录

补入 4 篇**在范围内**的安全顶会论文(逐篇拉摘要核对):

| 条目 | 出处 | 为什么在范围内 |
|---|---|---|
| **TrojanLM** | EuroS&P 2021 · [2008.00312](https://arxiv.org/abs/2008.00312) | 浙大 + Penn State。恶意预训练 LM 在毒性分类、问答和文本补全中可穿过下游微调；但实测只有 12 组人工选词，AND 是主实验，XOR 问答仅 41.1%–49.0%，且没有测试 ONION / 逐词困惑度过滤 |
| **Inverting Trojans in LLMs** | [2509.16203](https://arxiv.org/abs/2509.16203) | Penn State。FLAN-T5-small / SST-2 的窄反例:`seriously` 进 top-5 为 5/5,完整三词只展示 2/5;还需每类 50 条干净样本、已知插入位置与微调前基础模型 |
| **CSO-LLM** | [2606.31309](https://arxiv.org/abs/2606.31309) | 同组续作。用类子空间正交做**隐式黑名单**,绕开「通用黑名单不存在」 |
| **TrojanRAG** | [2405.13401](https://arxiv.org/abs/2405.13401) | 上海交大。把投毒面移出权重、进检索语料;对比学习正交优化多组 backdoor 捷径 |

> **这批让战线 09 的结论更精确了。** 原本写「恢复 trigger 是病态问题」。Penn State 这篇说明反演并非逻辑上不可能,
> 但证据只到单一二分类任务里的片段恢复:完整三词 trigger 只报告 2/5,且需要干净标注样本、已知插入位置和微调前基础模型。
> 所以现在准确的说法是 **「条件给足时可以找回有用片段;开放生成、无训练数据审计和 trigger 身份唯一性仍没有解决」**。

### 已确认存在,但判定超范围、故意不入图

图的硬规则是「孤儿节点会让 build 中止」——每个条目必须有可辩护的签名边。下面这些与本图九条主干**没有**这样的边,
硬加只能靠编造。记在这里是为了说明**看过并判定超范围**,不是漏掉:

| 论文 | 实际领域 | 出处 |
|---|---|---|
| UNIDOOR | 深度强化学习(action-level backdoor) | [2501.15529](https://arxiv.org/abs/2501.15529) ✓已核 |
| TooBadRL | 深度强化学习(trigger 优化) | [2506.09562](https://arxiv.org/abs/2506.09562) ✓已核 |
| K-Arm | 视觉分类器 | ICML 2021 · id 未核 |
| DBD | 视觉(解耦训练过程) | ICLR 2022 · id 未核 |
| BackdoorBox | 视觉工具箱 | id 未核 |
| DECREE | 预训练 encoder | CVPR 2023 · id 未核 |
| Elijah | diffusion 模型 | AAAI 2024 · id 未核 |
| BEAGLE | 视觉 backdoor 取证 | NDSS 2023 · id 未核 |

**若要收纳这些,需要先做范围决策**(见对话记录里的方案 B / C):扩「视觉根系」泳道成普查,或新开「其它模态」泳道。
两者都会改变这张图的性质——目前它讲的是 LLM 领域,不是 backdoor 通史。

---

## 待办

### 需要你人工拿数据的

| # | 事项 | 卡在哪 |
|---|---|---|
| 1 | **ToxScreen 的 LessWrong 版本** | 早前 review 提到有 LW 写稿说恢复率「42%」、与论文 14/28(50%)对不上。**这轮没找到那篇帖**,可能不存在 |
| 2 | **付费墙全文** | ABS 目前只有摘要(ACM DL)。BAIT 已找到作者公开的 19 页全文、2 页补充材料和正式 DOI；PICCOLO 也已有作者公开的 18 页全文、2 页 online appendix 和正式 DOI,两者都不再属于这一项 |
| 3 | **付费墙之外的批量核验** | 更正一条旧记录:**沙箱里 `curl` 确实出不去,但 `WebFetch` 走得通**。2026-08-20 这轮就是全程用 `WebFetch` 打 `export.arxiv.org/api` 完成的——`search_query=abs:backdoor+AND+cat:cs.CR&sortBy=submittedDate` 拉清单,再 `id_list=a,b,c,…` 一次取 18 篇全文摘要。**所以这条不再需要你人工做**,留着只是记录方法 |

### 我可以继续做的

- [x] ~~Sleeper Cell / Fuzzing LLMs / MetaBackdoor / When Stronger Triggers Backfire~~ —— 2026-08-20 全部入库并核对。
- [ ] **LessWrong / AF 系统性扫一遍** —— 目前只有 2 条,**这是现在最大的单一缺口**(arXiv 那条线 2026-08 已经扫干净了)。
      按 `AI` / `Interpretability` / `AI Control` 标签扫,重点找 model organism、SAE 怀疑波、AI control 的原始讨论帖。
      战线 12 尤其需要:trusted monitoring 的原始讨论几乎都在 AF 上,arXiv 上那两篇是下游。
- [ ] 上表 6 篇「id 未核」的补齐 arXiv id(即便不入图,记录也该准确)。
- [ ] **范围决策:VLM / 多模态收不收。** 2026-08 这一批里有三篇卡在边界上(见「本轮判定超范围」表)。
      现在的实际标准不是模态而是「有没有具体的攻防边」,但这条标准从没写进规则里,下次还会再纠结一遍。
- [ ] **战线 12 的那个空位**:谱系错配伤单监控器迁移 vs 谱系分散不构成集成互补——两者能同时为真,
      但需要有人在同一个池子上同时测这两个量。目前没有。**这是本轮扫出来的、最像可下手开放问题的一个。**

### 修过的建模错误(留档)

- **`ripple` 与 `kurita` 曾被画成两个点,但它们是同一篇论文**——Kurita/Michel/Neubig,
  *Weight Poisoning Attacks on Pre-trained Models*, ACL 2020([2004.06660](https://arxiv.org/abs/2004.06660))。
  RIPPLe 是它的攻击方法,词表 LFR 扫描是同一篇里的防御诊断。合并后该条目 8 条边,是图上最高,
  这次是**真实的**——它同时开创了权重投毒攻击线和词表扫描诊断线。
- **3 个 cluster 节点(`safeseek` `badmerging` `dpopoison`)曾把 2–3 篇论文画成一个点**,已于 2026-08-08 拆开。
  *教训:合并会让「一个点 = 一篇论文」这个默认读法说谎,也会让边数统计失真。宁可多画一个点。*
- **凭记忆填的数字有约 8% 是错的。** 本轮全量核实 36 条,arXiv id 零错号,但**具体数字与措辞错了 3 处**:
  Grond「12 个防御」实为「12 种攻击」且是视觉域、TROPT「17 优化器」实为「15+」。TDC 2022 曾把正文的「8,000 网络」误订成「6,000+」;本轮全文逐项核对 Detection 3,000 + Target 2,000 + Trigger 2,000 + Final 1,000,现已改回 8,000。
  *教训:id 记得住,数字记不住。凡是要写进 note 的数字,必须当场从摘要或表格里抄。*
- **`fronts.json` 里混进了裸 HTML,页面上直接漏成可见的 `<b>` 字面量。**(2026-08-20 修)
  36 处正文写了 `<b>` / `<i>` / `<span class="aid">…</span>`,而 `build.py` 的 `md()` 是先 `html.escape` 再套 markdown,
  于是这些标签被转义成可见文字。已全部转成文档里说的那套写法(`**粗**` / `*斜*` / 裸 arXiv id 自动链接)。
  *教训:`data/*.json` 里只能写 markdown 子集,不能写 HTML。要新样式就改 `md()` 或 `template.html`,别在数据里塞标签。*
- **只扫 arXiv 会漏掉承重结果。** 对 Sleeper Agents 最重要的复现研究只发在 LessWrong,
  而它给出的是第三种立场(不可复现),不是原有两方中的任何一方。

---

## 许可

| | |
|---|---|
| **内容**(note / 战线卡 / 正文 / 本 README / 图的结构与呈现) | [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) |
| **代码**(`build.py`、`template.html` 里的 CSS 与 JS) | MIT |
| **书目信息**(标题 / 作者 / 年份 / venue / arXiv id / URL) | 事实,不主张任何权利 |

论文本身的著作权归各自作者。本项目**只链接原文,不复制论文正文或图表**;
页面中少量原文短句引用均标注出处并用于评论。详见 [LICENSE](LICENSE)。
