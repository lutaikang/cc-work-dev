# cc-work-dev

通用 **7 阶段开发工作流** Claude Code plugin — 任务文档实时落盘 / docs 漂移自动守门 / light · standard · deep 三档复杂度 / 项目配置化。

> 基于一个内部开发工作流抽象而来:从 385 行项目定制流程整理为 214 行通用命令 + 15 字段项目配置。

## 一句话定位

接到代码任务时,把 Claude 拉进一个**强约束的 7 阶段流程**(Discovery → Explore → Clarify → Design → Implement → Review → Wrap),沿途自动产出可恢复的任务文档、跨任务的教训库、Phase 7 docs 漂移扫描,**防中断丢失 + 防 docs 漂移 + 防"看起来做完了实际没接通"**。

适合:有一定规模的代码仓(monorepo / 多端 / 多模块),需要一致的开发流程纪律和文档同步守门员。

---

## 1. 安装

### 方式 A:GitHub Release(推荐,v0.1.0 已发布)

```bash
claude plugin install https://github.com/lutaikang/cc-work-dev
```

### 方式 B:本地 link(开发阶段 / 调试)

```bash
# <local-path> 自选,如 ~/dev、~/code 等
git clone https://github.com/lutaikang/cc-work-dev <local-path>/cc-work-dev
ln -s <local-path>/cc-work-dev ~/.claude/plugins/local/cc-work-dev
# 重启 Claude Code 加载
```

安装完成后:

```bash
claude plugin list | grep work-dev
# 应输出:cc-work-dev (✓ Connected)
```

并且工具列表中应包含 `/work-dev` 斜杠命令。

---

## 2. 快速开始(零配置)

**最简体验**:任何项目装上 plugin 后,直接输入命令即可,不需要任何前置配置。

```
/work-dev light 修复 auth 模块的 token 失效后未跳转登录页的 bug
```

Claude 会:

1. **复述需求** + 复杂度采用 + 模块归属(自动推断:扫 `docs/features/*.md` 文件名)
2. **召回历史相似坑**(grep `tasks/_lessons.md` 三段,命中即提示)
3. **创建任务文档** `tasks/YYYY-MM-DD-HHMM-auth-fix-token-redirect.md`
4. **启动 explorer agent** 摸代码、出根因、出影响面
5. **(可能跳过 Phase 3 澄清,light 无下限题数)**
6. **启动 architect agent** 出方案 → AskUserQuestion 让你选
7. **实施** + 不中途 commit
8. **启动 reviewer agent** 跑测试 + findings → AskUserQuestion `fix-now/later/proceed`
9. **Phase 7**:docs 漂移扫描 → 同步 → 一次 commit 收口 → 复盘提炼到 `_lessons.md`

**全程零配置**。如果你的项目目录结构标准(`docs/` `tasks/` `src/` 或 `packages/`),自动推断会算出合理默认值。

如果默认值不够,可以创建 `.work-dev/config.jsonc` 显式 override(详见 §5)。

---

## 3. 核心概念

### 3.1 7 阶段工作流

| Phase | 名称 | 主要工作 | 产出 |
|---|---|---|---|
| 1 | Discovery | 复述需求 / 模块识别 / 召回历史坑 | 任务文档草稿 |
| 2 | Explore | explorer agent 摸代码 / 5-Why 根因 / 5 维影响面 | §2/§3/§4 |
| 3 | Clarify | 用 AskUserQuestion 问真正阻断的硬伤 | §5 |
| 4 | Design | architect agent 出方案 + AskUserQuestion 批准(**必用节点 1/2**) | §6/§7/§8 |
| 5 | Implement | 按拆解步骤实施,不动任务文档第 7 节,不 commit | (代码) |
| 6 | Review | 跑测试 + reviewer agent + AskUserQuestion(**必用节点 2/2**) | §9 |
| 7 | Wrap | docs 漂移自动扫描 + 同步 + 1 个 commit 收口 + 复盘提炼到 lessons | §10/§11 |

**关键纪律**:Phase 1-6 不强制检查 docs,**Phase 7 守门员自动扫描漂移并同步**;改 X 类代码 → 同步 X 对应 docs(features / stack / data / ops / decisions)。

### 3.2 三档复杂度

| 档 | 触发条件 | 任务文档节数 | agent 数 |
|---|---|---:|---|
| **light** | <10 行 / 单文件 / typo / 重命名 | 4 (§1/4/7/8;§11 可选) | explorer 1 / architect 1 / reviewer 1 |
| **standard**(默认) | 多文件 / 新功能 / 公开行为变更 | 9 (§1-8 + §11 三行复盘) | 各 2 |
| **deep** | DB / 认证 / 支付 / 跨 ≥3 端 / 删除文件表字段 | 11 (§1-11) | 各 3 |

用户显式指定优先(`/work-dev <complexity> <描述>`);不指定走 `{config.default_complexity}`(默认 `standard`)。

### 3.3 任务文档实时落盘

每个 Phase 完成立刻 Edit 到任务文档对应章节,frontmatter 同步更新 `phase` / `status` / `last-updated`。**意外中断时,下次 Claude 读 frontmatter 即可定位阶段恢复**。

任务文档命名:`{tasks_root}/YYYY-MM-DD-HHMM-<module>-<slug>.md`(HHMM 本地 24h)。

**SSOT 原则**:不为同一任务另开 `v2 / fix / final / debug_log` 派生文档,而是在原文件「变更记录」追加 / 章节覆盖更新。

### 3.4 跨任务教训库

`{lessons_path}` (默认 `{tasks_root}/_lessons.md`)结构:

```
## 高频坑(任何任务都先看,触发次数 ≥3)
## 模块:<module-name>
  ### 通用陷阱 / bug 修复类 / 死代码清理类
## 跨模块通用
  ### Phase 流程 / Phase 5 实施期 / Phase 6 测试覆盖
```

- Phase 1 自动 grep 召回命中条目,写入任务文档第 2 节
- Phase 7 按**沉淀判定四问**决定是否追加,每任务**最多 1 条**新规则(凑不出就 0 条)
- 触发次数 ≥3 自动移到顶部「高频坑」段
- 条目格式:`- <祈使句一句话规则>(来源: <task 文件>;触发次数: N)`

**沉淀判定四问**(全部 yes 才进库,任一 no 跳过):

1. **跨任务共性?** — 同类问题在别的任务也会出现?(项目特异 → 写 docs/,不进 lessons)
2. **可重现?** — 能预期下次某场景必触发?(纯偶发 → 不沉淀)
3. **可操作?** — 写成"做 X / 不做 X"祈使句?(只能描述现象 → 不沉淀)
4. **非已有?** — 库中无同类?(已有 → 仅触发次数 +1)

**任务复盘也对应精简**:§11 默认仅 3 行(问题 / 方案 / 沉淀?),用户喊 `retro` 才展开完整复盘(也仅 3 题)。详见 `templates/_template.md`。

### 3.5 docs 漂移自动守门

Phase 7 从 `git diff` 提取改动符号(函数 / 类 / 常量 / 配置项 / API 路径 / DB 字段),先按规则过滤(长度 ≥3、仅合法标识符 / API 路径 allowlist、非标识符字符自动排除),再对每个符号执行:

```bash
grep -rnwF -- "<symbol>" {docs_root}/ README.md {drift_scan_extra_targets[]}
```

`-F` 走字面匹配、`-w` 单词边界、`--` 终止 flag 解析,既防短词误报也保证 grep 参数始终是字面合法标识符。

列出"代码改了但 docs 仍是旧描述"的位置:
- `standard` / `deep`:Claude 主动 Edit 同步,不再问
- `light`:输出清单给用户决定(改动小,人工核更快)

同步完后再次 grep 校验,应为空或仅命中历史归档段。

---

## 4. 文档架构规约

plugin 假定项目用以下「三层文档结构」(任意一层缺失都能自动推断):

```
project-root/
├── README.md                     # L1 入口:架构图 + 技术栈 + 文档索引
├── docs/                         # L2 模块文档(5 类规约固定)
│   ├── features/                 # 业务/功能模块,改业务必读
│   │   ├── auth.md
│   │   └── ...
│   ├── stack/                    # 技术栈/包/进程层
│   │   ├── backend.md
│   │   └── ...
│   ├── data/                     # 数据 schema / API
│   │   ├── database.md
│   │   └── api-reference.md
│   ├── ops/                      # 部署 / 环境变量
│   │   └── deployment.md
│   └── decisions/                # 重大架构决策 ADR
│       └── <YYYY-MM-DD-slug>.md
├── tasks/                        # L3 流程文档
│   ├── _template.md              # 任务文档模板
│   ├── _lessons.md               # 跨任务教训库
│   └── YYYY-MM-DD-HHMM-*.md     # 具体任务
└── .work-dev/config.jsonc        # plugin 项目配置(可选)
```

**类别 → 同步触发表**(改 X 类代码 → 同步 X 对应 docs):

| 改动类型 | 同步目标 |
|---|---|
| 业务/功能代码 | `docs/features/<module>.md` |
| 技术栈/层结构 | `docs/stack/<layer>.md` |
| DB schema / API 端点 | `docs/data/database.md` / `docs/data/api-reference.md` |
| 部署 / 环境变量 | `docs/ops/deployment.md` |
| 重大架构决策 | 新建 `docs/decisions/<YYYY-MM-DD-slug>.md` |

项目可在 `extra_categories` 声明自定义类别(如 `docs/runbooks/`、`docs/research/`)。

---

## 5. 项目配置 `.work-dev/config.jsonc`

**所有字段可选**,留空走自动推断。需要 override 时创建 `.work-dev/config.jsonc`,加 `$schema` 引用获得 IDE 自动补全:

```jsonc
{
  "$schema": "https://raw.githubusercontent.com/lutaikang/cc-work-dev/main/schema/work-dev.schema.json",
  "project_name": "My Project",
  "test_commands": {
    "backend": "cd packages/backend && pytest",
    "web":     "cd packages/web && npm test"
  },
  "high_risk_ops_extra": ["微信 jscode2session"]
}
```

### 5.1 15 字段全表

| 字段 | 类型 | 默认 | 说明 |
|---|---|---|---|
| `docs_root` | string | `"docs/"` | 文档根目录 |
| `tasks_root` | string | `"tasks/"` | 任务文档根 |
| `code_roots` | string[] | `["src/"]` | 代码根;monorepo 填 `["packages/"]` |
| `modules.feature` | string[] | `[]` (自动推断) | feature 类别白名单 |
| `modules.stack` | string[] | `[]` | stack 类别白名单 |
| `modules.data` | string[] | `[]` | data 类别白名单 |
| `modules.ops` | string[] | `[]` | ops 类别白名单 |
| `modules.decisions` | string[] | `[]` | decisions ADR 条目白名单(非任务模块,仅 drift scan 与新建 ADR 路径校验) |
| `strict_modules` | boolean | `false` | true=新模块抛错;false=询问+建占位 |
| `extra_categories` | object | `{}` | 自定义类别(规约 5 类外) |
| `high_risk_ops_extra` | string[] | `[]` | 项目特有高风险变更(追加规约默认 5;字段名保留 `ops_extra` 兼容历史) |
| `project_name` | string | (从 git 推断) | 工作流标识 / 任务文档 frontmatter |
| `default_complexity` | enum | `"standard"` | `light` / `standard` / `deep` |
| `test_commands` | object | `{}` (推断) | Phase 6 验证命令,key 按路径前缀匹配 |
| `agent_counts` | object | (各档默认) | Override agent 数量 |
| `drift_scan_extra_targets` | string[] | `[]` | drift scan 额外目标(默认扫 `docs_root` + `README.md`) |
| `auto_commit` | boolean | `true` | false=Phase 7 只 staged 不 commit |
| `template_path` | string | `<tasks_root>/_template.md` | 模板路径 override |
| `lessons_path` | string | `<tasks_root>/_lessons.md` | lessons 路径 override |

完整示例见 [`config/work-dev.example.jsonc`](./config/work-dev.example.jsonc)。

### 5.2 自动推断规则表(M3.1)

零配置场景下,plugin 按以下规则探测项目状态:

| 字段 | 推断算法 | 兜底 |
|---|---|---|
| `docs_root` | 探测存在性:`docs/` → `documentation/` → `doc/` | `docs/`(`/work-dev init` 时建) |
| `tasks_root` | 探测:`tasks/` → `task/` → `worklog/` | `tasks/`(同上) |
| `code_roots` | 读 `package.json` workspaces / `pyproject.toml` packages / `Cargo.toml` workspace.members / `go.work` | `["src/"]` 或 `["lib/"]` |
| `modules.<category>` | `ls $docs_root/<category>/*.md` 文件名去 `.md` 后缀 | 空白名单(strict_modules=false 时新模块自动询问) |
| `extra_categories` | 扫 `$docs_root/*/` 子目录,不在规约 5 类的 → 询问用户是否纳入 | 不主动加 |
| `project_name` | `git config --get remote.origin.url` → 解析 repo 名 → `basename $PWD` | 目录名 |
| `default_complexity` | 无推断 | `"standard"` |
| `test_commands` | `package.json` scripts 找 `test` / `pyproject.toml` 找 pytest 配置 / `Cargo.toml` 找 `[[bin]]` | 空 dict,Phase 6 跳过测试(并提示 init) |
| `agent_counts` | 无推断,按规约表 | light=1/1/1, standard=2/2/2, deep=3/3/3 |
| `drift_scan_extra_targets` | 探测 `wiki/` / `ARCHITECTURE.md` / `CHANGELOG.md` → 询问用户是否纳入 | 空(只扫 `docs_root` + README) |
| `auto_commit` | 无推断 | `true` |
| `template_path` / `lessons_path` | 探测项目内对应文件 → 用项目的;否则用 plugin `templates/` | plugin 内置 |
| `high_risk_ops_extra` | 无推断 | 空(只用规约默认 5 条) |
| `strict_modules` | 无推断 | `false` |

**规约默认 5 条高风险变更**(`high_risk_ops_extra` 之外的基线):

1. 删除文件 / 目录 / 数据库表
2. 数据库迁移(生产环境)
3. 修改 / 部署生产配置
4. 修改权限 / 认证流程(JWT / RBAC)
5. 涉及支付 / 计费 / 配额

### 5.3 模块归属(strict_modules)

接到任务描述时:

1. 关键词比对当前白名单(`{config.modules.<category>}` 显式填的 ∪ 自动推断的)
2. 命中 → 用该模块
3. 不命中:
   - `strict_modules: true` → 抛错:`新模块 <X> 不在白名单。请显式编辑 .work-dev/config.jsonc modules 字段或建 {docs_root}/<category>/<X>.md 后重试`
   - `strict_modules: false`(默认)→ AskUserQuestion 询问"新模块 `<X>`,是否纳入 `<category>` 类别?",用户确认则:
     - `mkdir -p {docs_root}/<category>/`
     - `touch {docs_root}/<category>/<X>.md` 写入占位骨架
     - 白名单立刻包含,继续 Phase 1 流程

---

## 6. `/work-dev init` 子命令

新项目接入时一行命令完成骨架:

```
/work-dev init
```

行为:

1. 检测项目根有无 `README.md` / `{docs_root}/` / `{tasks_root}/`,无则交互式问后创建
2. 交互式问技术栈、模块白名单(填进 `.work-dev/config.jsonc`)
3. 拷贝 plugin 内置 `templates/_template.md` `templates/_lessons.md` 到 `{tasks_root}/`(若项目内已有同名文件 → 询问保留 / 覆盖 / diff)
4. 生成 `{docs_root}/{features,stack,data,ops,decisions}/.gitkeep`
5. 在 README.md 追加「## 文档索引」段(指向 `{docs_root}/`)
6. 在 CLAUDE.md(无则创建)追加「## 文档同步规则」段(从规约渲染)
7. 在 CLAUDE.md 追加「## 高风险变更清单」段(默认 5 条 + 用户补)

执行完毕后,该项目即可走完整 7 phase 流程。

---

## 7. Agents 说明

plugin 自带 3 个 agent,职责对齐 feature-dev 但针对 work-dev 7 phase 上下文改写。

### 7.1 `work-dev-explorer`(Phase 2)

**模型**:sonnet · **颜色**:yellow

输出**直接喂任务文档** §2/3/4:

1. 5-10 个核心代码文件清单(供主流程 Phase 5 深读)
2. 5-Why 根因链初稿(light 可省)
3. 5 维影响面分析(代码消费方 / 数据兼容 / 性能 / 跨端一致性 / 回滚成本)
4. 高风险标注(对照 CLAUDE.md + `high_risk_ops_extra`)
5. 历史相似坑命中清单(若主流程提供 `lessons_path` 召回)

**纪律**:`grep -n` 校准的文件:行号,不发明 API / 函数名 / 行号。

### 7.2 `work-dev-architect`(Phase 4)

**模型**:sonnet · **颜色**:green

输出**直接喂任务文档** §6/7/8:

1. Patterns & Conventions Found(发现的现有模式)
2. Architecture Decision(本架构师的决断,一段话)
3. Component Design(组件分解)
4. Implementation Map(创建 / 修改 / 删除文件清单,**含 caller 文件**)
5. Data Flow(数据流图)
6. Build Sequence(供任务文档 §7 拆解步骤)
7. Acceptance Criteria(供 §8 验收清单)
8. Essential Files for Phase 5(5-10 个 implementer 必读)
9. Risk & Trade-offs

**多 architect 并行差异化**:standard=2 / deep=3 时,每个 agent 被分配焦点角度(minimal-diff / refactor-with-feature / defer-tech-debt-only),不允许输出混合方案。并行场景下默认 compact mode(仅输出 §2 Decision + §4 Implementation Map + §9 Risk),用户选定后再以 full mode 补 §6/§7/§8。

### 7.3 `work-dev-reviewer`(Phase 6)

**模型**:sonnet · **颜色**:red

输出**直接喂任务文档** §9:

1. Reviewing 摘要(diff 范围 + 焦点角度)
2. Issues(高 / 中按 confidence ≥ 80 报,sorted by severity)
3. 验收清单逐项勾(✅ / ⚠ / ❌,对照 §8)
4. Recommendation(`proceed` / `fix-now` / `fix-later`)

**反直觉意图信号支持**:主流程 prompt 可前置 "Context for reviewer" 声明"我们故意 NOT 做 X 因为 Y",reviewer 会尊重这些 explicit decisions,不把它当 violation 报。

**Confidence ≥ 80 才报**,低于此阈值的疑虑不污染报告。

---

## 8. FAQ

### Q1. 我项目没有 `docs/` 目录,能用吗?

能。零配置场景下:

- 改业务代码时 plugin 会询问"创建 `docs/features/<X>.md` 吗?"`
- 走 `/work-dev init` 一次性建好骨架

### Q2. 我项目用 `documentation/` 不用 `docs/`,怎么办?

`.work-dev/config.jsonc` 写:

```jsonc
{ "docs_root": "documentation/" }
```

所有 plugin 行为(grep / drift scan / 模块识别)都基于该路径。

### Q3. 我有多个测试命令(backend pytest + web vitest + mobile jest),Phase 6 怎么选?

`test_commands` 写多 key,Phase 6 按改动文件路径前缀**自动选**:

```jsonc
{
  "code_roots": ["packages/"],
  "test_commands": {
    "backend":  "cd packages/backend && pytest",
    "web":      "cd packages/web && npm test",
    "mobile":   "cd packages/mobile && npm test"
  }
}
```

改 `packages/backend/x.py` → 触发 `backend` key;改 `packages/web/y.tsx` → 触发 `web` key。多 key 都被触发时全部跑。

### Q4. 我不想 plugin 自动 commit,只想 Phase 7 stage 给我看,我自己 commit。

```jsonc
{ "auto_commit": false }
```

Phase 7 走完所有 docs 同步和任务文档更新,但 commit 那一步停下,你审完自己 commit。

### Q5. 任务做到一半发现复杂度估错了,能改吗?

能。中途升级 / 降级:

- 实施中发现低估 → 停下,AskUserQuestion(升级 / 保持 / 取消);升级后 frontmatter `complexity` 改新值,回 Phase 2/4 补做对应内容
- Phase 2 explorer 实测后骨架预期失真(典型:DB 列实际不存在) → 直接降复杂度

详见 `commands/work-dev.md` §9。

### Q6. 我描述任务后,Claude 怎么知道该走哪个模块?

按以下顺序判断模块归属:

1. 关键词比对 `{config.modules.<category>}` 显式白名单(优先)
2. 关键词比对 `{docs_root}/<category>/*.md` 自动推断的白名单
3. 都不命中:
   - `strict_modules=true` 抛错
   - `strict_modules=false`(默认)询问用户

新模块走流程见 §5.3。

### Q7. 与 feature-dev plugin 的区别?

| 维度 | feature-dev | work-dev |
|---|---|---|
| 流程阶段 | 7 phase(discovery → explore → clarify → design → implement → review → done) | 7 phase(同名但带额外约束) |
| 任务文档 | 不强制落盘 | **强制实时落盘**(每 phase 末 Edit) |
| 复杂度分档 | 无 | **light / standard / deep 三档** |
| docs 同步 | 不强制 | **Phase 7 自动守门员扫描** |
| lessons 库 | 无 | **跨任务召回 + 自动追加** |
| 项目配置 | 无 | **`.work-dev/config.jsonc` 15 字段** |
| AskUserQuestion 节点 | 多次 | **仅 2 次(Phase 4 选方案 + Phase 6 fix/proceed)** |
| 适用场景 | 通用 feature 开发 | 有规模、需文档同步纪律的项目 |

可同时安装,根据任务选用(work-dev `deep` 任务大型重构 / 跨 ≥3 端时,可主动 fallback 到 `/feature-dev`)。

### Q8. lessons 库越来越大,怎么管理?

`tasks/_lessons.md` 在严格沉淀四问下通常稳定在 50-150 行(项目跑半年以上才到 200+)。机制保护:

- **入库门槛高**:Phase 7 沉淀判定四问(共性 / 可重现 / 可操作 / 非已有),每任务最多 1 条,通常 0-1 条
- 触发次数 ≥3 自动移到「高频坑」段(每次任务先看)
- 模块段内按类型分子段(`### 通用陷阱` / `### bug 修复类` / `### 死代码清理类`)
- 跨模块段按 Phase 流程分(`### Phase 流程` / `### Phase 5 实施期` / `### Phase 6 测试覆盖`)
- 用户可定期手动归档(如按年份拆 `_lessons-2026.md`)

### Q9. 我装了双 plugin(work-dev + feature-dev),命令同名 / agent 同名怎么办?

不会冲突:

- 命令:`/work-dev` 来自本 plugin,`/feature-dev` 来自 feature-dev,各走各的
- agent 名:本 plugin 用 `work-dev-{explorer,architect,reviewer}` 前缀,feature-dev 用 `code-{explorer,architect,reviewer}`,Claude Code 通过 frontmatter `name` 字段区分

---

## 9. 项目结构

```
cc-work-dev/
├── .claude-plugin/plugin.json     # name / version / description / author
├── commands/work-dev.md           # 7 phase 主流程
├── skills/work-dev/SKILL.md       # description 语义触发入口
├── agents/
│   ├── work-dev-explorer.md       # Phase 2 探查 agent
│   ├── work-dev-architect.md      # Phase 4 设计 agent
│   └── work-dev-reviewer.md       # Phase 6 review agent
├── templates/
│   ├── _template.md               # 任务文档模板(/work-dev init 拷贝)
│   └── _lessons.md                # lessons 库空骨架
├── config/work-dev.example.jsonc  # 15 字段全示例
├── schema/work-dev.schema.json    # JSON Schema(IDE 自动补全)
├── README.md                      # 本文件
└── LICENSE                        # MIT
```

---

## 10. 设计依据

完整设计 ADR(brainstorming 7 题对齐输出)将在后续版本拷贝到本 repo `docs/decisions/`。

主要决策:

1. **数据与规则分离** — 配置独立文件,CLAUDE.md 只留人读的纪律
2. **`.work-dev/config.jsonc`** — JSONC 格式,15 字段,独立目录便于扩展
3. **可选 + 静默** — 零配置可跑,`/work-dev init` 主动调用持久化
4. **两层 override** — plugin 默认 + 项目级
5. **方案 4 目录结构** — 原计划 4 类资源 + agents/ + 前缀 + 命令 ≤ 300 行
6. **6 大 work-dev 原则** — 7 phase / AskUserQuestion 2 节点 / 任务文档实时落盘 / Phase 7 守门员 / 复杂度 3 档 / agent 数量随复杂度,**全部保留不可配**

---

## 11. License

[MIT](./LICENSE)

Copyright (c) 2026 lichungang
