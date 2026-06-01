# mini-iptv

English | [中文](README.zh-CN.md)

`mini-iptv` is a minimal GitHub Actions IPTV playlist generator extracted from `Guovin/iptv-api`. It keeps only the collection, filtering, speed testing, EPG, and playlist generation flow, and removes the desktop UI, web service, Docker image build, RTMP service, release packaging, and other unrelated code.

## Technology

- Python 3.13 runs the update pipeline.
- GitHub Actions runs the pipeline on a schedule and commits generated files back to the repository.
- `requests`, `aiohttp`, and `beautifulsoup4` fetch and parse subscription and EPG sources.
- `tqdm` prints progress in Actions logs.
- `opencc-python-reimplemented` normalizes traditional Chinese EPG text to simplified Chinese.
- `m3u8` and FFmpeg/FFprobe are used during stream checking, speed testing, and media metadata probing.
- Plain text configuration files under `config/` control channels, subscription sources, whitelist, blacklist, and EPG sources.

## How It Works

1. GitHub Actions checks out the repository once per day or when manually triggered.
2. The workflow installs Python 3.13, FFmpeg, and packages from `requirements.txt`.
3. `python main.py` reads `config/config.ini` and channel templates from `config/demo.txt`.
4. Subscription URLs from `config/subscribe.txt` are fetched and matched against the target channel names.
5. Whitelist and blacklist rules from `config/whitelist.txt` and `config/blacklist.txt` are applied.
6. EPG sources from `config/epg.txt` are fetched and written to `output/epg/`.
7. Candidate stream URLs are tested for availability, speed, and resolution according to `config/config.ini`.
8. The final playlists are written to `output/`.
9. If generated files changed, the workflow commits and pushes them back to the repository.

The workflow runs once per day at `18:00 UTC`, which is `02:00` the next day in `Asia/Shanghai`. You can also run it manually from the GitHub Actions page.

## Generated Files

Main files:

- `output/result.m3u`: M3U playlist containing all generated results. This is the recommended file for most IPTV players.
- `output/result.txt`: TXT playlist containing all generated results.
- `output/ipv4/result.txt`: TXT playlist containing only IPv4 results.
- `output/ipv6/result.txt`: TXT playlist containing only IPv6 results.
- `output/epg/epg.gz`: compressed EPG file referenced by the generated M3U playlist.

Local runs may also create cache files, `output/epg/epg.xml`, and IPv4/IPv6 M3U files. Those files are ignored by Git because they are intermediate or duplicate outputs.

## Public URLs

After the workflow has run successfully, use these raw URLs:

```text
https://raw.githubusercontent.com/lonnnnnng/mini-iptv/main/output/result.m3u
https://raw.githubusercontent.com/lonnnnnng/mini-iptv/main/output/result.txt
https://raw.githubusercontent.com/lonnnnnng/mini-iptv/main/output/ipv4/result.txt
https://raw.githubusercontent.com/lonnnnnng/mini-iptv/main/output/ipv6/result.txt
https://raw.githubusercontent.com/lonnnnnng/mini-iptv/main/output/epg/epg.gz
```

## Configuration

Edit these files before running the workflow:

- `config/demo.txt`: target channel template. Only channels listed here are generated.
- `config/subscribe.txt`: IPTV subscription source URLs.
- `config/whitelist.txt`: URLs or sources that should be kept with priority.
- `config/blacklist.txt`: URL keywords or rules that should be filtered out.
- `config/epg.txt`: EPG source URLs.
- `config/config.ini`: feature switches and thresholds, including speed testing, resolution filtering, URL limits, IPv4/IPv6 preferences, and output paths.

Common settings in `config/config.ini`:

- `open_subscribe = True`: enable subscription source collection.
- `open_epg = True`: enable EPG generation.
- `open_speed_test = True`: enable stream speed testing.
- `open_filter_resolution = True`: filter streams by resolution.
- `open_filter_speed = True`: filter streams by minimum speed.
- `urls_limit = 5`: keep up to 5 URLs per channel.

`location` and `isp` filtering is disabled by default. If you enable either option, install `ipip-ipdb` and provide `utils/ip_checker/data/qqwry.ipdb`.

## GitHub Actions Setup

This repository already includes `.github/workflows/update.yml`.

For a new repository:

1. Push this project as the repository root.
2. Open `Settings -> Actions -> General`.
3. Set `Workflow permissions` to `Read and write permissions`.
4. Open `Actions -> Update IPTV`.
5. Click `Run workflow` for the first run.
6. Use the raw URLs after the workflow commits the generated playlist files.

No secrets are required.

## Local Run

Install FFmpeg first, then run:

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python main.py
```

Generated files will be written to `output/`.

## License

This project is derived from `Guovin/iptv-api` and keeps the original AGPL-3.0 license.
