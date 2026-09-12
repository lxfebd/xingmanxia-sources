# 星漫匣 源市场 / 能力市场 远端索引

本仓库托管「星漫匣」应用内两个市场的远端索引（`index.json`），以及 AI
能力的权重/引擎包（走 GitHub Release 分发，不入仓库、不占 git 体积）。

## index.json 格式

```jsonc
{
  "name": "市场名",
  "sources": [ /* 源市场条目：完整 CustomSourceDef JSON */ ],
  "capabilities": [ /* 能力市场条目 */ ]
}
```

- **sources**：源市场条目。每条是完整的源定义（含 baseUrl + rules），
  应用内「源市场页」一键安装（importJson 语义）。
- **capabilities**：能力市场条目。字段：
  - `id` / `name` / `category`（ai|video|utility）/ `version` / `author` / `description`
  - `weights`: 模型权重列表 `{name, url, sizeBytes, sha256}`（运行期下载）
  - `artifact`: 原生构件 `{url, sha256: {abi: hex}}`（桌面直链）

## Release

- `models-v1`：首个模型/引擎包发布
  - `ddcolor_fp32.tflite`（AI 上色权重，约 215MB）
  - `rife-engine-win.zip`（AI 插帧引擎包，Windows，约 12MB）

## 治理

- 权重/引擎包一律走 Release 附件，**绝不提交进仓库**（公开仓库红线）。
- 版本钉死：`sha256` 与代码常量一致才可发布；升级需同步 App 代码常量。
- 内部文档、密钥、开发域名不入库。
