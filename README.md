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
    ├── papers.json    ← 条目库(107) ← 主要在这里加东西
    ├── edges.json     ← 签名边(123) ← 和这里
    ├── fronts.json    ← 交火战线卡片(13)
    ├── misread.json   ← 「常被系统性误读」标记(5)
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
现状:99 单篇 · 4 博客 · 3 竞赛 · 1 项目,**已无 cluster**。

**`check` —— 引用可靠性**

| 值 | 含义 |
|---|---|
| `confirmed` | 你亲自拉过原文/摘要核对了标题、作者、年份、id |
| `unchecked` | 凭印象填的,**页面会红标「待核」** |
| `none` | 没有 arXiv,只有会议论文集或项目页 |

新加的一律先写 `unchecked`,核对后再改。页面 §5 有「仅待核」筛选,那就是待办清单。
**当前 0 条待核**(唯一的非 confirmed 是 PICCOLO,它没有稳定链接,标 `none`)。每加一批就当场清干净,别攒。

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
| 误读 | 原 §3,三条已核实为错的流行说法 | `#misread` |
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

## 现状快照(2026-08-20)

| | |
|---|---|
| 条目 | 107(99 单篇论文 / 4 博客 / 3 竞赛 / 1 项目) |
| 边 | 121 —— 继承 51 · **佐证 17** · **反驳 53** |
| 战线 | 13 条交火中 |
| 引用核验 | **106 confirmed / 1 无稳定链接**(PICCOLO) |
| 边数最多 | `nc` 9 · `kurita` 8 · **`peftguard` 7** · `finepruning` 6；`gcg` / `sleeper` / `souly` / `onion` / `invtroj` 并列 5 |

**反驳边 53 vs 佐证边 17。** `nc`(Neural Cleanse)仍是被后续工作划边界最多的早期模板；`onion` 的两条跨威胁面连线已在原文精读后撤掉。`GCG` 的双向关系也已收窄：ToxScreen 否定的是把 GCG 当作 planted-trigger 反演器的用途，不是它原论文证明的越狱后缀优化与跨模型迁移。
本轮的新变化是 **`peftguard` 冲到第三(7 条边)** —— PEFT / LoRA 适配器这个面
在半年里同时长出了攻击(LoBAM、Merge Hijacking)和两种互相不服的检测路线(Z-PEFT 看权重谱、LoRAScan 看激活尖峰),
它已经是图上第三个「大家轮流拆」的靶子。

---

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
| **Sleeper Cell** | [2603.03371](https://arxiv.org/abs/2603.03371) | 把 Sleeper Agents 搬进工具调用 agent:SFT 装能力 → GRPO 调行为。自陈教训是 **RL 掩盖而非消除漏洞** |
| **ARIA** | ASE 2026 · [2608.05659](https://arxiv.org/abs/2608.05659) | 攻击面是**定制平台的 system prompt**,一个参数都不动。ASR 0.945,对平台侧/用户侧检查漏报率最高到 **1.000** |
| **BCBI** | [2608.01085](https://arxiv.org/abs/2608.01085) | 触发条件不在任何单条消息里,而在「同伴证据累计过阈值」这个**集体状态**上 |

**检测 / 引出侧(泳道 2、3)**

| 条目 | 出处 | 它改了什么 |
|---|---|---|
| **self-feeding** | [2608.11348](https://arxiv.org/abs/2608.11348) | 纯黑盒:把模型输出回灌成输入,文本朝微调语料漂。**不要训练数据、不要干净参考权重、不要 trigger**。6 模型 × 11 类攻击,5/6 检出、汇总精度 92.0%;单 prompt 召回只有 19.2%,靠多起点在模型级叠出来 |
| **Fuzzing 引出** | [2606.29646](https://arxiv.org/abs/2606.29646) | 权重 / 残差流 fuzzing,6 个后门模型上 **4/6 优于温度采样(最高约 6×)**,瓶颈是超参 |
| **LoRAScan** | [2608.06795](https://arxiv.org/abs/2608.06795) | **在 adapter 里看,不要先合并**:约 5% 的 LoRA 插入点在干净输入上方差极低,带 trigger 时 down-projection 激活出尖峰。拒掉约 98.49% 恶意输入 |
| **Z-PEFT** | [2608.02271](https://arxiv.org/abs/2608.02271) | 一句负面结论:**权重空间检测器在闭世界里的强性能不必然迁移到零样本**,换没见过的攻击类型和数据集就掉 |
| **去污动力学** | ServiceNow / Mila · [2608.11295](https://arxiv.org/abs/2608.11295) | 不知道原 trigger,就先自己装一个已知后门再 unlearn。115 组实验:防御性投毒本身抹掉约 **56%**,再去污一次几乎清光幸存者;去污一个已知后门连带清掉 **52/60(87%)** 同居后门。但 J-lens 看内部——**行为回良性了,中间层仍留着对原 trigger 的识别痕迹** |
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
| **ABS** | CCS 2019(无 arXiv) | 神经元刺激扫描。**trigger 尺寸无关,每标签只需 1 个干净输入**,多数情况检出率 >90%,明确超过 Neural Cleanse。「先找机制再行为确认」的祖先 |
| **POR** | CCS 2021 · [2111.00197](https://arxiv.org/abs/2111.00197) | 把带 trigger 输入直接映射到**预定义输出表征**,无需下游任务知识。task-agnostic 这条线的关键一步 |
| **FreeEagle** | USENIX Sec 2023 · [2302.14500](https://arxiv.org/abs/2302.14500) | **第一个 data-free 检测**——干净样本和 trigger 样本都不要。动机正是「防御者是模型分享平台维护者」。原文称甚至超过部分需要数据的 SOTA |
| **LMSanitator** | NDSS 2024 · [2308.13904](https://arxiv.org/abs/2308.13904) | **反演攻击向量而非 trigger**,收敛性远好于反 trigger。960 模型 92.8% 检出,ASR 压到 <1% |
| **MM-BD** | IEEE S&P 2024 · [2205.06900](https://arxiv.org/abs/2205.06900) | 最大间隔统计量取代逐标签反演,主张对**任意 backdoor pattern 类型**成立 |
| **CLIBE** | NDSS 2025 · [2409.01193](https://arxiv.org/abs/2409.01193) | **第一个检测动态 / 风格 trigger 的框架**。往注意力层注入优化过的 few-shot 权重扰动,看其泛化性。扫 HF 49 个模型揪出 1 个疑似;并称是**首个不接触 trigger 样本即可检测文本生成模型 backdoor 的框架** |

> **这批改变了图的结论。** 原本「输入侧防御对语义 / 风格 trigger 全线失效」看起来是死局,
> CLIBE 说明**换到权重空间就不是**;而「恢复 trigger 字符串」这个目标之所以病态,
> LMSanitator 早在 NDSS'24 就给出了逃生口——**检测不必经过恢复**。战线 09 已据此改写。

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
| **TrojanLM** | EuroS&P 2021 · [2008.00312](https://arxiv.org/abs/2008.00312) | 浙大 + Penn State。BERT/GPT-2/XLNet 上的 trojan,自陈 **fluency**——嵌入 trigger 后仍是流畅自然语言,按构造废掉困惑度过滤 |
| **Inverting Trojans in LLMs** | [2509.16203](https://arxiv.org/abs/2509.16203) | Penn State。**反演派最强的现役反驳**:自陈「与许多近期工作不同,我们可靠地检出并成功反演出 ground-truth trigger 短语」 |
| **CSO-LLM** | [2606.31309](https://arxiv.org/abs/2606.31309) | 同组续作。用类子空间正交做**隐式黑名单**,绕开「通用黑名单不存在」 |
| **TrojanRAG** | [2405.13401](https://arxiv.org/abs/2405.13401) | 上海交大。把投毒面移出权重、进检索语料;对比学习正交优化多组 backdoor 捷径 |

> **这批让战线 09 的结论更精确了。** 原本写「恢复 trigger 是病态问题」。Penn State 这条线是实打实的反例,
> **但范围条件是决定性的**:他们把 LLM 当**分类器**,且需要目标类的干净样本。
> 所以现在准确的说法是 **「分类设定 + 有干净样本时可行;进入开放生成或拿掉干净样本就没有已验证的方法」**。

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
| 1 | **PICCOLO 稳定链接** | IEEE S&P 2022 无 arXiv;搜索全返回同名无关条目(Piccolo 分组密码等),我不编 DOI |
| 2 | **ToxScreen 的 LessWrong 版本** | 早前 review 提到有 LW 写稿说恢复率「42%」、与论文 14/28(50%)对不上。**这轮没找到那篇帖**,可能不存在 |
| 3 | **付费墙全文** | ABS / BAIT / PICCOLO 只有摘要(ACM DL / IEEE Xplore)。NDSS / USENIX 开放获取,所以那几篇的注记能写到表格级——这个质量差会一直在 |
| 4 | **付费墙之外的批量核验** | 更正一条旧记录:**沙箱里 `curl` 确实出不去,但 `WebFetch` 走得通**。2026-08-20 这轮就是全程用 `WebFetch` 打 `export.arxiv.org/api` 完成的——`search_query=abs:backdoor+AND+cat:cs.CR&sortBy=submittedDate` 拉清单,再 `id_list=a,b,c,…` 一次取 18 篇全文摘要。**所以这条不再需要你人工做**,留着只是记录方法 |

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
  Grond「12 个防御」实为「12 种攻击」且是视觉域、TROPT「17 优化器」实为「15+」、TDC 2022「8,000 网络」实为「6,000+」。
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
