# Agnes Factory

单文件 HTML 的 Agnes AI 图像/视频创作工具。双击即用，无需安装、无需构建、无需后端。

复刻 [Agnes 创意空间](https://ailabing.cn/tools/agnes-creative-space/) 的核心生图生视频能力，并加了**按 RPM 档位自适应的重试策略**，专治免费档"一生成就 429 / 一直排队"的问题。

## 功能

- **文生图** / **图生图**（最多 4 张参考图）
- **文生视频** / **图生视频**（首尾帧控制 或 多图参考两种方式）
- 模型切换：Agnes Image 2.0 / 2.1 / 2.5 Flash，Video 2.5 / 2.5-Flash
- 宽高比、图像尺寸档位（1K–4K）、视频时长（4–12s）
- 提示词优化（可选，调 Agnes 文本模型扩写）
- 生成结果网格 + 下载
- API key 存 `localStorage`，下次打开自动带出

## 使用方法

1. 双击 `index.html` 在浏览器打开
2. 右上角填入你的 Agnes API key（从 [Agnes 控制台](https://platform.agnes-ai.com) 获取）
3. 左侧选模式 → 填提示词 / 传参考图 → 点"生成"
4. 结果出现在右侧网格，可下载

## 为什么加"按 RPM 档位自适应重试"

Agnes 免费档的 RPM 限制（来自[官方 Token Plan FAQ](https://wiki.agnes-ai.com/zh-Hans/docs/tokenplan)）：

| 档位 | 实际 RPM |
|---|---|
| 图片 1K | 20 |
| 图片 2K | 10 |
| 图片 3K / 4K | 1 |
| 视频 | 1 |

旧版"失败自动重试"用固定 1.5s 间隔，在 RPM=1 的档位（3K/4K/视频）会连发打穿限流，导致"一直 429 / 一直排队"。本工具改为：

- **普通失败**（任务 failed / 5xx / 网络错）→ 按档位间隔自动重提：1K=3s，2K=6s，3K/4K/视频=60s
- **限流 429** → **不自动重提，停下来**，提示"等约 Ns 后手动点生成"

两层配合：间隔准让你尽量不撞墙，撞了就停、不连发。

## 已知限制

- **免费档视频 RPM=1**：后端算力紧张时任务进 `queued` 后可能排很久才到 `in_progress` → `completed`，这是正常现象，不是 bug。轮询会一直查下去直到出结果或失败。
- **CORS**：前端直连 `apihub.agnes-ai.com`。如果浏览器把请求拦了（控制台报 `CORS policy`），需要自己起个本地代理转发，或部署到同域/配好 CORS 的环境。
- 视频生成较慢，12s 视频渲染时间约为 4s 的 3 倍。

## 技术说明

- 单文件：HTML + 内联 CSS + 内联 JS，无依赖
- 图像 API：`POST /v1/images/generations`（`extra_body.image` 传参考图）
- 视频 API：`POST /v1/videos` 创建 + `GET /agnesapi?video_id=...&model_name=...` 查询（异步任务）
- 提示词优化：`POST /v1/chat/completions`（agnes-3.0-flash）

## 许可证

[MIT](LICENSE)
