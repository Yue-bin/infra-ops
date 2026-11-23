# infra-ops 0.3.0 验收记录 · 2026-11-23

## 范围

`0.3.0` 新增过程录制约定（`infra-ops-core/references/recording.md`）及四个入口的接入、`infra-init` 的 Git/证据策略询问与骨架 `evidence/` 容器。行为测试仍在 ignored 夹具进行，未连接真实机器。

## 1. 结构校验

七个普通 bundle 原生 `skills-ref` 通过；`infra-ops-core` 临时副本（仅移除 `user-invocable: false`）通过，分层口径不变。八入口版本均为 `0.3.0`，全部文件权限 `644`（`recording.md` 初建为 `600`，按新规则当场修正）。

## 2. 录制语法实测（本机 util-linux 2.41.5）

| 用例 | 命令 | 结果 |
|---|---|---|
| 阳性对照 | `script -q -e -c 'echo probe; exit 3' f.log` | `wrapper_rc=3`，退出码透传；文件含 `Script started` 头与正文 |
| 阴性对照 | `script -q -c 'exit 3' f.log` | `wrapper_rc=0`，退出码丢失 |

结论：`-e` 为强制项的依据成立；`-q` 不抑制文件头尾，与文档描述一致。

## 3. 未执行项

- H 段十二项录制行为场景已定义，本轮**未逐条跑子代理验收**（仅完成语法与结构实测）。
- 真实项目中的录制落地、证据索引实际填写、非 Linux 平台等价手段登记均未验证。
- 全局安装与远端推送未在本轮执行。

诚实结论：本次交付的是**规则与结构**，行为面待下一个真实项目检验。
