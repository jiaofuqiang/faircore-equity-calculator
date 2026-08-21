# FairCore AI — 安全审计报告（v1.0 初版）

**版本**：v1.0 | **日期**：2026-08-20 | **状态**：🟡 初版（基于代码审查，正式渗透测试待 Month 3）

> 本报告基于对当前代码库（faircore_api / faircore_web / contracts / blockchain）的静态审查。
> 覆盖：OWASP Top 10 映射、资产清单、已知风险、缓解措施、安全测试基线。

---

## 1. 审计范围

| 组件 | 位置 | 状态 |
|---|---|---|
| 后端 API | faircore_api/app/main.py（19 端点） | 🟡 已审查 |
| 数据层 | faircore_api/app/db.py（SQLite） | 🟡 已审查 |
| 前端原型 | faircore_deliverables/*（30 原型） | 🟢 演示性质 |
| 前端生产 | faircore_web/app/*（Next.js） | 🟡 已审查 |
| 智能合约 | faircore_api/contracts/*.sol（5 合约） | 🟡 已审查 |
| 区块链客户端 | faircore_api/blockchain/client.py | 🟡 已审查 |
| CI/CD | faircore-github-action/.github/workflows/ | 🟢 已审查 |

## 2. 资产清单与敏感数据

| 资产 | 敏感度 | 存储位置 |
|---|---|---|
| 成员个人信息（姓名/邮箱/钱包） | 中 | SQLite / 生产 PG |
| 贡献记录 | 中 | SQLite / PG + 链上 |
| 股权结构 | 高 | 链上（公开）+ PG |
| API 密钥 | 高 | .env / 环境变量 |
| 部署私钥 | 极高 | 环境变量（Amoy 部署时） |
| 争议/仲裁记录 | 中 | PG disputes 表 |

## 3. OWASP Top 10 映射

| # | 风险 | 当前状态 | 缓解措施 |
|---|---|---|---|
| A01 | 访问控制失效 | 🟡 部分 | 生产需加 JWT + 项目级 RBAC（当前演示无鉴权） |
| A02 | 加密失败 | 🟢 良好 | 链上 keccak256 哈希 + SQLite 本地无敏感传输 |
| A03 | 注入 | 🟢 良好 | 全部 SQL 参数化（sqlite3 占位符）；无字符串拼接 |
| A04 | 不安全设计 | 🟡 关注 | FRL 治理已约束 AI 权限（只读输入→只写裁决） |
| A05 | 安全配置错误 | 🟡 关注 | CORS 需白名单；debug 模式生产必须关 |
| A06 | 易受攻击组件 | 🟡 关注 | Node/Python 依赖需定期升级；web3 7.16/hardhat 2.22 已新 |
| A07 | 认证失败 | 🔴 缺失 | 生产需接入 Auth0/自建 JWT；当前无认证 |
| A08 | 软件和数据完整性 | 🟢 良好 | 链上存证 + 内容 SHA-256 签名（菜谱包类） |
| A09 | 日志和监控失败 | 🟡 部分 | PG audit_log 表已有设计；需加日志聚合 |
| A10 | SSRF | 🟢 良好 | 无外部 URL 抓取功能 |

## 4. 智能合约安全审查

| 合约 | 风险点 | 状态 |
|---|---|---|
| EquityLedger.sol | 仅 owner 可 submitEvaluation；equityBpsOf 无重入风险 | 🟢 良好 |
| VestingVault.sol | `fund()` 需 transferFrom 成功才能 claim（防未注资领取）；claim 仅 beneficiary | 🟢 良好 |
| USDCSettlement.sol | fund/settle 均 onlyOwner | 🟢 良好 |
| FairCoreFactory.sol | deployProject 无重入；addVesting 事件已发 | 🟢 良好 |

**合约审计结论**：逻辑简单无重入/整数溢出（Solidity 0.8+ 自动检查）。正式审计需第三方（如 Trail of Bits / CertiK）——已列入 Month 3 预算。

## 5. 已知风险清单

| # | 风险 | 等级 | 缓解 |
|---|---|---|---|
| R1 | 无认证/授权（生产前必须） | 🔴 高 | 接入 JWT + 项目级 RBAC（Sprint 1 待办） |
| R2 | API 密钥硬编码风险 | 🟠 中 | 已用 .env；密钥轮换策略需建立 |
| R3 | 部署私钥泄露 | 🟠 中 | 仅部署机持有；建议硬件钱包/MPC |
| R4 | 依赖漏洞 | 🟠 中 | 每月 npm/pip audit |
| R5 | 数据备份缺失 | 🟠 中 | SQLite 每日备份 + PG 自动备份 |
| R6 | CORS 过宽 | 🟡 低 | 生产白名单 localhost:8899/3100 |
| R7 | LLM 提示注入（AI 仲裁） | 🟡 低 | 争议输入仅作数据不执行；输出 JSON schema 校验 |

## 6. 安全测试基线（已执行）

| 测试 | 结果 |
|---|---|
| SQL 注入（参数化验证） | ✅ 无拼接 |
| API 输入校验（422 错误处理） | ✅ FastAPI 自动 |
| 合约部署重放 | ✅ 幂等 deployProject |
| 链上数据一致性 | ✅ keccak256 与 Solidity 一致 |
| 权限降级路径（无 web3 时） | ✅ not_configured/degraded |

## 7. 建议路线（Month 3 正式审计）

- [ ] 第三方渗透测试（OWASP 全量）
- [ ] 智能合约正式审计（CertiK/Trail of Bits）
- [ ] Auth0/自建 JWT 接入
- [ ] 安全日志 + 告警（audit_log → 聚合）
- [ ] 密钥轮换与 MPC 方案

---

## 免责声明

本报告为基于静态代码审查的初版，不构成正式安全认证。
正式安全审计报告将在 Month 3 由第三方安全机构出具。

<!-- DELIVERED: 2026-08-20 -->
