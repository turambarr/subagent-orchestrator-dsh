# subagent-orchestrator-dsh

实验编排 SOP 技能（DeepSeek Harness Skill）：**顶层模型（pro）控制实验设计与验收，智能调用轻量子智能体（flash）执行确定性子任务。**

## 这是什么

一个把「pro 控制 / flash 执行」的实验分工写死的可复用技能，解决长周期实验里"每一步该拆给谁、怎么下指令、怎么验收"的问题。

核心机制：

- **唯一硬规则**：flash 能接 = 过程确定 + 派工前能写下「数字 + 容差」。
- **分解计划**：每实验一次，pro 起草、用户确认后才派工（顶层控制显式化、可审查）。
- **自包含 prompt 模板**：采集 / 执行 / 校验 / 审查 / 转换 五类，flash 零上下文也能干。
- **两级验收 + 四级证据门 + 三条红线**：低级证据不许冒充高级证据。

## 结构

```
skills/experiment-orchestration/
├── SKILL.md                    # 主指令：核心循环 + 硬规则 + 模型映射
├── prompt-templates/
│   ├── plan.md                 # 分解计划模板
│   ├── fact-gather.md          # 采集
│   ├── execute-verify.md       # 执行
│   ├── consistency-check.md    # 一致性校验
│   ├── read-only-review.md     # 只读审查
│   └── mechanical-transform.md # 机械转换
└── acceptance/
    └── gates.md                # 两级验收 + 四级门 + 三条红线
```

## 安装

```bash
dsh plugin --profile <profile> add subagent-orchestrator-dsh
```

## 领域扩展（可选）

本技能不内置领域专属内容。需要接入特定领域时，新增 `domain/<name>/appendix.md`，核心机制不动。

## License

MIT
