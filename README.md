# D20 · Vibe Design

WaytoAGI 在 D20 全球设计院长峰会使用的动态网页演示稿。

## 主题

**Vibe Design：AI 时代的 Web Design 新范式**

从设计资产、社区内容与 Design.md 出发，展示网页、活动、OPC 产品、SEO 与商业流程如何进入同一套设计实践。

## 使用

直接打开 `index.html`。

- `→` / `Space`：下一页
- `←`：上一页
- `Home`：第一页
- `End`：最后一页

演示包含自动播放的网页录屏与产品视频，建议使用最新版 Chrome，并允许视频自动播放。

## 在线地址

[www.waytoagi.com/d20/](https://www.waytoagi.com/d20/)

旧域名 `http(s)://d20.waytoagi.com/` 重定向到新路径，历史资源路径与查询参数继续保留。

## 部署

项目无需构建，由 [waytoagi-static-pages](https://github.com/waytoagi-team/waytoagi-static-pages) 的 `mounts.yaml` 固定本仓库 commit，发布到统一 EdgeOne Pages。视频自动进入共享 OSS 媒体通道。

页面变更合入本仓库后，合并统一仓库对应的版本更新 PR 才会发布；旧 OSS 自动发布已关闭。更新和回退步骤见 [DEPLOYMENT.md](./DEPLOYMENT.md)。
