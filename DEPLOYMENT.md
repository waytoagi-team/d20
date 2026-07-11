# D20 静态站部署

这个项目不需要构建。生产发布内容只有：

```text
index.html
assets/
```

仓库内的 GitHub Actions 工作流会将这两部分同步到阿里云 OSS。首次手动部署会检查完整 `assets/`，之后的 `push` 只上传本次 Git 变更过的资源。它默认不会自动部署：只有完成环境配置并将 `OSS_DEPLOY_ENABLED` 设为 `true` 后，推送到 `main` 才会触发生产发布。

## 当前环境

根据现有 DNS 和 OSS 响应，当前线上环境可使用以下配置：

| 名称 | 建议值 |
| --- | --- |
| `OSS_BUCKET` | `waytoagi-d20` |
| `OSS_REGION` | `cn-beijing` |
| `OSS_ENDPOINT` | `https://oss-cn-beijing.aliyuncs.com` |
| `OSS_ADDRESSING_STYLE` | `virtual` |
| `OSS_ORIGIN_URL` | `https://waytoagi-d20.oss-cn-beijing.aliyuncs.com` |

如果控制台显示的 Bucket 或地域不同，以控制台为准。

## 1. 创建最小权限的 RAM 身份

不要使用阿里云主账号 AccessKey。为部署创建独立 RAM 用户，并将下面策略中的 `waytoagi-d20` 替换成实际 Bucket 名称：

```json
{
  "Version": "1",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "oss:ListObjects",
      "Resource": "acs:oss:*:*:waytoagi-d20"
    },
    {
      "Effect": "Allow",
      "Action": "oss:PutObject",
      "Resource": [
        "acs:oss:*:*:waytoagi-d20/index.html",
        "acs:oss:*:*:waytoagi-d20/assets/*"
      ]
    }
  ]
}
```

工作流不会删除 OSS 对象，因此不需要 `oss:DeleteObject`。废弃资源可在开启 OSS 版本控制后另行清理。

## 2. 配置 GitHub Environment

进入仓库的 **Settings → Environments → New environment**，创建 `production` 环境。

添加 Environment secrets：

| Secret | 说明 |
| --- | --- |
| `OSS_ACCESS_KEY_ID` | RAM 用户 AccessKey ID |
| `OSS_ACCESS_KEY_SECRET` | RAM 用户 AccessKey Secret |

添加 Environment variables：

| Variable | 初始值 | 说明 |
| --- | --- | --- |
| `OSS_BUCKET` | `waytoagi-d20` | OSS Bucket 名称 |
| `OSS_REGION` | `cn-beijing` | OSS 地域 ID |
| `OSS_ENDPOINT` | `https://oss-cn-beijing.aliyuncs.com` | OSS API Endpoint；必须使用 HTTPS |
| `OSS_ADDRESSING_STYLE` | `virtual` | 默认 Endpoint 使用 `virtual`；自定义 CNAME 使用 `cname` |
| `OSS_ORIGIN_URL` | `https://waytoagi-d20.oss-cn-beijing.aliyuncs.com` | 可选；用于发布后校验源站文件大小 |

另外在 **Settings → Secrets and variables → Actions → Variables** 添加仓库级变量：

| Variable | 初始值 | 说明 |
| --- | --- | --- |
| `OSS_DEPLOY_ENABLED` | `false` | 是否允许 `main` 自动部署；必须是仓库级变量，供任务启动前判断 |

建议为 `production` 环境设置 Required reviewers，避免未经确认的生产发布。

如果阿里云账号受中国内地 Bucket 数据 API 新策略限制，需要准备一个直连 OSS 且配置了 HTTPS 证书的上传 CNAME，并将 `OSS_ENDPOINT` 改成该域名、`OSS_ADDRESSING_STYLE` 改为 `cname`。不要把经过 EdgeOne/CDN 的公开站点域名用作上传 Endpoint。

## 3. 第一次部署

1. 在 **Actions → Deploy static site to OSS → Run workflow** 中运行工作流。
2. 第一次保持 `dry_run=true`，检查将要上传的对象。
3. 确认无误后再次运行，设置 `dry_run=false`。
4. 在 EdgeOne/CDN 中刷新 `/`、`/index.html` 和本次有变更的资源 URL。
5. 检查 `https://d20.waytoagi.com/`，确认页面和视频正常。
6. 最后将 `OSS_DEPLOY_ENABLED` 改为 `true`，开启 `main` 自动发布。

## OSS 和 CDN 设置

- OSS 静态网站默认首页设为 `index.html`。
- `index.html` 使用 `no-cache, no-store, must-revalidate`。
- `assets/` 当前使用一天浏览器缓存。资源文件名不是内容哈希，不要设置 `immutable`。
- 保留 MP4/WebM 的 Range 请求能力，否则视频跳转和渐进播放会受影响。
- EdgeOne/CDN 不应对 `index.html` 设置长时间边缘缓存；每次真实部署后仍应执行缓存刷新。

## 回滚

建议先为 Bucket 开启版本控制。出现问题时，恢复 `index.html` 和受影响资源的上一版本，然后刷新 EdgeOne/CDN 缓存。工作流不会删除任何 OSS 对象；不再被页面引用的旧资源会暂时保留，之后可单独清理。
