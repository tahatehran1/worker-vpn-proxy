<p align="center">
  <img src="assets/worker-vpn-proxy-icon.svg" alt="Worker VPN Proxy Tester icon" width="120" />
</p>

# Worker VPN Proxy Tester

A Python-based proxy tester that collects public HTTP/HTTPS proxy lists, validates them against a real target, and exports the fastest working proxies into `best_proxies.json` and `best_proxies.txt`.

This project is optimized for testing and collecting reliable HTTP and HTTPS proxies from multiple public sources, with concurrency and filtering designed to surface working endpoints quickly.

## Features

- Download public proxy lists from multiple sources
- Focuses on HTTP and HTTPS proxies, with additional mixed and SOCKS support
- Tests proxies concurrently with `aiohttp`
- Sorts successful proxies by response time
- Saves results to JSON and plain text output files
- Includes a scheduled GitHub Actions workflow for recurring updates
- Includes a Dockerfile and GitHub Container Registry build workflow

## Why this project

The script intentionally prioritizes HTTP and HTTPS sources because they are the most useful for browser, API, and service testing. It combines several public proxy lists and keeps duplicates out of the final output while ranking the best performers by latency.

## Requirements

- Python 3.11+
- Access to proxy list sources and `https://www.google.com`

Install dependencies:

```bash
pip install -r requirements.txt
```

## Run locally

```bash
python main.py
```

When the script completes, it writes these files to the project root:

```text
best_proxies.json
best_proxies.txt
```

You can also fetch the latest plain-text list directly from jsDelivr:

```text
https://cdn.jsdelivr.net/gh/tahatehran/worker-vpn-proxy/best_proxies.txt
```

## Output format

The JSON result contains:

- `total_proxies`: number of working proxies found
- `timestamp`: when testing finished
- `test_url`: the validation URL used during the check
- `proxies`: successful results sorted by speed

The TXT file contains one proxy per line:

```text
203.0.113.10:8080
198.51.100.20:3128
```

## Docker usage

Build locally:

```bash
docker build -t worker-vpn-proxy .
```

Run and save the generated proxies to your current directory:

```bash
docker run --rm -v "$PWD:/app" worker-vpn-proxy
```

The project also publishes a ready-to-use image via GitHub Container Registry:

```bash
docker pull ghcr.io/tahatehran1/worker-vpn-proxy:latest
docker run --rm -v "$PWD:/app" ghcr.io/tahatehran1/worker-vpn-proxy:latest
```

## GitHub Actions

The repository includes:

- `.github/workflows/proxy-tester.yml` for scheduled proxy testing and committing outputs
- `.github/workflows/docker-publish.yml` for building and publishing a Docker image to GHCR

These workflows support:

- automatic proxy list refreshes
- manual trigger execution
- package publishing for easier container-based testing and deployment

## Configuration

The most important values are near the top of `main.py`:

```python
CONCURRENT = 200
BATCH_SIZE = 400
PROXY_LIMIT = 1000
```

Proxy sources are defined in the `APIs` dictionary. Each entry contains:

- `url`: source URL
- `format`: `json` or `txt`
- `filter`: a custom data filter
- `type`: protocol label such as `http`, `https`, `mixed`, or `socks5`

## Notes

Public proxy lists are inherently unstable. Many entries disappear, rotate quickly, or fail under load. The script filters for response quality and latency, but results can still vary from run to run.
