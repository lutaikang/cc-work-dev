---
description: 7 阶段开发工作流 — 任务文档实时落盘、docs 漂移自动守门、项目配置化
argument-hint: [light|standard|deep] <feature/bug 描述>
---

# /work-dev v3 — 通用 7 阶段开发工作流

接到需求:$ARGUMENTS

> **配套**(从 `.work-dev/config.jsonc` 读,无则自动推断):
> - 任务文档目录:`{tasks_root}/`(默认 `tasks/`)
> - 任务模板:`{template_path}`(默认 `{tasks_root}/_template.md`,无则用 plugin `templates/_template.md`)
> - 教训库:`{lessons_path}`(默认 `{tasks_root}/_lessons.md`,无则用 plugin `templates/_lessons.md`)
> - 项目规则:`{project_root}/CLAUDE.md`(若存在)
>
> 冲突仲裁:与项目 CLAUDE.md 冲突时按 CLAUDE.md 优先级裁定。
> 配置全字段:参见 plugin 内 `config/work-dev.example.jsonc` 与 `schema/work-dev.schema.json`。

---

## 0. 子命令路由

**入口分支(必先判断,优先于复杂度解析)**:

- 若 `$ARGUMENTS` trim 后**首 token == `init`**(可后跟空白或行尾) → 跳到 §11 `/work-dev init` 子命令流程执行,执行完毕即退出,**不进入下方 7 phase**。
- 否则继续 §1,按 `light` / `standard` / `deep` 解析复杂度并走 Phase 1-7。

`init` 之外的子命令暂未保留;若首 token 非 `light` / `standard` / `deep` / `init`,视作复杂度未指定,直接把整个 `$ARGUMENTS` 当任务描述,复杂度走 `{config.default_complexity}`(默认 `standard`)。

---

## 1. 全局纪律

1. **复杂度手动指定**:`/work-dev light <描述>` / `/work-dev standard <描述>` / `/work-dev deep <描述>`。未指定时走 `{config.default_complexity}`(默认 `standard`)。三档含义详见 plugin README。
2. **阶段顺序**:严格 Phase 1 → 2 → 3 → 4 → 5 → 6 → 7;light 也按 7 phase 走,每阶段裁剪。
3. **AskUserQuestion 必用节点**:仅 Phase 4 末(选方案)+ Phase 6 末(fix/proceed),共 2 次。其他节点用文字段输出 + 隐式确认。
4. **任务文档按 phase 实时落盘**:每个 Phase 完成立刻 Edit 到任务文档对应章节,防中断丢失。
5. **不在 Phase 5 中途 commit**:所有改动留 working tree,Phase 7 一次 commit 收口(例外见 Phase 5 步骤 4)。
6. **docs 同步由 Phase 7 守门员负责**:Phase 1-6 不强制检查 docs,Phase 7 自动扫描漂移并同步。
7. **高风险操作识别**:对照项目 CLAUDE.md 高风险清单 + `{config.high_risk_ops_extra}`,Phase 2 影响面分析时显式标注。
8. **不臆造**:引用行号前 `grep -n` 校准,不发明 API / 函数名 / 行号。
9. **agent 数随复杂度**:explorer / architect / reviewer 三类,light 各 1 / standard 各 2 / deep 各 3(可被 `{config.agent_counts.<complexity>}` override)。
10. **中断恢复**:用户回话简短(如"继续")时,先读最新 `{tasks_root}/*.md` 的 frontmatter `phase` + `status` 定位阶段。
11. **工作流标识行**:每次回复**第一行**输出 `[work-dev v3 / <complexity> / Phase N]`(例:`[work-dev v3 / standard / Phase 4]`)。当复杂度走的是 `{config.default_complexity}` 兜底(用户未显式指定)时,在档位后追加 ` (default)` 提示,如 `[work-dev v3 / standard (default) / Phase 1]`。Phase 7 commit 后到下次启动前不再输出。

---

## 2. Phase 1 — Discovery

**目标**:理解需求 + 召回历史相似坑 + 落盘任务文档草稿。

**步骤**:

1. **复述需求**:用我的话 "你想 <X>,为了 <Y>,约束 <Z>";直接输出文字段,用户未反驳即视为确认。
2. **类型 + 复杂度 + 模块归属**:
   - 类型:bug / 新功能 / 重构 / 性能 / 文档 / 配置
   - 复杂度:用户显式优先;未指定走 `{config.default_complexity}` 或 `standard`
   - 模块归属:对照 `{config.modules.<category>}` 白名单;留空则自动推断 `{docs_root}/<category>/*.md` 文件名;新模块走 §10 strict_modules 分支
3. **召回历史相似坑**:grep `{lessons_path}` 的「高频坑」+「模块:<当前模块>」+「跨模块通用」三段,输出命中清单 N 条。
4. **创建任务文档草稿**:
   - 路径:`{tasks_root}/YYYY-MM-DD-HHMM-<module>-<slug>.md`(HHMM 本地 24h)
   - 拷贝 `{template_path}` 内容
   - 写入第 1 节(问题陈述)+ frontmatter(`status: draft` / `phase: 1` / `complexity` / `created`)
   - 历史坑暂存,Phase 2 详细列入第 2 节

---

## 3. Phase 2 — Codebase Exploration

**目标**:用 explorer 摸清代码 + 给出根因 + 影响面,落盘第 2/3/4 节。

**步骤**:

1. **定位代码**:README 路由树 → `{docs_root}/<category>/<module>.md` 核心代码文件清单 → 深读清单文件。**禁止全仓 grep**。
2. **启动 explorer 并行**(数量按 §1 第 9 条):
   - light:1 个(综合分析:实现 + 调用链 + 边界)
   - standard:2 个(A 实现 + 调用链;B 集成点 + 跨端一致性)
   - deep:3 个(A 类似功能 + 历史;B 架构 + 调用链;C 集成 + 边界 + 数据兼容)
   - 每个 agent 返回 5-10 个关键文件 + 总结;Claude 整合后深读
3. **根因分析 5-Why**:表象 → 为什么 → 为什么 → 根因。标注问题类型(产品定义模糊 / 设计缺陷 / 实现 bug / 性能瓶颈 / 数据问题)。**light 可省**。
4. **影响面分析 5 维**(代码消费方 / 数据兼容 / 性能 / 跨端一致性 / 回滚成本)各 1 句话。deep 可补 1-2 证据源(代码 grep / git log / docs 实采)。**不做 Pre-mortem**。
5. **高风险标注**:对照项目 CLAUDE.md 高风险清单 + `{config.high_risk_ops_extra}`,列出本任务涉及项(无则写"无")。
6. **落盘**:第 2/3/4 节 + frontmatter `phase: 2` / `last-updated`。

---

## 4. Phase 3 — Clarifying Questions

**目标**:只问真正阻断的硬伤,落盘第 5 节。

**步骤**:

1. **识别澄清点**:Phase 2 影响面的不确定点 / explorer 并列问题或对称缺口 / 高风险细节 / 历史坑分歧。
2. **发问**:AskUserQuestion 弹枚举(≤4 题/轮,>4 拆多轮);无下限题数(light 可能 0 题,直接跳过本 Phase);不强制凑数。
3. **落盘**:Q&A 列表写第 5 节,frontmatter `phase: 3`。
4. **回流影响**:若澄清答案改变根因或影响面判断,回 Phase 2 步骤 3/4 更新第 2/3 节;**不直接进 Phase 4**。

---

## 5. Phase 4 — Architecture Design

**目标**:用 architect 出方案 + 落盘任务文档完整内容 + AskUserQuestion 批准。

**步骤**:

1. **启动 architect 并行**(数量按 §1 第 9 条):
   - light:1 个(直接出方案,sole architect → full mode)
   - standard:2 个(A minimal-diff + B refactor-with-feature,compact mode:仅 §2/§4/§9)
   - deep:3 个(A minimal-diff + B refactor-with-feature + C defer-tech-debt-only,compact mode)
   - compact mode 用于节约多 agent 并行上下文;用户选定方案后,主流程再让获选 architect 以 `mode: full` 补出 §6/§7/§8 进任务文档
   - 每个 agent 返回完整 blueprint(或 compact 三块)+ 必读文件清单(5-10 个)
2. **对比 + 推荐**:输出方案对比表(方案 / 关键决策 / 优 / 劣)+ 推荐项 + 理由(复杂度 vs 价值 / 一致性 / 维护成本 / 紧急程度)。只有 1 个合理方案时显式说"评估了 X/Y/Z 都不行,只 A 合理,理由..."。
3. **落盘任务文档完整内容**:第 6/7/8 节 + frontmatter `phase: 4` / `status: confirmed`。**必须在 AskUserQuestion 之前落盘**,确保中断时磁盘有完整方案。
4. **AskUserQuestion(必用节点 1/2)**:`开干 A` / `开干 B` / `改方案` / `取消`
   - 开干 → frontmatter `status: in-progress`,进 Phase 5
   - 改方案 → 回 Phase 3 或 Phase 4 步骤 1,**Edit 覆盖更新第 5/6/7/8 节**(SSOT,不开派生文档)
   - 取消 → `status: cancelled`,第 10 节追加理由,流程结束

---

## 6. Phase 5 — Implementation

**目标**:按拆解步骤实施,不动任务文档第 7 节(待 Phase 6/7),不 commit。

**步骤**:

1. **准备**:重读 Phase 2/4 agent 返回的必读文件清单;第 7 节拆解 >3 步时用 `TaskCreate` 建运行时 todo。
2. **实施**:逐项做;改代码即 `TaskUpdate` 标 completed;**不中途 commit**(例外见步骤 4);**不实时改第 7 节**(等 Phase 6/7 一并处理)。
3. **边实施边核对**:引用行号前 `grep -n` 校准;改公开行为时回看第 8 节验收清单;历史坑命中条目严格对照。
4. **中途 commit 例外**:仅以下场景允许 — 用户显式"边做边 commit" / hotfix 紧急 / 跨多端"分块 commit" / 长任务中断风险高。触发时第 10 节追加 commit 摘要(不强制 hash,git log 是真相)。
5. **结束自检**:第 7 节全 `[x]` 或 `[!]`(留 PR),无 `[ ]` 未做;第 8 节可勾(留 Phase 6 跑测后确认);frontmatter `phase: 5` / `last-updated`。

---

## 7. Phase 6 — Quality Review

**目标**:跑测试 + 多 agent review + fix-then-recheck 直到全绿。

**步骤**:

1. **跑测试**:按 `{config.test_commands}` 的 key 自动选(按改动文件路径前缀匹配 `{code_roots[i]}/<key>/`);无 config 时按 `package.json scripts` / `pyproject.toml` 推断。必须先跑,有红才修。
2. **启动 reviewer 并行**(数量按 §1 第 9 条):
   - light:1 个(综合 review)
   - standard:2 个(A bugs + 边界 + 错误处理 + 安全;B 项目规范一致性)
   - deep:3 个(A simplicity + DRY + elegance;B bugs + 边界 + 安全;C 项目规范一致性)
3. **合并 findings**:按高 / 中 / 低排序去重;对照第 8 节验收清单逐项勾(✅ / ⚠ / ❌)。
4. **fix-then-recheck 回路**:
   - AskUserQuestion(必用节点 2/2):`fix now` / `fix later` / `proceed`
   - `fix now` → 修 → 重跑测试 → 改动 >20 行或高风险时重起一轮 reviewer → 再 AskUserQuestion → 直到 proceed
   - `fix later` → 第 10 节追加 TODO,进 Phase 7
   - `proceed` → 进 Phase 7
5. **落盘第 9 节**:测试结果摘要 / multi-agent findings / 验收勾选;frontmatter `phase: 6` / `status: reviewing` / `last-updated`。

---

## 8. Phase 7 — Summary & Wrap

**目标**:docs 漂移自动守门 + 1 个 commit 收口 + 复盘提炼到 lessons。

**步骤**:

1. **docs 漂移扫描(自动守门员)**:从 `git diff` 提取本任务改动的代码符号(函数名 / 类名 / 常量名 / 配置项 / API 路径 / 数据库字段)。

   **符号过滤规则**(防误报 + 防注入):
   - 长度 ≥3 字符,丢弃 `is` / `id` / `fn` 之类过短词
   - 只保留正则 `^[A-Za-z_][A-Za-z0-9_]*$` 命名的标识符(以及 `/api/...` 形式的 API 路径整段当作字面串)
   - 含 shell 特殊字符(`;` / `$` / `` ` `` / `&` / `|` / `<` / `>` / 反斜杠 / 引号)→ 跳过

   对每个保留下来的符号执行(注意 `--` 终止 flag 解析,`-F` 走字面串,`-w` 单词边界防子串误命中):
   ```
   grep -rnwF -- "<symbol>" {docs_root}/ README.md {config.drift_scan_extra_targets[]}
   ```
   列出"代码改了但 docs 仍是旧描述"的位置清单。
2. **同步 docs**:
   - `standard` / `deep`:Claude 主动 Edit 同步,不再问
   - `light`:输出清单给用户决定
   - 同步完后再次 `grep -rnwF --` 校验,应为空或仅命中历史归档段
3. **复盘提炼**:默认 1-2 句话摘要写入第 11 节"摘要";若用户喊 `retro` 走完整 6 题(4 枚举 + 2 叙述)。无论哪种,提炼 1-2 条规则:
   - 检查 `{lessons_path}` 是否已有同类条目
   - 已有 → 触发次数 +1;触发次数 ≥3 自动移到顶部「高频坑」段
   - 无 → 追加到对应模块段(`### 通用陷阱` / `### bug 修复类` / `### 死代码清理类` / `### Phase 流程` 等)
   - 格式:`- <一句话规则>(来源: <task 文件>;触发次数: 1)`
   - 第 11 节"追加到 lessons 的条目"小节回填实际写入的规则
4. **落盘第 10/11 节**:第 10 节(变更记录,本次一次性写完,不含 commit hash);第 11 节(摘要 / 完整复盘 + lessons 追加条目);frontmatter `phase: 7` / `status: done` / `last-updated`。
5. **一次 commit 收口**:打包 1 个 commit,含代码 + 测试 + docs 同步(步骤 2 结果)+ 任务文档(第 1-11 节最终态)+ lessons 更新。commit message 概括整个 PR 范围,**不再做 hash 回填 commit**。`{config.auto_commit: false}` 时只 staged 不 commit,等用户审批。
6. **任务归档建议**:若 `{tasks_root}/` 根目录 `status: done` 任务 ≥ 30,主动建议归档到 `{tasks_root}/done/YYYY-MM/`(按 created 月份分桶),并更新该月任务文档内相对链接。

---

## 9. 中途升级与降级

- **升级**:实施中发现复杂度被低估 → 停下,AskUserQuestion(升级 / 保持 / 取消);若升级,frontmatter `complexity` 改新值,回 Phase 2 / 4 补做对应内容,第 10 节追加升级理由。
- **降级**:Phase 2 explorer 实测后骨架预期失真(典型:DB 列实际不存在)→ 直接降复杂度,frontmatter 改新值,第 10 节追加降级理由。

禁止"当 light 做完了再说"。

---

## 10. 模块归属(strict_modules)

**白名单来源**:`{config.modules.<category>}` 显式填 → override;留空 → 自动推断 `{docs_root}/<category>/*.md` 文件名(去后缀)。

**接到任务描述时**:

1. 关键词比对当前白名单
2. 命中 → 用该模块
3. 不命中:
   - `{config.strict_modules: true}` → 抛错:"新模块 `<X>` 不在白名单。请显式编辑 `.work-dev/config.jsonc` modules 字段或建 `{docs_root}/<category>/<X>.md` 后重试"
   - `{config.strict_modules: false}`(默认)→ AskUserQuestion 询问"新模块 `<X>`,是否纳入 `<category>` 类别?",用户确认则 mkdir + 建占位 `{docs_root}/<category>/<X>.md`,白名单立即包含

---

## 11. `/work-dev init` 子命令

新项目接入时一行命令完成骨架。`/work-dev init` 行为:

1. 检测项目根有无 `README.md` / `{docs_root}/` / `{tasks_root}/`,无则交互式问后创建
2. 交互式问技术栈、模块白名单(填进 `.work-dev/config.jsonc`)
3. 拷贝 plugin 内置 `templates/_template.md` `templates/_lessons.md` 到 `{tasks_root}/`
4. 生成 `{docs_root}/{features,stack,data,ops,decisions}/.gitkeep`
5. 在 README.md 注入「## 文档索引」段(指向 `{docs_root}/`)
6. 在 CLAUDE.md(无则创建)注入「## 文档同步规则」段(从规约渲染)
7. 在 CLAUDE.md 注入「## 高风险操作清单」段(默认 5 条 + 用户补)

执行完毕后,该项目即可走完整 7 phase 流程。

---

## 12. 转 /feature-dev

`deep` 任务且(≥3 模块 / 架构调整 / 用户措辞含"探索/调研/重构")→ 主动建议切 `/feature-dev`(若已装 `feature-dev` plugin)。

切换时本任务文档 `status: cancelled`,feature-dev 完成后回本流程整合结果。
