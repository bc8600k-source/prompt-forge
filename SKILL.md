---
name: prompt-forge
description: 融合 5 个开源 prompt 优化 skill 精华，支持评分、框架推荐、eval 驱动迭代。
version: 1.0.0
author: 知微
license: MIT
metadata:
  hermes:
    tags: [prompt, optimization, evaluation, skill]
    related_skills: [cl-skill-learn, cl-continuous-learning]
---

## When to Use

- 用户要求优化/改进 prompt 时
- 用户要求评分/分析 prompt 质量时
- 用户要求写新 prompt 时
- cron 输出质量不稳定需要系统化改进时

---

# prompt-forge — 融合型 Prompt 优化引擎

## 核心理念

**数据驱动，不凭感觉。** 每次改动都有测量，分数升了才保留。

---

## 三种工作模式

### 模式 A：评分（6 维）

| 维度 | 问题 |
|------|------|
| **Clarity** | 无上下文的 LLM 能无歧义理解吗？ |
| **Specificity** | 约束、输出格式、期望行为明确吗？ |
| **Structure** | 信息逻辑组织、层级清晰吗？ |
| **Completeness** | 覆盖上下文、示例、边界情况、错误处理吗？ |
| **Efficiency** | 每句话都承载必要信息吗？ |
| **Robustness** | 10 次运行能产生一致高质量输出吗？ |

每维 1-10 分，输出 scorecard + 最弱维度分析。

### 模式 B：框架推荐

根据任务类型推荐框架：

| 任务类型 | 推荐框架 |
|----------|----------|
| 内容创作 | CO-STAR |
| 多步骤流程 | RISEN |
| 数据分析 | RISE-IE |
| 专家任务 | RACE |
| 简单任务 | RTF / APE |
| 改写/重构 | BAB |
| 推理/决策 | Tree of Thought / CoT |
| Agent 任务 | ReAct |
| 迭代改进 | Self-Refine |

### 模式 C：Eval 驱动迭代

```
1. 定义 eval suite（测试用例）
2. 跑 baseline → 记录分数
3. 改一个变量
4. 重跑 → 对比分数
5. 分数升 → 保留；分数降 → 回滚
6. 重复直到达标
```

---

## Eval Suite 格式

```yaml
cases:
  - input: "<输入>"
    expect: "<期望输出>"
    assert:
      - "<硬性检查>"
    rubric:
      - name: "<软性维度>"
        weight: 0-1
        criteria: "<评分标准>"
```

**硬性检查**：确定性断言（字数、格式、必需字符串）
**软性评分**：LLM-as-judge（语气、连贯性、角色一致性）

---

## 质量闸门

每次改写必须通过四阶段闸门：

1. **行数限制**：不超过 200 行
2. **格式验证**：结构完整
3. **LLM 评分**：不低于当前分数
4. **最小提升**：Δscore ≥ 0.05

任一阶段失败 → 拒绝改写，保留当前版本。

---

## 反模式检测

| 反模式 | 症状 | 修复 |
|--------|------|------|
| 目标模糊 | "帮我优化" | 明确目标受众和成功标准 |
| 约束缺失 | 输出不稳定 | 加 dos/don'ts |
| 无示例 | 输出格式不一致 | 加 3-5 个 few-shot |
| 无错误处理 | 边界情况崩溃 | 加 fallback 规则 |
| 信息冗余 | token 浪费 | 删除不承载信息的句子 |
| 角色缺失 | 输出风格漂移 | 加角色定义 |

---

## 使用步骤

### 评分模式
```
1. 用户提供 prompt
2. 6 维评分
3. 输出 scorecard
4. 最弱维度分析
5. 针对性改进建议
```

### 迭代模式
```
1. 用户提供 prompt + eval suite
2. 跑 baseline
3. 进入迭代循环
4. 每次改一个变量
5. 分数升 → 保留
6. 达标 → 停止
```

### 创作模式
```
1. 用户描述任务
2. 分类任务类型
3. 推荐框架
4. 生成 v0
5. 进入迭代循环
```

---

## 输出格式

```
## Prompt Scorecard v<N>
Clarity:      <N>/10  (<+/-N>)
Specificity:  <N>/10  (<+/-N>)
Structure:    <N>/10  (<+/-N>)
Completeness: <N>/10  (<+/-N>)
Efficiency:   <N>/10  (<+/-N>)
Robustness:   <N>/10  (<+/-N>)
---------------------------
Composite:    <N>/10  (<+/-N>)
Weakest:      <维度>
Verdict:      <通过/不通过>
```

---

## 注意事项

- 每次只改一个变量
- 分数相同时选更短的版本
- 保留原意，不改变目标
- 所有分数标注依据
- 信息不足时说「信息不足」