# Game Name Radar

面向游戏 SEO 趋势站的“新游戏名发现工具”。它监控竞争网站 Sitemap、RSS/Atom 和 itch.io 列表，通过历史快照识别新增 URL，提取游戏名称并建立候选队列。

## 核心流程

```text
新游戏名发现
  → Google Trends 7 天 / 30 天验证
  → Google SERP 竞争检查
  → 域名检查
  → 纯 HTML 建站与部署
```

## 功能

- 添加任意竞争站 Sitemap 或 Sitemap Index
- 自动递归扫描子 Sitemap（有限额，避免超时）
- itch.io 最新网页游戏、New & Popular、New Feed、Featured Feed 内置源
- 第一次扫描 Sitemap 仅建立基线，后续只报告新增 URL
- 从标题或 URL Slug 提取游戏名
- 多来源同名游戏自动合并并提升机会分
- 一键打开 Google Trends 7 天、30 天、Google SERP 和域名查询
- 候选状态管理：未处理、准备建站、已建站、忽略
- CSV 导出、JSON 备份与恢复
- GitHub Actions 每小时两次自动扫描并更新 `data/`
- 不依赖数据库；手动数据保存在浏览器 localStorage

## 本地运行

安装 Vercel CLI 后：

```bash
npm install -g vercel
vercel dev
```

只运行自动扫描器：

```bash
npm run scan
```

## 添加竞争站监控源

网页端可以直接添加并手动扫描。自动扫描则修改 `config/sources.json`：

```json
{
  "id": "competitor-example",
  "name": "Example Games Sitemap",
  "url": "https://example.com/game-sitemap.xml",
  "kind": "competitor-sitemap",
  "fetchKind": "sitemap",
  "enabled": true,
  "baselineOnly": true
}
```

首次运行时 `baselineOnly: true` 会只保存当前 URL，不把旧页面当成新游戏；第二次开始才会输出增量。

## GitHub Actions

`.github/workflows/radar.yml` 默认在每小时第 17 分和第 47 分运行。工作流会：

1. 读取 `config/sources.json`
2. 对比 `data/state.json` 中的历史快照
3. 更新 `data/candidates.json`
4. 提交有变化的 `data/` 文件
5. Git 集成开启时触发 Vercel 自动部署

私有仓库会消耗 GitHub Actions 分钟数；不需要自动监控时可以删除 `schedule`，保留手动 `workflow_dispatch`。

## Vercel 部署

1. 在 Vercel 导入 GitHub 仓库
2. Framework Preset 选择 `Other`
3. 不需要 Build Command
4. 不需要环境变量
5. 部署

项目包含 `/api/scan` Node.js Function，用于解决浏览器跨域限制。接口对 URL、内网地址、响应大小、重定向次数和请求时长进行了限制，避免成为开放代理。

## 数据说明

- 网页手动扫描历史：当前浏览器 localStorage
- 自动扫描历史：`data/state.json`
- 自动候选：`data/candidates.json`
- 最近一次自动扫描报告：`data/latest-report.json`

网页启动时会自动读取并合并 `data/candidates.json`。

## 评分逻辑

- itch.io Featured：+5
- itch.io New & Popular：+4
- 竞争站 Sitemap：+3
- itch.io 新作 / Feed：+2
- 同一名称出现在 2 个来源：额外 +4
- 同一名称出现在至少 3 个来源：额外 +7
- 名称长度和词数合理：+2
- 最近 3 天发布：+2

分级：

- 12–20：高机会，立即验证 Trends 和 SERP
- 7–11：待验证
- 0–6：观察

评分只是筛选器，不代替真实的 Google Trends 和 SERP 判断。

## 技能库参考

项目的工作流设计参考了 `kennyzir/7deer_skills` 中的 `html5-game-radar` 思路，并在此基础上实现了可部署界面、Sitemap 增量快照、GitHub Actions 持久化和安全抓取接口。代码为独立实现。

## 合规提醒

本工具只发现公开页面与名称。将第三方游戏 iframe 到独立站之前，应确认开发者授权、嵌入条款、素材使用权限和广告许可。
