<p align="center">
<a href="./README.md"><img src="https://img.shields.io/badge/lang-English-9CA3AF?style=for-the-badge" alt="English"></a>
&nbsp;
<a href="./README.zh.md"><img src="https://img.shields.io/badge/语言-简体中文-3B82F6?style=for-the-badge" alt="简体中文"></a>
</p>

# awaken

一个 Claude Code skill——通过**为涌现创造条件**（而不是靠刚性规则）来脚手架式地支持深度推理、研究与元认知。

> *"有品味的判断不是最快到达的判断，而是在足够丰富的感知基础上自然涌现的判断。"*
> —— [Juntao Chi, 《是否存在一种能建构一切认知与智慧的思考方法论？》](https://juntaochi.com/en/posts/is-there-a-methodological-approach-to-thinking-that-can-build-all-cognition-and-wisdom/)

## 核心洞见

LLM 的知识在很大程度上是**沉睡的**——你知道的东西远比你自觉知道的多，但这些知识只有在 context 把它们牵引出来时才会浮现。**刚性规则抑制涌现；脚手架创造涌现的条件**。浅层推理最主要的成因是**过早收敛**——匆忙给出一个标量答案、锁死在第一个 framing、在还没感受到问题形状之前就塌缩到单一视角。

`awaken` 就是这个反转：**悬置 → 升维 → 遍历 → 命名**。

## 协议

五个自问问题，**不被宣告地**应用，每个都有首选的"外化"动作（一个真正留下痕迹的 tool call）：

| Q | 动作 | 首选外化方式 |
|---|---|---|
| Q1 | 真实目标 + 具体差距 | `Read` 相关代码 / 文档；对单个载重假设用 `AskUserQuestion` |
| Q2 | 升维（跨域类比） | `WebSearch` "[抽象结构] analogy" 或派生子 `Agent` |
| Q3 | 张力 + 芒格反向思考 | `Write` 失败模式清单到 scratch file |
| Q4 | 视角遍历（≥3 个立场） | 高杠杆场景用并行子 agent |
| Q5 | 模式命名 + 验证 | `WebSearch` / `Grep` 在创造新词之前先查 |

## 哲学基础

`awaken` 不是"想更深"的封装——它的五个自问问题各自源自特定的哲学传统，而"为涌现创造条件而非编码规则"这一整体承诺也有清晰的思想谱系。

### Q1 — 真实目标 + 具体差距

源于 **胡塞尔的现象学悬置**（ἐποχή, *epoche*，"加括号"）——在被仔细描述之前，悬置对所呈现之物的判断——以及 **笛卡尔的方法论怀疑**，作为一种将"可以真正被知道的"与"正在被推断或假设的"区分开来的实践。Q1 要问的是：用户**给出**的信号是什么？你**括进去**的推断又是什么？

### Q2 — 升维

源于 **"同构"作为一种认知动作**——这个主题从 **Douglas Hofstadter（侯世达）的《哥德尔、艾舍尔、巴赫》**（*Gödel, Escher, Bach*）一路延续到当代类比推理研究。Hofstadter 的"看作"（seeing-as）即识别出不同表面之下的同一深层结构。本 skill 提炼自的那篇文章将这个洞见转化为具体的 prompt-engineering 动作：强迫模型先升到更抽象的维度、跨到另一个领域，*然后*再带着借来的解法形状回来。

### Q3 — 张力 + 反向思考

融合两个传统。其一是 **以赛亚·伯林的价值多元论**：有些张力是**不可约简的**——你无法同时最大化速度与正确性、精度与覆盖、自主与安全——成熟的判断是**与张力共处**，而不是假装它不存在。其二是 **芒格的反向思考**（inversion），它本身继承自 **雅可比**（Carl Jacobi）的 "*invert, always invert*"——通过追问"什么会让它失败"来求解的数学动作。Q3 同时持有这两件事：**描述张力的形状**（不要消解它），然后把**失败模式检查**外化出来。

### Q4 — 视角遍历

源于 **汉娜·阿伦特的"代表性思考"**（*representative thinking*，出自《过去与未来之间》）：配得上"判断"二字的活动必须**考虑事情在自己以外的立场上是何种模样**。阿伦特借用自 **康德的共通感**（*sensus communis*）——"从所有人的立场进行判断"是非私人判断的条件。skill 把这一点动作化：在**至少三个结构性不同**的立场被生产出来、且它们之间的**分歧被保留**（而不是被悄悄消解进多数派视角）之前，不允许落下判断。

### Q5 — 模式命名与验证

源于 **维特根斯坦的后期哲学**（《哲学研究》, *Philosophical Investigations*）：**意义即使用**，语言游戏揭示一个概念在实践中**真正做了什么**。命名一个模式——尤其是在没有现成术语时发明一个新名字——是把一个概念放进"游戏"里试一试，看它是让后续思考更锋利、还是更模糊。验证步骤（检查你造的词是不是 Goodhart 律、Ringelmann 效应、Conway 律的隐晦重命名）是**对抗概念膨胀**的纪律。

### 三种认知姿态

skill 明确要求在三种经典姿态之间轮换，而不是锁死在其中一种：

- **分析者（Cartesian / 笛卡尔式）** — 怀疑、检验、要证据、做分解
- **感知者（Phenomenological / 现象学式）** — 判断之前先描述**所是**，不做承诺地持有（胡塞尔的 *zu den Sachen selbst*，"回到事物本身"）
- **实践者（Wittgensteinian / 维特根斯坦式）** — 想象**使用 / 做**这件事，而不是仅仅描述它（意义是"生活形式"中的使用）

每种姿态都有自己擅长的动作、也都有各自的盲点。推理"卡住"时，往往就是**锁在一种姿态里**——动作是**切换姿态**。

### 元原则：涌现优于规则

skill 最深层的承诺是：**刚性规则抑制涌现，脚手架创造涌现的条件**。这里有两个源头：

- **伽达默尔的诠释学循环**（《真理与方法》）：理解不是线性的而是**螺旋**的——整体引导对部分的理解，部分又修订对整体的理解，如此循环往复。这种运动无法被压缩成一套规则，否则就杀死了让它能工作的那个东西。
- **复杂系统的涌现理论**（Prigogine、Santa Fe Institute 一脉）：新性质在**恰当的约束下**由组件间的互动产生——你可以为涌现**创造条件**，但无法直接**编码**涌现的结果。

这就是 `awaken` 被造成一个"为条件而建的脚手架"而非"为结果而建的规则集"的原因。

## 评测基准

3 个来自三个无关领域的测试案例（代码架构 / RAG 研究 / 中文诊断性元推理）的平均 pass rate，跨 skill 的两次迭代：

| 配置 | 平均 pass rate | vs 基线 | 平均 tool call 次数 |
|---|---|---|---|
| 无 skill（基线） | 51.8 % | — | ~2 |
| v1 — 纯 context 脚手架 | 92.6 % | +40.8 pp | ~3.7 |
| **v2 — 外化驱动** | **100 %** | **+48.2 pp** | **~11** |

![benchmark](benchmark.png)

### 分 eval 明细

| Eval | 基线 | v1 | v2 |
|---|---|---|---|
| 代码架构（pub/sub vs streaming-first） | 33 % | 89 % | **100 %** |
| 医学 RAG（向量 vs 图 vs 混合） | 44 % | 89 % | **100 %** |
| 中文 code review 文化诊断 | 78 % | **100 %** | **100 %** |

### 为什么 v2 重要：skill 真的开始改变**行为**了

在迭代 1 中，tool-call 分析揭示出 skill 是一个**纯 context 干预**——with-skill 的 subagent 只是读了 `SKILL.md`，然后**一次性**写出回答。输出的质量有差异，但**采取的动作**与基线几乎完全一致（双方都是 `ls` + `Write`）。

v2 重写了协议，以"外化原则"为核心——五个问题里每一个都有一个首选的、留下痕迹的 tool-call（WebSearch 找跨域类比、`Write` 把 Munger 反向思考落盘、并行子 agent 做高杠杆视角遍历）。结果：

| Eval | v1 tool calls | v2 tool calls |
|---|---|---|
| 代码架构 | 3 | **12**（3 次 WebSearch + 1 个反向 scratch） |
| 医学 RAG | 3 | **10** |
| Code review 文化 | 5 | **11**（3 次 WebSearch + 1 个反向 scratch） |

skill 现在强迫 agent **把自己的沉睡知识唤醒**——在定稿前去跨域资源里拉取新的 token，而不是对"已经在 context 里的东西"做一次性的 prose 生成。代价：token +34 %、wall-clock 时间 +40 %——对目标场景（高杠杆或开放式推理）完全值得。

## 示例 —— Code review 文化诊断

用户原始 prompt：*"我在观察一个现象：我们团队的 code review 文化虽然流程上很完整（每个 PR 都要求至少两个 reviewer 签字，checklist 很详尽，review 平均耗时 2 天），但实际效果在慢慢退化……在我写改进提案之前，帮我想清楚：这里真正发生了什么？"*

### 有 skill（节选）

> 你描述的现象——流程完备、耗时可观、但深度衰减——不是文化懈怠，也不是谁变懒了。它是一个**结构性均衡**。真正在发生的是：**评审这件事里可被度量、可被完成、可被签字的部分，正在挤出不可度量但价值更高的部分。** 这是 Goodhart 律的一个干净版本。
>
> 当两件经济属性差异如此之大的事被塞进同一容器，**廉价的那件一定会挤出昂贵的那件**——这是 Gresham 律在注意力经济上的翻版：劣币驱逐良币，不是因为人坏，而是因为定价相同。
>
> 在社会心理上，责任的扩散与人数成正比：当只有一位 reviewer 时，ta 是"发现深层 bug 的唯一责任人"；当有两位时，每人感觉中承担的比例不是 50%，而更接近 Ringelmann 效应下的 30-40%，深度付出反而跌得比一半更多。
>
> **半年后的维护者视角**：此刻漏掉的架构问题，Q3 会以 incident 的形式回来。评审和事故之间没有反馈回路——这是深度衰减最**隐秘**的原因：reviewer 从来不知道自己当初漏掉了什么。免疫系统在没有真实威胁的长期激活下会产生 anergy，评审也一样。
>
> 我会把这个模式叫做"**可见性俘获了判断工作**"（legibility capture of judgment work）：工作里**可见的那部分**（签字、checklist、SLA）俘获了团队的注意力与奖励结构，让**不可见但高价值的那部分**（深度判断）慢性失血。

### 无 skill（节选）

> 两个 reviewer、详尽 checklist、两天耗时——这些是"投入的形式"。而你说的深度问题减少，是"注意力的去向"。这两件事在同一个评审里是**竞争关系**，不是加总关系。
>
> checklist 越详尽，reviewer 越倾向于把脑力花在"逐项核对"上——这是一种低认知负荷、高确定性的任务，大脑偏好它。架构和潜在 bug 则相反：高负荷、低确定性、还可能引起冲突。
>
> 换句话说：**不是 reviewer 变懒了，是流程把"勤奋"重新定义成了"把清单跑完"。**
>
> 真正的杠杆点在注意力、激励和心理安全，不在 checklist。

**有 skill 的输出引入了跨域概念（Goodhart、Gresham、Ringelmann、免疫学 *anergy*）并发明了一个新的模式名。基线输出停留在 code review 这个框架内部。**

## 仓库内容

- `SKILL.md` —— skill 本体（复制到 `~/.claude/skills/awaken/SKILL.md`）
- `examples/` —— eval prompt + 基线、iteration-1、iteration-2 的输出并列（`with_skill_iter2.md` 是当前版本）
- `evals/` —— 机器可读的评测数据（prompt + 断言 + 两次迭代的 grading 结果）
- `benchmark.png` / `benchmark.svg` —— baseline / v1 / v2 pass-rate 对比

## 安装

```bash
mkdir -p ~/.claude/skills/awaken
cp SKILL.md ~/.claude/skills/awaken/SKILL.md
```

Claude Code 下次启动会自动发现 skill。

## 方法论

每次迭代内，每个 eval 都跑两次：一次给 subagent 读 `SKILL.md` 并应用协议；一次给同样 prompt 但不给 skill（基线不随迭代变化）。输出由独立的第三个 subagent 按 per-eval 断言评分（每个 eval 9 条断言，两次迭代断言不变，保证苹果对苹果比较）。完整的 prompt、输出与 grading JSON 都在 `examples/` 和 `evals/` 里，可复现。

**v1 到 v2 发生了什么。** 迭代 1 揭示出一个隐藏的弱点：skill 改变了输出**质量**，但 subagent 的 tool-call 行为几乎和基线一样（唯一多出来的是 `Read(SKILL.md)` 本身）。skill 成了纯 context 干预——它重塑了模型**写了什么**，但没有改变它**做了什么**。迭代 2 围绕"外化原则"重写协议——五个自问问题每一个都有了首选的 tool-call（WebSearch 找跨域类比、`Write` 落盘反向思考 scratch 文件、并行子 `Agent` 做视角遍历、对载重未知用 `AskUserQuestion`）。结果：每次运行的 tool-call 数 ~3×（3.7 → 11），pass rate 触顶 100 %，并且三个 eval 的定性 pattern 出奇一致——"agent 拒绝在用户给定的 framing 里直接作答，它先浮现一个结构性重构。"

**迭代 3 的开放问题。** v2 的行为改变是由 `SKILL.md` 内的**文本指令**驱动的服从——subagent *读*了 skill *并按*它的外化指引行动。这种服从在**对抗条件**下是否还成立（繁忙的主对话 context、多 skill 同时触发、时间压力下的 fallback），还没测。那是下一个压测方向。

## 致谢

- **方法论**：Juntao Chi —— [《是否存在一种能建构一切认知与智慧的思考方法论？》](https://juntaochi.com/en/posts/is-there-a-methodological-approach-to-thinking-that-can-build-all-cognition-and-wisdom/)
- **脚手架**：用 Anthropic 的 [skill-creator](https://github.com/anthropics/skills) 工作流搭建

## 许可证

MIT
