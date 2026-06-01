# mini-iptv

[English](README.md) | 中文

`mini-iptv` 是从 `Guovin/iptv-api` 提取出来的最小化 GitHub Actions IPTV 播放列表生成器。它只保留采集、筛选、测速、EPG 和播放列表生成流程，移除了桌面 UI、网页服务、Docker 镜像构建、RTMP 服务、发布打包和其它无关代码。

## 使用的技术

- Python 3.13 运行更新流水线。
- GitHub Actions 按计划运行流水线，并把生成文件提交回仓库。
- `requests`、`aiohttp` 和 `beautifulsoup4` 用于获取和解析订阅源、EPG 源。
- `tqdm` 在 Actions 日志中输出进度。
- `opencc-python-reimplemented` 把 EPG 中的繁体中文规范化为简体中文。
- `m3u8` 和 FFmpeg/FFprobe 用于直播流检查、测速和媒体信息探测。
- `config/` 下的纯文本配置文件控制频道、订阅源、白名单、黑名单和 EPG 源。

## 具体实现

1. GitHub Actions 每天或手动触发时检出仓库。
2. workflow 安装 Python 3.13、FFmpeg 和 `requirements.txt` 中的依赖。
3. `python main.py` 读取 `config/config.ini` 和 `config/demo.txt` 中的频道模板。
4. 程序获取 `config/subscribe.txt` 中的订阅源，并按目标频道名进行匹配。
5. 程序应用 `config/whitelist.txt` 和 `config/blacklist.txt` 中的白名单、黑名单规则。
6. 程序获取 `config/epg.txt` 中的 EPG 源，并写入 `output/epg/`。
7. 候选直播流会按 `config/config.ini` 中的规则进行可用性、速度和分辨率测试。
8. 最终播放列表写入 `output/`。
9. 如果生成文件发生变化，workflow 会自动提交并推送回仓库。

workflow 每天 `18:00 UTC` 执行一次，也就是 `Asia/Shanghai` 时区的次日 `02:00`。你也可以在 GitHub Actions 页面手动运行。

## 生成文件

主要文件：

- `output/result.m3u`：包含全部结果的 M3U 播放列表。大多数 IPTV 播放器建议使用这个文件。
- `output/result.txt`：包含全部结果的 TXT 播放列表。
- `output/ipv4/result.txt`：只包含 IPv4 结果的 TXT 播放列表。
- `output/ipv6/result.txt`：只包含 IPv6 结果的 TXT 播放列表。
- `output/epg/epg.gz`：压缩后的 EPG 文件，会被生成的 M3U 播放列表引用。

本地运行时也可能生成缓存文件、`output/epg/epg.xml` 和 IPv4/IPv6 M3U 文件。这些文件会被 Git 忽略，因为它们是中间产物或重复产物。

## 公开地址

workflow 成功运行后，使用这些 raw 地址：

```text
https://raw.githubusercontent.com/lonnnnnng/mini-iptv/main/output/result.m3u
https://raw.githubusercontent.com/lonnnnnng/mini-iptv/main/output/result.txt
https://raw.githubusercontent.com/lonnnnnng/mini-iptv/main/output/ipv4/result.txt
https://raw.githubusercontent.com/lonnnnnng/mini-iptv/main/output/ipv6/result.txt
https://raw.githubusercontent.com/lonnnnnng/mini-iptv/main/output/epg/epg.gz
```

## 配置

运行 workflow 前先编辑这些文件：

- `config/demo.txt`：目标频道模板。只有这里列出的频道会被生成。
- `config/subscribe.txt`：IPTV 订阅源 URL。
- `config/whitelist.txt`：需要优先保留的 URL 或源。
- `config/blacklist.txt`：需要过滤的 URL 关键词或规则。
- `config/epg.txt`：EPG 源 URL。
- `config/config.ini`：功能开关和阈值，包括测速、分辨率过滤、URL 数量限制、IPv4/IPv6 偏好和输出路径。

`config/config.ini` 中常用设置：

- `open_subscribe = True`：开启订阅源采集。
- `open_epg = True`：开启 EPG 生成。
- `open_speed_test = True`：开启直播流测速。
- `open_filter_resolution = True`：按分辨率过滤直播流。
- `open_filter_speed = True`：按最低速度过滤直播流。
- `urls_limit = 5`：每个频道最多保留 5 个 URL。

`location` 和 `isp` 过滤默认关闭。如果你开启其中任意一项，需要安装 `ipip-ipdb`，并提供 `utils/ip_checker/data/qqwry.ipdb`。

## GitHub Actions 配置

本仓库已经包含 `.github/workflows/update.yml`。

如果用于新仓库：

1. 把本项目作为仓库根目录推送。
2. 打开 `Settings -> Actions -> General`。
3. 把 `Workflow permissions` 设置为 `Read and write permissions`。
4. 打开 `Actions -> Update IPTV`。
5. 第一次点击 `Run workflow` 手动运行。
6. workflow 提交生成的播放列表文件后，即可使用 raw 地址。

不需要配置 secrets。

## 本地运行

先安装 FFmpeg，然后运行：

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python main.py
```

生成文件会写入 `output/`。

## 许可证

本项目派生自 `Guovin/iptv-api`，保留原 AGPL-3.0 许可证。
