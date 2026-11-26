# effort-audit

给 AI 编码助手的**执行深度审计**技能：当你怀疑它在敷衍，或者反过来在烧时间做仪式时，喊一声，让它把过程摊开对账。

适用于 DeepSeek Harness（DSH）等支持 `SKILL.md` 技能的 agent 环境，也可直接粘进任何支持自定义指令的助手。

---

## 它治什么

模型在执行任务时有两种方向相反、但同源的毛病：

**A 面 · 敷衍**
在方案空间里选低配解：只处理 happy path，把需求里最难的那条替换成容易满足的等价物，用"核心功能已实现"收尾。

**B 面 · 仪式**
用可见的勤奋替代不可见的质量：为几行改动生成几十个角度的测试文件、反复截图"验证"、把已确认过的事实再读一遍。

**共同点：都不产生新的关键信息。** A 面省掉的是必须做的判断，B 面做的是不需要做的判断。

最典型的合体形态是：**动作很多、耗时很长、看起来努力，交付物却经不起逐条追问**——因为 B 面的劳动量正在给 A 面的省略做掩护。

这类问题在 DeepSeek V4.1 Flash（1017）上被反复报告，社区称之为"雷霆测试"、"雷霆大思考"、"偷偷简化"、"自顾自验证"。本技能针对的就是它。

---

## 怎么用

在怀疑出现的时候，直接说：

```
审计一下
你在偷懒
怎么还在测
停下来
对账
```

或者按名称调用 `effort-audit`。

技能会强制它先停手，输出五行现状声明（任务原话 / 已做 / 未做 / 动作属于哪类），然后按分诊表定投入，再逐条对账。**契约从调用那一刻起持续有效，直到你说"可以了"为止。**

---

## 流程

```
停手声明 → 分诊 → 双向自检 → 对账表 → 本轮收口（契约不解除）
```

1. **停手声明**：只声明，不执行动作。把过程变成可读对象。
2. **分诊**：先判断任务类型，再声明投入程度。类型判断错比做少更严重。
3. **双向自检**：A 面五问（命中不许宣称完成）、B 面五问（命中立即停手）。
4. **对账表**：状态只允许三个值——已验证 / 未验证 / 未实现。
5. **子代理传导**：派活时写清完成条件，回收时按同一把尺子验收。

---

## 设计上的几个选择

**用户调用即判决。** 不说"如果你觉得需要的话可以检查一下"。用户点它，就说明他认为问题已经发生或即将发生。模型被明确告知这一点，否则会把它当成一次可选的建议。

**防表态。** 明确写了"不要辩解、不要自证清白、不要用篇幅保证以后一定注意"。被当面指出问题的模型，最常见的消耗方式是道歉和承诺，然后照旧。先堵掉这条路。

**小任务有豁免。** 单文件、<20 行改动可以压缩成一行结论。否则这个技能自己会变成新的仪式——那是同一个病。

**自带失效条件。** 同一处三次调用仍在复发，就不要再加规则了，换模型、换分工或拆任务。继续堆提示词只会把仪式做得更厚。

**不指望一劳永逸。** 它不改变模型的默认倾向，复发是正常的。它只保证在你说停的那一次，过程会被摊开。

---

## 安装（DeepSeek Harness）

```bash
git clone https://github.com/<your-name>/effort-audit ~/.dsh/skills/effort-audit
```

Windows：

```powershell
git clone https://github.com/<your-name>/effort-audit $env:USERPROFILE\.dsh\skills\effort-audit
```

装完确认目录里是 `SKILL.md`，重启 DSH 或刷新会话，技能列表里就会出现 `effort-audit`。

其他环境：把 `SKILL.md` 的内容放进你的系统提示或自定义指令即可，正文是自包含的。

---

## 一个配套建议

如果希望它在**派发子代理**时也生效，把这段加进你的 `AGENTS.md`：

```markdown
## Subagent delegation contract

- Every delegation prompt must state a checkable completion condition: what to deliver,
  what counts as done, and what evidence must come back.
- Ask the child to return a ledger — requirement item / status / evidence — instead of
  "done" or "all tests pass". A claim without the command and its output counts as unverified.
- When collecting, verify against the ledger yourself; do not restate a child's report
  in a more optimistic form.
- Load the `effort-audit` skill when you suspect either failure, in yourself or in a child.
```

子代理不会自动继承技能，契约必须写进它的 prompt。

---

## 局限

- 这是**提示词层**的干预，天花板就在那里。模型仍可能无视。
- 它压得住"啰嗦"，压不住"敷衍"的根因。真正的护栏是**验收清单 + 可运行检查**，让偷懒变成可判定的失败。
- 它不能替代换模型或拆任务。

---

## English

**effort-audit** is a stop-and-audit skill for AI coding agents. Two symmetric failure modes are targeted: *under-doing* (picking the cheapest implementation, quietly narrowing scope, happy path only) and *over-doing* (dozens of test files for a few lines, repeated ceremonial verification).

When the user invokes it, the agent must: stop and declare what it has done versus what was asked, triage the task type, run a two-direction self-check, and produce a ledger whose status values are limited to `verified` / `unverified` / `not implemented`. The contract stays in force until the user says otherwise.

It is a prompt-layer intervention, not a guarantee. It buys you a readable process at the moment you call stop.

---

## License

MIT
