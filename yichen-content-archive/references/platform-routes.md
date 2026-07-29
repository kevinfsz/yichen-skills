# 已知链接与精确容器路由

只引用现有 Skill、后端和脚本，不复制实现。调用前读取目标 Skill/参考文件或运行脚本 `--help`；本表只限制可用分支，不替代目标入口的安全规则。

| 平台 | 已知单项 `read` | 已知单项 `download/archive` | `known_collection` 精确枚举 | 明确禁止 |
|---|---|---|---|---|
| 普通网页 | Jina Reader `https://r.jina.ai/<URL>`；需图片/格式控制时用 Web Reader | 把同一已知 URL 的 Markdown、文本、HTML 或用户明确指定的原始 HTTP 响应写入新文件 | URL 文件逐行处理；不从网页继续提取站内链接 | 站点爬取、sitemap 扩展、搜索结果页扩展、相似链接 |
| 小红书 | `$yichen-xiaohongshu-fetch` 的 `fetch.py <URL> <dir> --skip-media` | 同一 Skill 的已知笔记下载；只有当轮授权后才可 `--use-cookie` | 只接受已经给出的 URL 文件，不枚举用户主页或收藏 | 搜索笔记、作者发现、从收藏页扩展 |
| 抖音 | `$yichen-douyin-fetcher` 的 `download.py <URL> --metadata-only` | 同一 Skill 的已知视频下载 | 只接受已经给出的 URL 文件，不枚举账号或收藏 | 搜索、推荐页采样、账号发现 |
| 微信公众号 | 单个已知 URL 用 `$yichen-wechat-mp-batch-exporter` 读取；本地归档也可用 `~/.agents/skills/yichen-content-archive/scripts/wechat_mp_local.py download` | 已知 URL 文件用同一 Skill 的 `download_urls.py` 或本机脚本 | 对用户精确指定的公众号名称/容器，当轮授权后用本机 `wechat_mp_local.py search --allow-local-account-session --account "<EXACT_ACCOUNT>" --limit-per-account N --download` 枚举并归档 | 跨公众号关键词搜索、模糊账号扩展、增强指标、评论、代理；任何微信 UI 代操作 |
| YouTube | `yt-dlp --dump-json <URL>` 或已知 URL 字幕路径 | `yt-dlp <URL>` 下载音视频或字幕 | 对用户给出的播放列表 URL 用 `yt-dlp --flat-playlist --dump-single-json <URL>` 生成固定条目清单，再归档 | `search`、`channel`、相关推荐和从条目跨到其他播放列表 |
| B站 | 用 `bili-cli` 的 `bili video <BV_OR_URL> --json` 读取 BV/完整 URL；AV 先规范成 `https://www.bilibili.com/video/av<ID>/` 再读取 | `yt-dlp` 只下载已知 BV/AV/完整 URL；使用 `--download-archive`、`--continue`、`--no-overwrites` | 对用户给出的播放列表/合集/分P容器 URL 用 `yt-dlp --flat-playlist --dump-single-json <URL>`；后端不支持时停止 | `bili search`、UP 主空间扩展、相关视频、私人收藏和稍后再看 |
| 小宇宙 | 已知 episode URL 用 `~/.agents/skills/yichen-content-archive/scripts/xiaoyuzhou_stepfun.py --inspect-only <URL>`，匿名优先 | 同一脚本 `--download-only`；只有用户明确要求转写并确认数量/额度时才去掉该参数 | 已知 episode URL 文件直接用 `--batch-file` 匿名处理；已知 podcast ID/URL 需当轮账号令牌授权后用本 Skill 的 `xiaoyuzhou_opencli.py episodes` 生成清单，再回到匿名 `--batch-file` | 全站关键词搜索、从单集找相似播客、未授权账号令牌、自动调用付费 ASR |

## 路由纪律

- 短链接的单次规范化或跟随重定向属于已知链接解析，不得借机抓取推荐列表。
- `known_collection` 必须先写固定清单，并记录容器引用、用户指定上限、实际条数和是否截断。清单条目不能继续扩展子来源。
- URL 文件属于用户已提供的精确清单，不运行链接发现器或站内爬虫。
- 微信公众号历史只允许精确账号容器和本机 `127.0.0.1:18901` exporter，并要求当轮 `--allow-local-account-session` 授权。登录失效时等待用户本人完成本地页面扫码/手机确认；不得操控微信客户端，不进入代理、评论或阅读量分支。
- YouTube 只能使用已知 URL/播放列表路径；即使目标 Skill 同时支持搜索，也不得调用 `search` 或 `channel`。
- B站公开内容匿名优先。高画质、高码率、4K/HDR/杜比、会员/已购/地区年龄限制、私人数据或明确要求登录的字幕，必须说明具体目标和原因，取得当轮 Cookie 授权；普通 412 不自动升级为登录态。
- 小宇宙 episode 页面、音频和已给出的 episode URL 清单匿名优先。只有已知 podcast 结构化列表、平台已有转写或匿名失败时，才按目标取得当轮账号令牌授权；批量 StepFun 前先确认节目数量和额度。
- 小宇宙 StepFun 默认固定目录冲突时写入新的 `-run-N` 目录；只有显式 `--resume` 且 metadata、大小和 SHA-256 一致时复用。OpenCLI 的 `--overwrite` 是兼容性破坏模式，Agent 禁止自动使用；缺少与 `--output-dir` 完全一致的 `--confirm-overwrite-exact-dir` 时必须失败。
- 公众号包装器的显式既有目录默认失败；`--resume-existing` 只读取既有 `index.csv` checkpoint，并把待处理项写入旧 run 同级的 `<name>-resume-<run_id>` 新目录，旧目录树保持不变。
- 平台 Skill 要求登录态、Cookie、代理、证书或其他人工门时，以它的更严格规则为准。
- 容器类型不受现有后端支持时只生成失败项：`stage=enumeration`、`reason=unsupported_container`；不得降级成搜索。
- 不支持的平台只生成失败项：`stage=route`、`reason=unsupported_platform`。
