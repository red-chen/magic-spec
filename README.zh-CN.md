# Magic Spec

**面向 AI 编程代理的规范驱动开发技能集。**

[English](./README.md) | 中文 | [日本語](./README.ja.md)

通过结构化工作流将模糊想法变为交付功能：探索 → 提案 → 实现 → 审查。每一步都可追溯、可测试、可自省。

```
 探索             提案                实现              审查
┌─────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│ 想法     │───▶│ proposal.md  │───▶│ RED-GREEN-   │───▶│ 代码审查     │
│ 分析     │    │ specs/       │    │ REFACTOR     │    │ 安全审计     │
│ 调研     │    │ design.md    │    │ 自省         │    │              │
│          │    │ test-design  │    │ 验证         │    │              │
│          │    │ tasks.md     │    │ 归档         │    │              │
└─────────┘    └──────────────┘    └──────────────┘    └──────────────┘
```

## 为什么选择规范驱动？

AI 编程代理能力强大，但缺乏结构时会产出不一致的结果——跳过测试、遗忘需求、偏离设计、在没有证据的情况下声称"已完成"。

magic-spec 通过 **6 个技能** 在每个阶段强制执行纪律：

| 问题 | magic-spec 的解决方案 |
|------|---------------------|
| 盲目触碰陌生或遗留代码 | `/magic-codebase-recon` 在任何改动前完成架构考古、热点分析、爆炸半径评估与脆弱性评分 |
| 构建了错误的东西 | `/magic-explore` 在承诺之前先调研 |
| 需求模糊 | `/magic-proposal` 创建带 Given/When/Then 场景的正式规范 |
| 跳过测试 | `/magic-proposal` 在实现之前设计测试 |
| 偏离计划 | `/magic-apply` 每个任务后对照 design.md 自省 |
| "应该能用"式断言 | `/magic-apply` 要求每个勾选框都有新鲜的验证证据 |
| 遗漏安全问题 | `/magic-security-review` 运行 12 阶段 CSO 级审计 |
| 浅层代码审查 | `/magic-code-review` 调度并行专家审查员 |

## 技能一览

### `/magic-codebase-recon` — 代码库侦察与安全变更情报

在触碰陌生或遗留代码库之前，先完成全面侦察。最终输出 `brief.md`，覆盖五个分析维度：

- **架构考古** — 分层结构、集成点、历史架构迁移，以及从 git 历史和源码注释中挖掘的显性技术债
- **Git 热点分析** — 高频变更文件、隐式协同变更耦合、Bug 吸引器、休眠模块
- **依赖追踪与爆炸半径评估** — 扇入/扇出分析、循环依赖检测，以及任意目标模块的三层爆炸半径（直接 → 传递 → 运行时）
- **脆弱性评分** — 基于变更频率、耦合度、爆炸半径、测试覆盖缺口的加权公式，输出五档风险等级（🟢 稳定 至 ⛔ 极危）
- **安全变更策略** — 按风险排序的变更顺序，以及针对每个高危模块的策略建议（绞杀者模式、特征化测试、隔离层、功能开关、并行运行、渐进抽取）

只读模式。所有分析结果写入 `brief.md`，后续改动通过 `/magic-proposal` 和 `/magic-apply` 完成。

### `/magic-explore` — 深度探索与发现

在写任何代码之前，先调研想法、分析代码、比较方案或追踪根因。

**4 种探索模式：**
- **想法探索** — 将模糊想法转化为清晰理解
- **代码库调查** — 梳理架构、追踪数据流、发现模式
- **根因分析** — 基于证据链和假设检验的系统性调试
- **方案比较** — 用具体的权衡对比 2-3 个选项

产出洞察，而非工件。当理解结晶时，流转到 `/magic-proposal`。

### `/magic-proposal` — 结构化变更提案

将理解转化为包含 5 个工件的完整变更提案：

```
magic-spec/changes/<变更名称>/
├── proposal.md        为什么和做什么 — 范围、能力、影响
├── specs/             什么变了 — 增量规范（ADDED/MODIFIED/REMOVED）
├── design.md          怎么做 — 架构决策、关键文件、权衡
├── test-design.md     如何验证 — 规范到测试的映射、测试用例
└── tasks.md           步骤 — 有序的实现清单
```

**核心特性：**
- **增量规范（Delta Specs）** — 描述变更而非整个系统（适用于存量项目）
- **Given/When/Then 场景** — 每个需求都有可测试的场景
- **规范到测试的映射** — 每个需求在写代码之前就映射到具体的测试用例
- **依赖图** — 工件按顺序创建：proposal → specs ←→ design → test-design → tasks

### `/magic-apply` — 规范驱动的实现

以 TDD 纪律和持续自省执行任务：

```
对每个任务：
  1. 读取任务上下文 + test-design.md
  2. 编写失败的测试（RED）
  3. 最小化实现（GREEN）
  4. 重构（REFACTOR）
  5. 用证据验证
  6. 自省：设计合规？规范满足？测试覆盖？
  7. 标记完成（仅在自省通过后）
```

**自省不是可选的。** 每个任务之后，代理停下来检查：
- 这与 design.md 匹配吗？
- 所有规范场景都被测试覆盖了吗？
- test-design.md 中的测试用例是否真正实现了？
- 我在偷工减料吗？

所有任务完成后，**最终自省** 在归档前审计整个变更。

### `/magic-security-review` — 安全态势审计

CSO 级别的自动化安全审计，包含 12 个阶段：

| 阶段 | 检查内容 |
|------|---------|
| 0 | 架构心智模型 + 技术栈检测 |
| 1 | 攻击面普查 |
| 2 | 密钥考古（git 历史） |
| 3 | 依赖供应链 |
| 4 | CI/CD 管道安全 |
| 5 | 基础设施影子面 |
| 6 | Webhook 与集成审计 |
| 7 | LLM & AI 安全 |
| 8 | OWASP Top 10 |
| 9 | STRIDE 威胁建模 |
| 10 | 假阳性过滤 + 主动验证 |
| 11 | 数据分类 |

**两种模式：** 日常模式（8/10 置信度，零噪音）和全面模式（2/10 置信度，月度深度扫描）。

### `/magic-code-review` — 多专家代码审查

两阶段审查 + 并行专家分析：

**阶段 1：规范合规** — 我们是否构建了所要求的？逐项对照规范验证。

**阶段 2：代码质量** — 构建的东西是否精良？架构、测试、性能、安全、可维护性。

**专家组**（并行）：测试专家、安全专家、性能专家、可维护性专家 — 发现交叉验证并去重。

## 安装（Claude Code）

### 1. 克隆仓库

```bash
git clone https://github.com/anthropics/magic-spec.git
```

### 2. 将技能添加到项目

将技能复制或符号链接到项目的 `.claude/skills/` 目录：

```bash
# 在项目根目录下
mkdir -p .claude/skills

# 方式 A：符号链接（推荐 — 拉取时自动更新）
ln -s /path/to/magic-spec/skills/magic-codebase-recon .claude/skills/magic-codebase-recon
ln -s /path/to/magic-spec/skills/magic-explore .claude/skills/magic-explore
ln -s /path/to/magic-spec/skills/magic-proposal .claude/skills/magic-proposal
ln -s /path/to/magic-spec/skills/magic-apply .claude/skills/magic-apply
ln -s /path/to/magic-spec/skills/magic-security-review .claude/skills/magic-security-review
ln -s /path/to/magic-spec/skills/magic-code-review .claude/skills/magic-code-review

# 方式 B：复制
cp -r /path/to/magic-spec/skills/* .claude/skills/
```

### 3. 在 CLAUDE.md 中添加技能路由（推荐）

在项目的 `CLAUDE.md` 中添加以下内容以启用自动技能调用：

```markdown
## Skill routing

当用户的请求匹配可用技能时，将其作为首要操作调用：

- 首次接触代码库、遗留系统改造、"动这个会炸什么" → 调用 magic-codebase-recon
- 功能创意、头脑风暴、"X 是怎么工作的" → 调用 magic-explore
- "构建 X"、"添加功能"、"制定计划" → 调用 magic-proposal
- "实现"、"执行计划"、"开始编码" → 调用 magic-apply
- 安全审计、"检查漏洞" → 调用 magic-security-review
- 代码审查、"审查我的更改" → 调用 magic-code-review
```

## 使用示例（Claude Code）

### 示例 1：新功能 — 完整工作流

```
你: 我想给应用添加深色模式

Claude: [调用 /magic-explore]
  → 探索当前样式系统、主题架构、用户偏好存储
  → 提出 3 种方案及权衡
  → 推荐 CSS 自定义属性方案
  → 置信度：8/10

你: 看起来不错，就这么做

Claude: [调用 /magic-proposal add-dark-mode]
  → 创建 magic-spec/changes/add-dark-mode/
  → 编写 proposal.md（为什么要深色模式，改了什么）
  → 编写 specs/theme-switching.md（ADDED 需求及场景）
  → 编写 design.md（CSS 自定义属性、ThemeProvider、localStorage）
  → 编写 test-design.md（8 个测试用例映射到需求）
  → 编写 tasks.md（4 组 14 个任务）
  → "请在继续之前审查这些工件。"

你: 批准了，开始构建

Claude: [调用 /magic-apply add-dark-mode]
  → 任务 1.1：编写 ThemeProvider 的失败测试 → RED
  → 任务 1.1：实现 ThemeProvider → GREEN
  → 任务 1.1：重构 → 清理
  → 任务 1.1：自省 → 设计合规 ✓，规范覆盖 ✓
  → 任务 1.2：编写 toggle 的失败测试...（继续）
  → ...
  → 最终自省：全部 14 个任务已验证，设计匹配 ✓，规范覆盖 ✓
  → 完成

你: 合并前审查一下代码

Claude: [调用 /magic-code-review --spec add-dark-mode]
  → 阶段 1：规范合规 — 所有需求满足 ✓
  → 阶段 2：代码质量 — 1 个重要问题（缺少错误边界），2 个次要问题
  → 裁决：需要修复 — 1 个重要问题
```

### 示例 2：快速安全检查

```
你: /magic-security-review --diff

Claude: [对当前分支变更运行 12 阶段审计]
  → 阶段 0：检测到 Next.js + TypeScript 技术栈
  → 阶段 2：git 历史中无密钥泄露 ✓
  → 阶段 7：发现用户输入流入系统提示词（严重，9/10）
  → 阶段 8：认证端点缺少速率限制（高危，8/10）
  → 2 个发现，0 个假阳性
```

### 示例 3：先调查再修复

```
你: 最近一次部署后登录页面挂了

Claude: [以根因分析模式调用 /magic-explore]
  → 阶段 1：读取错误日志，追踪认证流程
  → 阶段 2：找到可工作的示例，与出问题的代码比较
  → 阶段 3：假设 — 依赖更新中会话令牌格式改变了
  → 验证：@auth/core 5.x 更改了令牌序列化方式
  → 置信度：9/10，准备提出修复方案

你: 修复它

Claude: [调用 /magic-proposal fix-auth-token]
  → 创建定向提案，1 个规范，3 个任务
  → test-design 将回归测试映射到具体的失败点
  ...
```

## 架构

### 变更生命周期

```
规划                        实现                           完成
────                        ────                           ────
/magic-explore              /magic-apply                   /magic-code-review
     │                           │                              │
     ▼                           ▼                              ▼
/magic-proposal             逐任务循环：                    规范合规
     │                      RED → GREEN →                  代码质量
     ▼                      REFACTOR →                     专家组
 proposal.md                验证 →                              │
 specs/                     自省                                ▼
 design.md                       │                         发现 + 修复
 test-design.md                  ▼
 tasks.md                   最终自省
                                 │
                                 ▼
                            归档至
                            changes/archive/
```

### 增量规范（Delta Specs）

与描述整个系统的传统规范不同，magic-spec 使用**增量规范** — 只描述变化的部分：

```markdown
## ADDED 需求
之前不存在的新行为。

## MODIFIED 需求
变更的行为（包含完整的更新后行为，而非仅是差异）。

## REMOVED 需求
正在废弃的行为（包含迁移指南）。
```

这使得规范在真实项目中切实可用 — 你总是在改变既有系统，而非从零开始描述。

### 自省机制

magic-apply 包含两个层次的内省：

**逐任务自省**（每个任务完成后）：
```
任务 1.1 自省
  设计合规：  ✓ 匹配 design.md 决策
  规范满足：  ✓ 所有场景已覆盖
  测试覆盖：  ✓ 所有 test-design.md 用例已编写
  置信度：    8/10
  → 标记完成
```

**最终自省**（所有任务完成后）：
```
最终自省：add-dark-mode
═══════════════════════
任务：          14/14（全部验证：是）
设计匹配：      ✓
规范覆盖：      ✓
测试覆盖：      ✓
回归：          ✓ 无
总体：          完成
```

## 贡献

1. Fork 本仓库
2. 创建功能分支
3. 使用 magic-spec 自身的技能来提案和实现变更
4. 提交 PR

## 许可证

MIT
