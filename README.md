# infra-ops

一套面向**基础设施装维与生产运维全生命周期**的 Agent Skills。

它不是某个产品的安装手册，也不是要求从头走到尾的线性 wizard。套件把证据、授权、可恢复、todo、上下文卫生、状态与 Git 放在同一个共享核心中，再由不同入口塑造初始化、设计、实施、诊断、验证和交接任务。

## 入口

| Skill | 用途 |
|---|---|
| `/infra-ops` | 自动 / 混合模式：持有用户目标，按实况动态组合一个或多个专用入口，可回退，目标达成即停止 |
| `/infra-init` | 新工作区初始化：接机器连接、采最小环境画像、建立目录约定与项目 `AGENTS.md`、状态基线和本地 Git 首提交 |
| `/infra-design` | 装维调研、基础设施规划、方案选型与关键决策确认 |
| `/infra-apply` | 安装部署、升级迁移和生产变更的精确授权实施 |
| `/infra-diagnose` | 故障、异常、性能与数据一致性问题的证据驱动诊断 |
| `/infra-verify` | 装维验收、生产巡检、变更后验证与独立复核 |
| `/infra-handoff` | 状态收敛、窗口收尾、决策记录和可行动交接 |
| `infra-ops-core` | 仅供模型及其他技能调用的共享核心，不作为用户直接入口 |

所有专用入口同时支持模型按 description 自动选择和用户斜杠调用。自动加载技能只加载规则，不会因此获得写授权。

## 使用方式

跨阶段或尚不确定应从哪一步开始时：

```text
/infra-ops 我要把一套新服务从现状调查做到可运维交付，先核实环境，关键方案和我确认，写操作先给精确计划。
```

只需要一个阶段时直接使用专用入口：

```text
/infra-init 这是刚建好的空工作区。机器连接方式：host-a（密钥）、host-b（密码，稍后我配到连接工具里）；先把工作区初始化起来，计划给我看。
```

```text
/infra-diagnose 只做只读诊断。检查工具返回空列表，先证明这个判据有效，再解释原因。
```

```text
/infra-verify 从消费方视角验收这次变更，分别给出通过、失败和未验证项。
```

专用入口不会因为自己的阶段结束就自动扩大任务；只有用户目标原本跨阶段或后来明确扩展时，才升级到 `/infra-ops`。

## 共享约定

- **Todo**：非琐碎任务必须维护工作索引，有专用工具使用工具，没有则维护项目内文件。
- **证据**：统计先划时间窗、分母与操作者；高影响结论验证判据并经独立路径复核。
- **授权**：精确变更计划覆盖明确命令、目标、对象与上限，偏离重新授权。
- **恢复**：agent 只执行低爆炸半径且证明可恢复的写；高风险步骤由人执行。
- **验收**：读取消费方和目标资源的真实值，不把退出码当作生效证明。
- **上下文**：长日志落盘检索、长测量后台运行，子代理返回结论和证据位置。
- **状态与 Git**：项目约定优先，默认管理状态与决策最小集；本地原子提交与远端操作分开授权。
- **过程录制**：改变目标环境状态、长时交互式或被引为证据的命令必须录制；不可用时停写而非降级。
- **工作区骨架**：空白工作区由 `infra-init` 提供四容器默认骨架，项目约定可覆盖，存量只映射不搬迁。

## 目录

```text
skills/
├── infra-ops-core/
│   ├── SKILL.md
│   └── references/
│       ├── evidence.md
│       ├── change-safety.md
│       ├── recording.md
│       └── state-and-git.md
├── infra-init/
│   ├── SKILL.md
│   └── references/skeleton.md
├── infra-ops/SKILL.md
├── infra-design/SKILL.md
├── infra-apply/SKILL.md
├── infra-diagnose/SKILL.md
├── infra-verify/SKILL.md
└── infra-handoff/SKILL.md
tests/scenarios.md
```

核心正文只保存每次都需要的不变量，三份 references 按需加载。专用入口和自动编排入口只保留各自的阶段职责与完成条件。

## 安装

本仓库以 `skills/` 保存开源源码；安装时再将 bundle 放入 DSH 的发现目录。复制时保留八个平级 skill 目录，尤其不要把 `infra-ops-core` 嵌套进其他入口，否则 DSH 不会发现它。

- **源码校验**：直接对根目录 `skills/infra-*` 下的 bundle 运行校验器。
- **项目级安装**：将 `skills/` 下的八个目录放入目标项目的 `.agents/skills/`。
- **DSH 用户级安装**：放入 `~/.dsh/skills/` 或 `~/.agents/skills/`。
- **其他 Agent Skills 客户端**：按客户端的技能目录约定安装同一组标准 `SKILL.md` bundle。

DSH 支持直属 `SKILL.md` 热更新（以所装版本文档为准）；安装或修改后通过实际目录与调用结果确认，不需要用重启代替验证。

## 校验与行为验收

七个普通 bundle 直接使用 [Agent Skills specification](https://agentskills.io/specification) 的参考校验器：

```bash
for skill in skills/infra-{init,apply,design,diagnose,handoff,ops,verify}; do
  skills-ref validate "$skill"
done
```

`infra-ops-core` 使用 DSH 的 `user-invocable: false` 扩展来实现“模型可调用、用户不直接调用”。官方校验器会拒绝这个厂商扩展字段；在临时副本中只移除该行，可校验其余标准字段，实际 M-only 行为则由 DSH 加载验收确认：

```bash
tmp="$(mktemp -d)"
cp -R skills/infra-ops-core "$tmp/"
sed -i '/^user-invocable: false$/d' "$tmp/infra-ops-core/SKILL.md"
skills-ref validate "$tmp/infra-ops-core"
```

这不等于宣称原始核心被官方校验器原生接受。[tests/scenarios.md](tests/scenarios.md) 进一步定义自动路由、反馈回退、核心去重、todo、长日志、证据、授权、Git、初始化和交接场景。行为验收应在无生产副作用的模拟环境或夹具中进行；未实际执行的场景不得标记为通过。结构、DSH 加载、代表性行为和敏感信息检查结果见 [0.1.0 验收记录](tests/validation-2026-11-20.md)与 [0.2.0 验收记录](tests/validation-2026-11-21.md)。

## 范围与许可

当前版本：`0.3.0`。本仓库不包含原始复盘、现场日志或凭据，只保留通用规则和匿名小例。后续真实装维项目用于继续检验任务边界与可恢复性，不预先视作已通过实战验收。

采用 [MIT License](LICENSE)，版权署名 `2026 Yue-bin`。
