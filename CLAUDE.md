# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file Bash CLI (`paperless`) that wraps the [Paperless-ngx](https://docs.paperless-ngx.com/) REST API. No build step — the script runs directly.

## Running and testing

```bash
# Set required environment variables
export PAPERLESS_HOST=https://your-paperless-instance
export PAPERLESS_TOKEN=your-api-token

# Run directly
bash paperless search -q "invoice"

# Install on PATH
chmod +x paperless
ln -s "$PWD/paperless" /usr/local/bin/paperless
```

There is no test suite. Manual testing requires a live Paperless-ngx instance.

## Architecture

Everything lives in `paperless` — one flat Bash script structured in four sections:

1. **Env validation & dep checks** — `PAPERLESS_HOST` / `PAPERLESS_TOKEN` must be set; `curl` and `jq` must be available.
2. **API helpers** (`api_request`, `api_get`, `api_post`, `api_patch`, `api_get_raw`) — all HTTP goes through `api_request`, which captures the HTTP status code separately from the body using a temp file pattern.
3. **Resolution helpers** (`fetch_tag_map`, `fetch_correspondent_map`, `resolve_results`) — tag and correspondent IDs are resolved to names client-side. Maps are built with `jq` into `{"id": "name"}` objects. Note: **no associative arrays** — the code is intentionally Bash 3.2 compatible (macOS default shell).
4. **Command functions** (`cmd_search`, `cmd_get`, `cmd_content`, `cmd_download`, `cmd_tags_*`) — one function per subcommand, dispatched from `main`.

## Key conventions

- All output is JSON (including download success response), errors go to stderr.
- `resolve_results` always runs after fetching documents; it strips internal fields and replaces IDs with names.
- `--all` in `search` paginates automatically using Paperless-ngx's `.next` cursor URL.
- `tags add` creates the tag if it doesn't exist, then patches the document.
- URL encoding is done via `jq -Rr @uri` (the `url_encode` function).
- No `declare -A` — use `jq` for map operations to stay Bash 3.2 compatible.
