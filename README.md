# mini-iptv

Minimal GitHub Actions runner extracted from Guovin/iptv-api for collecting, filtering, speed testing, and generating IPTV playlists.

## Usage

1. Edit `config/demo.txt` for target channels.
2. Edit `config/subscribe.txt` for subscription sources.
3. Optionally edit `config/whitelist.txt`, `config/blacklist.txt`, and `config/epg.txt`.
4. Push this directory to a GitHub repository.
5. Enable `Settings -> Actions -> General -> Workflow permissions -> Read and write permissions`.
6. Run `Update IPTV` manually from Actions, or wait for the 6-hour schedule.

Generated files are written to `output/`, mainly:

- `output/result.txt`
- `output/result.m3u`
- `output/ipv4/result.txt`
- `output/ipv6/result.txt`
- `output/epg/epg.xml`
- `output/epg/epg.gz`

`location` and `isp` filtering is disabled by default. If you enable either option, install `ipip-ipdb` and provide `utils/ip_checker/data/qqwry.ipdb`.

## Local run

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python main.py
```
