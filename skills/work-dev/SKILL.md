---
name: work-dev
description: 当用户描述适合 7 阶段开发工作流的编码任务时调用 — 新功能 / bug 修复 / 重构 / 性能优化等需要走 explore→clarify→design→implement→review→wrap 循环并配合任务文档实时落盘 + docs 漂移守门。也通过 `/work-dev light|standard|deep <描述>` 显式命令触发。
---

# work-dev — 通用 7 阶段开发工作流 Skill

This skill is the description-based entry point for the `cc-work-dev` plugin. For full command behavior, read `commands/work-dev.md` in this plugin and follow it exactly.

## 触发场景

调用此 skill 当用户:

- 输入显式命令 `/work-dev light <描述>` / `/work-dev standard <描述>` / `/work-dev deep <描述>`
- 描述一个**需要分析 + 实施**的代码改动(不只是问问题),例如:
  - "帮我修 X bug"
  - "实现 Y 功能"
  - "重构 Z 模块"
  - "优化 W 性能"
  - "调整 V 配置"
- 用户提到「按工作流走」/「先分析再动手」/「完整流程」等关键词

## 不要触发

- 纯问答 / 概念解释 / 文档查询
- 单行 typo / 显而易见的修改
- 用户已明确"直接做,不要走流程"
- 单纯调试一个错误信息(无后续 fix-then-recheck 回路)

## 触发后的动作

完整流程定义在 `commands/work-dev.md`,关键约束:

1. 严格 7 phase(Discovery → Explore → Clarify → Design → Implement → Review → Wrap)
2. AskUserQuestion 仅 2 次必用节点(Phase 4 选方案 + Phase 6 fix/proceed)
3. 任务文档按 phase 实时落盘到 `{tasks_root}/`
4. docs 漂移 Phase 7 自动守门员扫描 + 同步
5. agent 数随复杂度(light=1 / standard=2 / deep=3)
6. 项目配置从 `.work-dev/config.jsonc` 读(可选,无则自动推断)

读 `commands/work-dev.md` 后按其指令执行,不要在本 skill 文件中复述流程细节。

## Skill 与 Command 的关系

- 用户输入显式斜杠命令 `/work-dev ...` 时,**以 `commands/work-dev.md` 为唯一入口**,本 skill 不应再独立触发(避免双重 prompt 注入)。
- 用户用自然语言描述任务时,本 skill 提供 description-based 触发,触发后立即引用 `commands/work-dev.md` 的流程,不再复述。
