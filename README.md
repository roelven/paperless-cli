# paperless-cli

A minimal bash CLI for [Paperless-ngx](https://docs.paperless-ngx.com/) designed for LLM and scripting use. Outputs clean JSON.

## Requirements

- `curl`
- `jq`
- Bash 3.2+

## Setup

```bash
export PAPERLESS_HOST=https://your-paperless-instance
export PAPERLESS_TOKEN=your-api-token
```

Get your API token from the Paperless-ngx web UI under **Settings → My Account**, or via:

```bash
curl -s -X POST https://your-paperless-instance/api/token/ \
  -d '{"username":"you","password":"secret"}' \
  -H "Content-Type: application/json" | jq -r .token
```

Make the script executable and put it on your PATH:

```bash
chmod +x paperless
ln -s "$PWD/paperless" /usr/local/bin/paperless
```

## Usage

```
paperless <command> [options]

Commands:
  search [-q QUERY] [-t TITLE] [-s SENDER] [--tag TAG] [--limit N] [--all]
  get <id>
  content <id>
  download <id> [--output PATH] [--original]
  tags list
  tags add <doc_id> <tag_name>
  tags remove <doc_id> <tag_name>
```

### Examples

```bash
# Full-text search
paperless search -q "invoice" -s "Amazon" --limit 10

# Fetch all results across pages
paperless search -t "bank statement" --all

# Get document metadata + OCR content
paperless get 42

# Get raw OCR text (pipe-friendly)
paperless content 42 | wc -w

# Download PDF
paperless download 42 --output ~/Downloads/doc.pdf

# Tag management
paperless tags list
paperless tags add 42 "urgent"
paperless tags remove 42 "urgent"
```

## Output

All commands return JSON. `search` and `get` resolve tag and correspondent IDs to names automatically.
