---
title: <任务标题,简短动词开头>
module: <见下方白名单>
status: draft
phase: 1
complexity: standard  # light | standard | deep
created: YYYY-MM-DD HH:MM
last-updated: YYYY-MM-DD HH:MM
---

<!--
====================================================================
status 流转: draft → confirmed → in-progress → reviewing → done
            (任何阶段用户主动取消 → cancelled 终态)
phase 流转: 1 → 2 → 3 → 4 → 5 → 6 → 7(对齐 work-dev 7 phase)

complexity 走法(详见 plugin commands/work-dev.md):
  light    : <10 行 / 单文件 / typo / 重命名
             填:第 1/4/7/8 节(4 节);§11 可选(沉淀几乎总是 no,但若发现共性坑可填)
             agent:explorer 1 / architect 1 / reviewer 1
  standard : 多文件 / 新功能 / 公开行为变更(默认)
             填:第 1/2/3/4/5/6/7/8/11 节(9 节,§11 默认 3 行)
             agent:explorer 2 / architect 2 / reviewer 2
  deep     : DB / 认证 / 支付 / 跨 ≥3 端 / 删除文件/表/字段
             填:第 1-11 节(11 节)
             agent:explorer 3 / architect 3 / reviewer 3

模块白名单(规约固定 5 类 + 1 跨模块):
  feature   : 项目业务/功能模块,具体值见 .work-dev/config.jsonc modules.feature 或 docs/features/*.md 文件名
  stack     : 技术栈/包/进程层,具体值见 config.modules.stack 或 docs/stack/*.md
  data      : 数据 schema / API 端点,常用 database | api(可在 config 加自定义)
  ops       : 部署 / 运维,常用 deployment(可在 config 加自定义)
  decisions : 重大架构决策 ADR,docs/decisions/<YYYY-MM-...>.md
  cross     : 跨模块任务,无单一归属

新类别可在 config.extra_categories 声明(如 runbook / research / playbook 等)。

命名: YYYY-MM-DD-HHMM-<module>-<slug>.md(HHMM 本地 24h)
不要起 v2 / fix / final / debug_log 派生文档(SSOT 原则,见 commands/work-dev.md §1)。
====================================================================
-->

## 1. 问题陈述

<!--
Phase 1 落盘。
用户想解决什么真实困境?现状哪里不通畅?为什么现在做?
light 可只写"现象 + 触发"。
-->

## 2. 根因分析

<!--
Phase 2 落盘。standard / deep 必填,light 可省。
5-Why 链 + 历史相似坑(自动从 _lessons.md 召回)。
-->

**5-Why 链**

- 表象:
- 为什么 →
- 为什么 →
- 根因:

**问题类型**:产品定义模糊 / 设计缺陷 / 实现 bug / 性能瓶颈 / 数据问题

**历史相似坑**(从 _lessons.md 召回,Phase 1 自动写入)

- (无,或列出命中条目)

## 3. 影响面分析

<!--
Phase 2 落盘。standard / deep 必填,light 可省。
5 维各 1 句话;deep 可补 1-2 个证据源。
-->

| 维度 | 分析 |
|------|------|
| 代码消费方 | 谁在用我要改的代码? |
| 数据兼容 | 老 DB / JSON / Storage 怎么办? |
| 性能影响 | 多一次 join / AI / 网络? N+1? |
| 跨端一致性 | shared / 多端是否同步? |
| 回滚成本 | git revert 干净? DB 可逆? |

## 4. 涉及代码文件

<!--
Phase 2 落盘。只列代码文件,docs 同步交给 Phase 7 守门员扫描。
执行中发现新文件 → 追加,不要静默扩张。
-->

- (按 code_roots 分组列出)

## 5. 澄清 Q&A

<!--
Phase 3 落盘。只问真正阻断的硬伤,无下限题数(light 可能 0 题)。
-->

- Q1: ? → A:
- Q2: ? → A:

## 6. 方案对比

<!--
Phase 4 落盘。standard 默认 2 方案,deep 3 方案,light 可只列 1 方案。
-->

| 方案 | 关键决策 | 优 | 劣 |
|------|---------|----|-----|
| A    |         |    |     |
| B    |         |    |     |

**推荐**:A
**理由**:
**用户选择**:(待确认 / A / B / Other)

## 7. 拆解步骤

<!--
Phase 4 落盘。
`- [ ]` 未完成 / `- [x]` 完成 / `- [!]` 留下批 PR。
不再包含"同步 docs"步骤;docs 同步由 Phase 7 守门员自动跑。
-->

- [ ] 1.
- [ ] 2.

## 8. 验收标准

<!--
Phase 4 落盘。具体清单,不写"测试通过"空话。
不再包含"文档同步"勾选项;docs 同步由 Phase 7 守门员负责。
-->

**功能性**

- [ ] Happy path:
- [ ] 边界 1:
- [ ] 边界 2:

**不破坏**

- [ ] 测试全绿(`{config.test_commands}` 全部 key)
- [ ] 现有 happy path 不变(列出具体几个)

**测试**

- [ ] 单测覆盖:
- [ ] 集成测试:
- [ ] 手动验证清单:

## 9. 测试与 Review 摘要

<!--
Phase 6 落盘。
合并 work-dev 分散在 Phase 5 / 6 的测试 + reviewer agent findings 输出。
-->

**测试结果**

- (按 {config.test_commands} key 分组)

**Multi-agent Findings**(reviewer agent 合并去重)

- 高 / 中 / 低: <issue>(reviewer-X,处理: fix now / fix later / proceed)

**验收勾选**(对照第 8 节)

- [ ] / [x] / [!] 逐项标注

## 10. 变更记录

<!--
Phase 7 落盘。本次任务一次性写完,不每步追加,不含 commit hash(git log 是真相)。
-->

- YYYY-MM-DD HH:MM 创建任务文档(complexity=X)
- YYYY-MM-DD HH:MM 完成 phase 4,落盘方案对比 + 拆解 + 验收;用户批准方案 A
- YYYY-MM-DD HH:MM phase 6 review 通过;<N> 文件 +<X>/-<Y> 行
- YYYY-MM-DD HH:MM phase 7 收尾;docs 漂移扫描同步 <N> 个文件;status=done

## 11. 复盘

<!--
Phase 7 落盘。默认三行(问题 / 方案 / 沉淀?);用户喊 `retro` 才展开下方"完整复盘"段。
沉淀判定见 commands/work-dev.md Phase 7 步骤 3 四问;只有四问全 yes 才追加到 _lessons.md。
每个任务最多沉淀 1 条新规则;凑不出 1 条就 0 条,不强求。
-->

**问题**:(1 句话,这次卡在哪)
**方案**:(1 句话,怎么解的)
**沉淀?**:no <!-- yes/no;默认 no,yes 时必须满足四问且填写下方"追加到 _lessons.md 的条目" -->

**追加到 _lessons.md 的条目**(仅当"沉淀? = yes"时填)

- (一句话祈使句规则,例:"改 DB schema 前先 grep 字段名 in {code_roots}/ 防漏改 caller")

---

**完整复盘**(仅 `retro` 模式填,默认折叠)

<!--
retro 仅 3 题:什么坑 / 根因 / 下次怎么避免。
工时 / 澄清 / 方案回看等季度回顾向问题不放在任务复盘。
-->

- 踩了什么坑(具体到代码 / 文档 / 流程,1-2 句话):
- 根因(技术 / 流程 / 沟通哪一类,1 句话):
- 下次怎么避免(操作层面,可执行,1 句话):
