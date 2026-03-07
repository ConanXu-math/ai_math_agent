# 基于 OpenCode 的数学研究多智能体平台 — 文档索引

本目录存放平台设计、难点分析与实施规划文档。

| 文档 | 说明 |
|------|------|
| [01-项目难点分析](./01-项目难点分析.md) | 项目主要难点（编排一致性、可复现、长任务可靠性、质量闸门、资源与成本、接口边界）及应对方向 |
| [02-实施步骤与阶段](./02-实施步骤与阶段.md) | 四阶段实施计划：任务合同 → 三个适配器 → 最小编排流 MVP → 治理能力 |
| [03-平台接口列表与调用顺序](./03-平台接口列表与调用顺序.md) | 平台层 API 列表（任务/运行/阶段/产物/审计）及一次完整 run 的调用顺序 |
| [04-存储与目录设计](./04-存储与目录设计.md) | `platform/` 目录布局、artifacts 约定、元数据表结构（tasks / runs / stages / artifacts / events / metrics） |

---

相关资源：

- **OpenCode 接口**：`opencode/specs/project.md`
- **Numina 集成说明**：`numina-lean-agent/README.md`
- **OpenEvolve 使用**：`opencode/openevolve/README.md`、`opencode/openevolve/CLAUDE.md`
