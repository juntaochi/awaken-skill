# 你们的 code review 文化发生了什么

## 先把现象的结构剥掉领域名词

抛开"代码评审"这个具体外壳，你描述的是一个更通用的结构：**一套为了保证某种"质量"而设置的验证流程，随时间演化出一个可观测、可记录、可交付的外壳（两个签字、checklist、2 天周期），而这个外壳本身逐渐替代了它原本要保护的那个"质量"**。社会学里对这个结构有非常成熟的命名——Meyer & Rowan 所说的 **decoupling**（形式结构与实质活动脱钩），Michael Power 在《The Audit Society》里直接称之为"**验证的仪式化**"（rituals of verification）。审计行业、学校的教学评估、食品安全认证、甚至医院的 length-of-stay 指标——走的是同一条路径，不是巧合。

这个同构关系很关键，因为它告诉你：**问题不在你们团队的纪律，也不在单个 reviewer 的敬业度**。你们正在经历的是一个在结构上会稳态发生的过程，只要满足几个条件，它就会出现。你们满足了这些条件。

## 正在同时发生的三股力量

**第一股是 Goodhart / Campbell 效应的慢速版。** 你们一开始用"两个签字 + checklist"作为"有认真评审"的**代理指标**。当这个代理被当成目标本身——reviewer 要按时签字、checklist 要填完、PR 要动起来——它就不再是"认真评审"的有效度量了。真正的评审质量（架构、潜在 bug、设计债）从来没有被直接观测过，于是它在优化的引力下悄悄让位。这不是有人在偷懒，而是系统在被"评审发生了"这个可观测事实所替代，"评审起作用了"这个不可观测事实被静默放弃。

**第二股是"注意力的 Gresham's law"。** 评审者的注意力是有限额度的。checklist 上的每一项、风格规则、边界条件——这些是**廉价信号**：容易识别、容易指出、没有社交成本、容易被 author 接受、不会引发争论。架构问题、设计气味、跨模块耦合——这些是**昂贵信号**：需要 reviewer 先 load 整个上下文，成本高、易被反驳（"这不是这个 PR 的范围"）、可能延迟合并、可能伤关系。当两种信号共用一个稀缺槽位时，廉价信号会稳定驱逐昂贵信号。*不是有人决定不谈架构——而是谈架构的边际成本在这套流程下已经比廉价评论高太多。*

**第三股是 2 天延迟本身制造的反馈。** 两天之后，author 已经上下文切换，深度评论的**行权成本**（重构、merge 冲突、解释沟通、二次评审）显著上升。reviewer 对此有潜意识感知，于是把深度评论压缩为自己能承受的那一档——风格和边界。延迟不是中性事实，它是一种**对深度的税**。

## 核心张力，不能被调和只能被看见

**可验证性 ↔ 判断力**——你们想要的东西（架构洞察、设计品味）本质上是不可流程化的判断工作；你们用来保障它的东西（两个签字、checklist、平均耗时）本质上是可流程化的合规工作。你们是在**用可验证的外壳来承接不可验证的内核**，这两个东西在长期里会分离。Power 叫这个现象 "legibility capture of judgment work"——判断工作被"可被记录的东西"捕获，于是它的边缘被磨掉，只剩下能被 checklist 容纳的那部分。

## 从三个位置看同一件事（它们不应该得出同一个结论）

*从流程设计者的位置*看：流程是完整的，指标是正向的，每个 PR 都合规了，表面上"这就是 code review 应有的样子"——问题是隐形的。

*从一个被你们期待去做深度评审的 senior 工程师的位置*看：每次要做一个架构层面的评论，他都在默默权衡"这个话题说出去之后的下游成本"（讨论、延误、被标记为难缠），而他从"我今天指出了一个架构问题"里收到的组织奖励是零。在这个梯度下，他会训练出一种"只评论安全区"的本能，并把它误以为是自己的风格。

*从六个月后收到技术债账单的未来团队的位置*看：每个现在的绿色 check 都是一份未来债务的预付凭证。但这个位置在你们当下的流程里**完全没有代言人**——没有人在评审里代表"六个月后会痛的我们"说话。这是所有此类系统的共同盲点：未来没有座位。

这三个视角不应该收敛到同一个结论，如果它们收敛了，说明你还没有真正站到不同的位置上。

## 这到底是什么——精确命名

这不是"评审文化退化"。这是一个更具体的现象，有现成的名字：

**这是一次 decoupling 事件**——你们的形式结构（PR 流程）和它要保护的实质活动（判断性评审）发生了松耦合。并且这次 decoupling 不是失败，而是**按系统的激励梯度成功了**：每个参与者在他的局部都做了理性的事，整体却滑到了一个实质质量降低的稳态。

这也是 **Goodhart + Gresham 的复合**：代理指标取代了目标，廉价信号驱逐了昂贵信号。这两个机制相互强化。

所以你在写改进提案之前真正要想清楚的是：任何还在**加强流程、细化 checklist、增加 reviewer 数量、引入更多 bot 检查**的改动，都是在同一个方向上继续走——它们会让 decoupling 更完整，不是相反。要逆转这个过程，你需要动的不是流程的"表面积"，而是**昂贵信号的经济学**——让架构层面的评论成本更低、奖励更清晰、延迟更短、在未来有代言人。具体怎么动是下一步的话题，但前提是先承认：现在的状态不是纪律问题，是激励结构的稳态输出。

## Sources

- [Institutionalized Organizations: Formal Structure as Myth and Ceremony (Meyer & Rowan)](https://www.journals.uchicago.edu/doi/abs/10.1086/226550)
- [The Audit Society: Rituals of Verification (Michael Power)](https://ideas.repec.org/r/oxp/obooks/9780198296034.html)
- [Paperwork and the decoupling of audit (Escobar & Demeritt)](https://journals.sagepub.com/doi/full/10.1177/0263774X16646771)
- [Supermarkets and private standards: unintended consequences of the audit ritual](https://link.springer.com/article/10.1007/s10460-012-9414-6)
- [Goodhart's law](https://en.wikipedia.org/wiki/Goodhart's_law)
- [Goodhart's Law, Campbell's Law, and the Cobra Effect](https://psychsafety.com/goodharts-law-campbells-law-and-the-cobra-effect/)
- [Gresham's law](https://en.wikipedia.org/wiki/Gresham's_law)
- [Gresham's Law: The Bad Drives Out the Good As Time Passes (Farnam Street)](https://fs.blog/mental-model-greshams-law/)
