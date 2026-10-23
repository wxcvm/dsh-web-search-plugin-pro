# 安装与推送

## 这是什么

`dsh-web-search-plugin@0.6.0` 的加强分支（MIT，上游 `X-C1811/dsh-web-search-plugin`）。
在 0.6.0 基础上做了四件事，全部是本地改动、无新增依赖：

| 补丁 | 内容 |
|---|---|
| `0001` | 有 key 的 Tavily 取正文（`include_raw_content`），短摘要自动升级为长正文；错误体里 `{error:"文本"}`/`{code}` 能被读到并带上 provider id；429/503/5xx 重试一次（尊重 `Retry-After`）；SearXNG 返回 HTML 时明确报「实例关了 format=json」 |
| `0002` | 新增 `auto` 路由：按 `autoOrder` 顺序试，第一个出结果的赢；失败的引擎冷却 `autoCooldownMs`（默认 60s）；全挂时抛聚合错误 |
| `0003` | `auto` 命中后结果末尾标注 `(auto: tavily)`，`autoAnnotate: false` 可关 |
| `0004` | 本文档与补丁说明 |

## 从 GitHub 安装

```powershell
dsh plugin --profile web add github:wxcvm/dsh-web-search-plugin-pro
```

装上之后重启 DSH（web profile 关掉了 HMR，刷新页面不算重启）。

> 包名仍是 `dsh-web-search-plugin`（沿用上游，方便做 fork）。**不能和原插件同时装**——同名包会互相覆盖，且两者都注册 `dsh-web-search` 这个 provider id。升级前先 `dsh plugin remove dsh-web-search-plugin`。

## 本地安装（不打 tag 也行）

```powershell
git clone https://github.com/wxcvm/dsh-web-search-plugin-pro
dsh plugin --profile web add ./dsh-web-search-plugin-pro
```

## 只想要补丁

```powershell
cd "C:\Users\Kerna\.dsh\profiles\desktop\node_modules\dsh-web-search-plugin"
git apply 0001-*.patch 0002-*.patch 0003-*.patch
```

`patches/` 里是 `git format-patch` 产物，可 `git am` 保留提交信息。

## 配置

```yaml
provider: auto
autoOrder: "tavily,brave,serper,exa,searxng,firecrawl,deepseek-official"
autoCooldownMs: 60000
autoAnnotate: true
```

设置页的引擎下拉里也有 **Auto（按顺序取第一个可用引擎）**。

## 已知边界

- `web_search` 的工具 schema 在 DSH 核心（`dsh-tool-web`），返回结构固定为 `content + sources[]`；**图片搜索塞不进来**，需要独立工具（另见 `dsh-web-images`）。
- `include_raw_content` 只在 `keyed` 模式发，keyless 免费额度不支持，避免 400。
- 免费引擎的 keyless 额度本身很紧，429 依然常见；补丁 1 的重试只能兜一次，不能凭空造额度。
