# API Base URL 与余额查询控制总览

## 固定线路
- 默认值存储：`config_manager.py` 中 `_DEFAULT_ROUTE_KEY=hk`、`_DEFAULT_API_BASE_URL_CODEPOINTS` 与 `_API_BASE_URLS_ENCODED`。默认线路为香港专线 `https://hk-api.aabao.top`。
- 显性线路固定三条：香港专线 `https://hk-api.aabao.top`、直连美区 `https://api.aabao.top`、CF专线 `https://cf-api.aabao.top`。节点 UI “线路”下拉按该顺序展示，默认香港专线。
- 隐藏线路：当 API Key 以 `fixsk-` 前缀（不区分大小写）开头时，自动切换 Base URL 为 `https://api666.zeabur.app`，并将请求密钥改为去除 `fix` 前缀后的值（例：`fixsk-abc` → `sk-abc`）。显性线路选择在该场景下被忽略。
- Base URL 不再读取 `config.ini` 或 `banana_gemini_test.local.ini` 中的覆盖字段，统一由线路选择与隐藏前缀决定。

## 线路选择与节点行为
- “心宝❤Banana” 节点将原“高峰模式”开关替换为“线路”下拉；生成请求、余额查询与 KV 鉴权统一使用所选线路解析的 Base URL。
- 未显式选择时使用默认线路（香港）。隐藏前缀逻辑优先级最高。
- 网络超时策略统一：连接 15s + 读取 120s；`gemini-3-pro-image*` 系列读取超时统一放宽到 320s。

## 余额查询链路
- 前端 `web/extensions/token-balance.js` 挂载的查询按钮会携带 `api_key`、`bypass_proxy`、`disable_ssl_verify` 以及 `route`（节点“线路”值）到 `/banana/token_usage`。
- 后端 `balance_service.py` 在 `ensure_route()` 中根据 `route` 调用 `ConfigManager.get_route_base_url()`，刷新或读取缓存后返回摘要。
- 缓存键包含 Base URL 与 API Key，避免跨线路混用。

## 维护脚本
- `tools/set_api_base_url.ps1` 仅支持三条显性线路的选择，用于更新 `_DEFAULT_ROUTE_KEY` 与 `_DEFAULT_API_BASE_URL_CODEPOINTS`，并输出四条线路的混淆值（含隐藏线路）。不再写入 `config.ini`，也不接受自定义 URL。
