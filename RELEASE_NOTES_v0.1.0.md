# cc-work-dev v0.1.0 — Initial Release

> 草稿(M8.1 push 后用作 GitHub Release notes)

## 🎯 What is cc-work-dev?

通用 **7 阶段开发工作流** Claude Code plugin。把不规范的"开发对话"约束成强结构、可中断恢复、自动同步 docs 的工作流。

抽象自 [Work Score 项目 work-dev v3](https://github.com/lichungang/work_score)(385 行项目定制 → 214 行通用命令 + 15 字段项目配置)。

## ✨ 核心特性

- **三档复杂度**:`/work-dev light <描述>` / `standard` / `deep`,agent 数随档位增长(1 / 2 / 3)
- **任务文档实时落盘**:每 phase 末 Edit 到 `tasks/YYYY-MM-DD-HHMM-<module>-<slug>.md`,意外中断时下次 Claude 读 frontmatter 即可定位恢复
- **docs 漂移自动守门**:Phase 7 从 git diff 提取改动符号,grep `docs/` + README,列"代码改了但 docs 仍是旧描述"清单 → standard/deep 主动同步,light 列清单给用户决定
- **跨任务教训库**:Phase 1 召回 / Phase 7 追加,触发 ≥3 次自动移到「高频坑」段
- **AskUserQuestion 仅 2 必用节点**:Phase 4 选方案 + Phase 6 fix/proceed,其他节点用文字段输出 + 隐式确认,减少打扰
- **零配置可跑**:无 `.work-dev/config.jsonc` 时自动推断(docs/<category>/*.md 文件名 / package.json workspaces / git remote 等),需要时再用 `$schema` 自动补全的 JSONC 显式 override

## 📂 组件

| 组件 | 数量 | 说明 |
|---|---:|---|
| Skill | 1 | description 语义触发 |
| Command | 1 | `/work-dev <complexity> <描述>` 显式触发 |
| Agent | 3 | explorer / architect / reviewer(基于 feature-dev 改写 + work-dev 7 phase 上下文) |
| Template | 2 | `_template.md`(任务文档骨架)+ `_lessons.md`(教训库骨架) |
| Config | 1 | `config/work-dev.example.jsonc` 15 字段全示例 |
| Schema | 1 | `schema/work-dev.schema.json` draft-07 中文注释 |
| Docs | 1 | README.md 11 章节 / 477 行 |

## 🚀 安装

### 从 GitHub Release(推荐)

```bash
claude plugin install https://github.com/lichungang/cc-work-dev
```

### 本地开发

```bash
git clone https://github.com/lichungang/cc-work-dev ~/Downloads/cc-work-dev
claude plugin marketplace add ~/Downloads/cc-work-dev
claude plugin install cc-work-dev@work-dev-local --scope <user|project>
```

`--scope project` 只在当前项目启用,不污染其他项目。

## 📖 文档

- [README.md](./README.md) — 11 章节完整文档(安装 / 快速开始 / 核心概念 / 文档架构规约 / 项目配置 / `/work-dev init` / Agents / FAQ / 项目结构 / 设计依据 / License)
- [config/work-dev.example.jsonc](./config/work-dev.example.jsonc) — 15 字段全示例
- [schema/work-dev.schema.json](./schema/work-dev.schema.json) — JSON Schema(IDE 自动补全)

## 🆚 与 feature-dev 的区别

| 维度 | feature-dev | cc-work-dev |
|---|---|---|
| 流程阶段 | 7 phase 通用 | **7 phase + 强约束**(任务文档实时落盘 / docs 漂移 Phase 7 守门 / lessons 库) |
| 复杂度分档 | 无 | **light / standard / deep 三档** |
| 项目配置 | 无 | **15 字段 `.work-dev/config.jsonc`** |
| 适用场景 | 通用 feature 开发 | **有规模、需文档同步纪律的项目** |

可同时安装,根据任务选用。

## 🙏 致谢

- agent 设计参考 [feature-dev](https://github.com/anthropics/claude-plugins-official) 3 个 agent(code-explorer/architect/reviewer)
- brainstorming + spec 流程:[superpowers](https://github.com/anthropics/skills) 套件
- 完整设计 ADR:[2026-05-20-work-dev-plugin-config-interface](https://github.com/lichungang/work_score/blob/main/docs/decisions/2026-05-20-work-dev-plugin-config-interface.md)

## 📜 License

[MIT](./LICENSE)
