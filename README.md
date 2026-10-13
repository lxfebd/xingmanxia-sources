# 星漫匣 源市场 / 能力市场 远端索引

本仓库托管「星漫匣」应用内两个市场的远端索引（`index.json`），以及 AI
能力的权重/引擎包（走 GitHub Release 分发，不入仓库、不占 git 体积）。

## index.json 格式

```jsonc
{
  "name": "市场名",
  "sources": [ /* 源市场条目：完整 CustomSourceDef JSON */ ],
  "capabilities": [ /* 能力市场条目 */ ],
  "_sig": "base64(r‖s)，见下方「发布：索引必须签名」"
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

## 发布：索引必须签名

App 内置 ECDSA P-256 公钥，对 `index.json` 做**来源校验**——防止 gh-proxy /
jsDelivr / 仓库内容被替换时，App 把伪造的源定义（恶意请求 URL、选择器、
请求头）当货装上。规则：**签名错误 → 拒绝加载并回退缓存；未签名 → 过渡期
告警放行**。所以每次改完 `index.json` 都必须重新签名再 push。

```bash
# 在 App 仓库（clean_manhua_flutter）内执行
dart run tool/sign_index.dart <本仓库路径>/index.json
```

- 首次生成密钥对：`dart run tool/sign_index.dart --genkey`；把打印的公钥追加进
  App 的 `lib/utils/index_signature.dart` → `kIndexPublicKeysHex`，私钥落在
  `~/.xingmanxia/index_signing_key.json`，**绝不进任何仓库**。
- 签名字段是顶层 `_sig`（base64 的 r‖s，各 32 字节）。签名对象 = 去掉 `_sig`
  后重新编码的 JSON 字节，**顶层键序参与签名**：手工编辑后务必重跑签名脚本，
  只改内容不重签会被 App 当成篡改拦下。
- 验签只证明「作者签过」，**不证明内容新鲜**：jsDelivr 的 `@main` 分支缓存
  可达 7 天。故 App 把官方直连排在候选首位，见 `market_index_urls.dart`。
- 回归证据在 App 的 `test/fixtures/market_signed_sample.json` +
  `test/regression_market_index_signature_test.dart`：脚本格式一旦被改动，
  那条测试立刻红。

## 治理

- 权重/引擎包一律走 Release 附件，**绝不提交进仓库**（公开仓库红线）。
- 版本钉死：`sha256` 与代码常量一致才可发布；升级需同步 App 代码常量。
- 内部文档、密钥、开发域名不入库。
