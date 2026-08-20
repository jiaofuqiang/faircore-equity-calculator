# FairCore AI — 股权计算器

> 免费在线创业团队股权分配工具（Slicing Pie 2.0）

## 🔗 在线地址

**https://jiaofuqiang.github.io/faircore-equity-calculator/**

## ✨ 功能

- Slicing Pie 2.0 动态股权模型：工时×\×4x + 现金×3x + IP×2x
- 多成员增删 + 实时股权分配条形图
- 邮箱捕获（localStorage，可对接后端）
- SEO 优化（title/description/og/canonical）

## 🚀 本地运行

`bash
python -m http.server 8899
`

访问 http://localhost:8899

## 🧮 计算模型

| 贡献类型 | 计算方式 | 风险倍数 |
|---|---|---|
| 时间 | 工时 × \/小时 × 4x | 4x（高风险）|
| 现金 | 金额 × 3x | 3x |
| IP/资源 | 价值 × 2x | 2x |

示例：Alice 100h → 28.6% / Bob 200h → 71.4%

## 📦 部署

纯静态单文件，支持 Vercel / Netlify / GitHub Pages 一键部署。

<!-- DELIVERED: 2026-08-20 -->
