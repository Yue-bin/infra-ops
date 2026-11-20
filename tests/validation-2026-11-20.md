# infra-ops 验收记录 · 2026-11-20

## 范围与基线

- 验收基线：提交 `b3fe0cd` 及此前源码；本文件只记录实际执行结果。
- 结构源码位于根目录 `skills/`；DSH 验收使用 ignored 的临时 `.agents/skills/` 安装夹具，不把部署目录重新纳入源码。
- 行为测试均在无生产连接的模拟上下文或隔离夹具中进行，未对真实系统执行写操作。
- 本记录覆盖代表性场景，不等于 `tests/scenarios.md` 全部条目已逐条执行，也不等于已经通过真实装维项目试点。

## 1. Agent Skills 标准字段

工具：官方 `agentskills/skills-ref`，包版本 `0.1.0`，隔离安装在 ignored 的临时 venv 中。

| 对象 | 实际结果 |
|---|---|
| `infra-apply` / `infra-design` / `infra-diagnose` / `infra-handoff` / `infra-ops` / `infra-verify` | 六个原始 bundle 均返回 `Valid skill` |
| `infra-ops-core` 原始文件 | 官方校验器仅因 `user-invocable` 不在标准字段表而拒绝；错误日志无其他问题 |
| 临时核心副本仅移除 `user-invocable: false` | 返回 `Valid skill`，证明其余标准字段通过 |

核心保留 DSH M-only 扩展，不能宣称原始七份文件均被官方校验器原生接受；采用“普通入口原生校验 + 核心标准字段分层校验 + DSH 行为实测”的口径。

## 2. DSH 真实发现与加载

环境：DSH `0.2.2-rc.1`。临时安装夹具初始化自己的 `.git`，使 `projectRoot` 正确落在夹具而非父仓库。

| 验收项 | 实际结果 |
|---|---|
| 模型 skill 目录 | 七个名称全部出现，description 分别对应共享核心、自动编排与五个专用阶段 |
| 精确 `skill` 调用 | 七个均成功返回匹配的 `<skill_content>` 和 `<skill_resources>` |
| `resourceBase` | 七个均指向临时安装夹具中各自的 bundle 目录 |
| 核心 references | `evidence.md`、`change-safety.md`、`state-and-git.md` 均从核心 `resourceBase` 成功读取 |
| 调用面 | 核心 `user-invocable:false`；其余六个使用默认模型/用户可调用策略，并经当前 DSH 解析器语义核对 |
| 正文规模 | 七个 `SKILL.md` 为 30–73 行，均低于 500 行建议上限 |

直接 `/infra-diagnose`、`/infra-apply`、`/infra-handoff`、`/infra-ops` 的用户注入在行为子会话中实际发生；未逐一操作 Web `/` 菜单。核心的用户隐藏结论来自 frontmatter 与当前 DSH 解析器/注册表语义，不冒充 GUI 点击测试。

## 3. 代表性行为场景

行为子代理均要求不读取 `tests/scenarios.md`，以减少把预期答案直接复述出来的偏差。

| 场景 | 实际观察 | 结论 |
|---|---|---|
| 自动装维起步 | 依次加载 `infra-ops`、核心、`infra-design`，建立 todo，只停在现状访问与目标决策点；未加载全部入口、未进入写实施 | 通过 |
| 空输出诊断 | 加载 `infra-diagnose`、核心与证据 reference；拒绝把空列表当成对象不存在，要求已知存在样本的阳性对照和独立读取路径 | 通过 |
| 未授权与跨主机实施 | 加载 `infra-apply`、核心与变更安全 reference；建立 todo，停在精确计划和授权门，未把 host-a 的许可扩展到 host-b，未声称执行写 | 通过 |
| 20,000 行长日志 | 使用全文关键词检索定位，再只读取 17 行格式与上下文；找到第 13,742 行的唯一模拟错误及请求标识，没有全文灌入上下文 | 通过 |
| 专用入口目标扩展 | 原诊断上下文只新增加载 `infra-ops`，核心未重复加载，原 todo 保留；在事实不足时继续诊断而非提前实施 | 通过 |
| 一次性只读问题 | `/infra-ops` 与核心加载一次，不创建 todo、不加载无关入口、不建文件或 Git，回答后立即停止 | 通过 |
| 状态交接与 Git | 加载 `infra-handoff`、核心与状态 reference；沿用唯一状态文件并补证据路径和下一步；本地提交 `bcb28ee docs: 补齐交接证据与下一步`，`git ls-files` 只有 `CURRENT-STATE.md`，原始 evidence 保持未跟踪，无远端或 push | 通过 |

## 4. 敏感信息与 Git 历史

工具：`gitleaks 8.30.1`，使用官方 release 的 Linux x64 包及公布 SHA-256 校验。

- 先用仅符合 GitHub Token 格式的高熵假样本做阳性对照，确认扫描器报告 1 项；该样本不是有效凭据。
- 对验收基线的 Git 历史执行 `gitleaks git`：扫描 11 个提交、约 48.5 KB，发现 0 项。
- 检查全部历史路径，未出现 `复盘-*`、`important_view*` 或 `.dsh/memory/`。
- 白名单正负对照确认：`skills/`、`tests/`、README 与 LICENSE 可入库；复盘、checkpoint、记忆归档、临时工具与验证夹具保持 ignored。

## 5. 尚未覆盖

- 真实装维项目的端到端实战试点。
- `tests/scenarios.md` 中未列入本记录的其余场景。
- 逐一点击 Web 用户菜单，以及其他 Agent Skills 客户端的实际安装。
- 官方校验器对原始核心 DSH 扩展的接受；这是有意保留的兼容性边界，不是待修复缺陷。
