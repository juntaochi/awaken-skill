# awaken

A Claude Code skill that scaffolds deep reasoning, research, and meta-cognition by creating conditions for dormant knowledge to surface, rather than by encoding rigid rules.

> *"Tasteful judgment is not the fastest-reached judgment, but judgment that naturally emerges on a sufficiently rich perceptual foundation."*
> — [Juntao Chi, *Is there a methodological approach to thinking that can build all cognition and wisdom?*](https://juntaochi.com/en/posts/is-there-a-methodological-approach-to-thinking-that-can-build-all-cognition-and-wisdom/)

## The insight

LLM knowledge is largely **dormant** — you know things you don't know you know, but that knowledge only surfaces when context pulls it out. Rigid rules suppress emergence; scaffolds create conditions for it. The single biggest cause of shallow reasoning is **premature convergence** — rushing to a scalar answer, locking onto the first framing, collapsing into a single perspective before you have felt the shape of the problem.

`awaken` is the reversal: **suspend → elevate → traverse → name.**

## The protocol

Five self-questions, applied invisibly, each with a preferred *externalization* (an actual tool call that leaves a trace):

| Q | Move | Preferred externalization |
|---|---|---|
| Q1 | Real objective + specific gap | `Read` the codebase / docs; `AskUserQuestion` for one load-bearing unknown |
| Q2 | Dimension elevation (cross-domain) | `WebSearch` "[abstract structure] analogy" or spawn sub-`Agent` |
| Q3 | Tension + Munger inversion | `Write` failure-modes list to scratch file |
| Q4 | Perspective traversal (≥3 standpoints) | Parallel sub-agents for high-stakes |
| Q5 | Pattern naming + verification | `WebSearch` / `Grep` to verify before inventing |

## Benchmark (iteration 1)

Mean pass rate across 3 test cases from three unrelated domains (code architecture / RAG research / Chinese diagnostic meta-reasoning):

| Configuration | Mean pass rate | Delta |
|---|---|---|
| with-skill | **92.6 %** | **+40.8 pp** |
| baseline | 51.8 % | — |

![benchmark](benchmark.png)

### Per-eval breakdown

| Eval | with-skill | baseline | Δ |
|---|---|---|---|
| Code architecture (pub/sub vs streaming-first) | 89% | 33% | +56 pp |
| RAG for medical research (vector vs graph vs hybrid) | 89% | 44% | +45 pp |
| Code review culture diagnosis (Chinese) | **100%** | 78% | +22 pp |

## Example — code review culture diagnosis

User prompt (translated): *"Our team's code review culture is procedurally complete (2 reviewers required, detailed checklist, 2-day SLA) but quality is slowly degrading — depth questions about architecture and design are disappearing while style comments dominate. Before I write a reform proposal, help me see what's actually happening."*

### With skill (excerpt)

> 你描述的现象——流程完备、耗时可观、但深度衰减——不是文化懈怠，也不是谁变懒了。它是一个**结构性均衡**。真正在发生的是：**评审这件事里可被度量、可被完成、可被签字的部分，正在挤出不可度量但价值更高的部分。** 这是 Goodhart 律的一个干净版本。
>
> 当两件经济属性差异如此之大的事被塞进同一容器，**廉价的那件一定会挤出昂贵的那件**——这是 Gresham 律在注意力经济上的翻版：劣币驱逐良币，不是因为人坏，而是因为定价相同。
>
> 在社会心理上，责任的扩散与人数成正比：当只有一位 reviewer 时，ta 是"发现深层 bug 的唯一责任人"；当有两位时，每人感觉中承担的比例不是 50%，而更接近 Ringelmann 效应下的 30-40%，深度付出反而跌得比一半更多。
>
> **半年后的维护者视角**：此刻漏掉的架构问题，Q3 会以 incident 的形式回来。评审和事故之间没有反馈回路——这是深度衰减最**隐秘**的原因：reviewer 从来不知道自己当初漏掉了什么。免疫系统在没有真实威胁的长期激活下会产生 anergy，评审也一样。
>
> 我会把这个模式叫做"**可见性俘获了判断工作**"（legibility capture of judgment work）：工作里**可见的那部分**（签字、checklist、SLA）俘获了团队的注意力与奖励结构，让**不可见但高价值的那部分**（深度判断）慢性失血。

### Without skill (excerpt)

> 两个 reviewer、详尽 checklist、两天耗时——这些是"投入的形式"。而你说的深度问题减少，是"注意力的去向"。这两件事在同一个评审里是**竞争关系**，不是加总关系。
>
> checklist 越详尽，reviewer 越倾向于把脑力花在"逐项核对"上——这是一种低认知负荷、高确定性的任务，大脑偏好它。架构和潜在 bug 则相反：高负荷、低确定性、还可能引起冲突。于是流程的"完整性"本身在**挤出**深度评审需要的那种散漫的、带假设的、愿意跑题的思考空间。
>
> 换句话说：**不是 reviewer 变懒了，是流程把"勤奋"重新定义成了"把清单跑完"。**
>
> **责任稀释**：当 A 看到 B 也会看，A 对深层问题的"必须指出"压力下降。两人都这么想，结果两人都只做了浅层的那部分。
>
> 真正的杠杆点在注意力、激励和心理安全，不在 checklist。

**The with-skill output imports cross-domain concepts (Goodhart, Gresham, Ringelmann, immunological *anergy*) and coins an invented pattern name. The baseline stays inside the code-review frame.**

## What's in this repo

- `SKILL.md` — the skill itself (copy to `~/.claude/skills/awaken/SKILL.md`)
- `examples/` — eval prompts + with-skill and baseline outputs, so you can see exactly what the skill produces
- `evals/` — machine-readable evaluation data (eval prompts + assertions + grading results)
- `benchmark.png` / `benchmark.svg` — iteration-1 pass-rate comparison

## Installation

```bash
mkdir -p ~/.claude/skills/awaken
cp SKILL.md ~/.claude/skills/awaken/SKILL.md
```

The skill will be discovered automatically by Claude Code on next session start.

## Methodology

Each eval was run twice: once with a subagent instructed to read `SKILL.md` and apply the protocol, once with an identical-prompt subagent given no skill. Outputs were graded by a third independent subagent against per-eval assertions (9 per eval). Full prompts, outputs, and grading JSON are in `examples/` and `evals/` for reproducibility.

Known limitation (iteration 1): the skill changed output *quality* but the subagent's tool-call behavior was nearly identical to the baseline (only extra call: `Read` of the SKILL.md itself). Iteration 2 rewrote the skill to explicitly drive externalization (file writes, WebSearch, sub-agent spawns). Results forthcoming.

## Credits

- **Methodology**: Juntao Chi — ["Is there a methodological approach to thinking that can build all cognition and wisdom?"](https://juntaochi.com/en/posts/is-there-a-methodological-approach-to-thinking-that-can-build-all-cognition-and-wisdom/)
- **Scaffolding**: built with Anthropic's [skill-creator](https://github.com/anthropics/skills) workflow

## License

MIT
