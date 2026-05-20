# claude-plugin-work-dev

7-phase development workflow plugin for Claude Code.

> ⚠️ 仓库正在搭建中(M1 骨架完成,M2-M7 待实施)。完整文档详见 M6 完成后。

## 设计依据

抽象自 [Work Score 项目 work-dev v3 工作流](https://github.com/lichungang/work_score/blob/main/.claude/commands/work-dev.md)(385 行)。

设计 ADR(暂引用本地路径,M8 发布时改 GitHub 链接):
- `~/Downloads/work_score/docs/decisions/2026-05-20-work-dev-plugin-config-interface.md`

## 当前进度

- [x] M0 启动决策(独立 GitHub 仓库 / claude-plugin-work-dev / GitHub Release)
- [x] M1 plugin 骨架搭建
- [ ] M2 `commands/work-dev.md` 剥离 + 精简(目标 ≤ 300 行)
- [ ] M3 自动推断 + `/work-dev init` 子命令逻辑
- [ ] M4 skill + 3 agents(基于 feature-dev 改写)
- [ ] M5 种子资源 + JSON Schema
- [ ] M6 完整 README + LICENSE 文案
- [ ] M7 Work Score 迁移验证(质量门)
- [ ] M8 GitHub Release 发布

## 目录结构(方案 4)

```
claude-plugin-work-dev/
├── .claude-plugin/plugin.json     ✓ M1 已写
├── commands/work-dev.md           (M2)
├── skills/work-dev/SKILL.md       (M4)
├── agents/
│   ├── work-dev-explorer.md       (M4)
│   ├── work-dev-architect.md      (M4)
│   └── work-dev-reviewer.md       (M4)
├── templates/
│   ├── _template.md               (M5)
│   └── _lessons.md                (M5)
├── config/work-dev.example.jsonc  (M5)
├── schema/work-dev.schema.json    (M5)
├── README.md                      (M6 完整版)
└── LICENSE                        ✓ M1 已写 (MIT)
```

## 设计要点(完整内容见 ADR + M6 README)

- **双层触发**:`/work-dev light|standard|deep <描述>` 斜杠命令 + description 语义触发 skill
- **项目配置**:`.work-dev/config.jsonc`(可选,15 字段,零配置走自动推断)
- **两层 override**:plugin 默认 + 项目级 config
- **7 phase 主流程**:Discovery / Explore / Clarify / Design / Implement / Review / Wrap
- **agent 角色**:explorer / architect / reviewer(数量随 light=1 / standard=2 / deep=3)
- **核心机制**:任务文档实时落盘 / Phase 7 docs 漂移守门 / `_lessons.md` 召回 + 追加
