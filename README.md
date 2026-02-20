# Great Transport

YouTube-to-Bilibili video transport system with LLM-powered discovery and ML ranking.

## Architecture

- **Go backend** — channel scanning, rule-based filtering, video download (yt-dlp), Bilibili upload (biliup), performance tracking
- **Python ML service** — competitor monitoring, LLM-powered keyword discovery (Ollama), ML view prediction (LightGBM/GPBoost), auto-labeling pipeline

## Requirements
- `yt-dlp` in `PATH`
- `ffmpeg` recommended (improves format handling)
- [`biliup` CLI](https://github.com/biliup/biliup) in `PATH` for Bilibili uploads
  - Run `biliup --user-cookie cookies.json login` once to create upload credentials
- Python 3.11+ with dependencies in `ml-service/requirements.txt`
- [Ollama](https://ollama.com/) running locally for LLM-powered discovery

## Usage

### Go Backend
```bash
go run . --video-id dQw4w9WgXcQ --platform bilibili
go run . --channel-id UC_x5XG1OV2P6uZZ5FSM9Ttw --limit 3 --sleep-seconds 5
```

Options:
- `--channel-id` YouTube channel ID or URL
- `--video-id` YouTube video ID or URL
- `--platform` `bilibili` or `tiktok`
- `--output` output directory (default: `downloads`)
- `--limit` max videos for channel downloads (default: 5)
- `--sleep-seconds` sleep between downloads to reduce rate (default: 5)
- `--biliup-cookie`, `--biliup-line`, `--biliup-limit`, `--biliup-tags`, etc. for uploader knobs

### ML Service
```bash
cd ml-service

# Collect training data
python collect.py --round 1

# Run discovery pipeline (trending keywords → YouTube search → LLM scoring → ML ranking)
python -m app.cli --db-path data.db discover

# Train view prediction model
python -m app.cli --db-path data.db train
```

## Docker
```bash
docker build -t great-transport .
docker run --rm -v "$PWD/downloads:/app/downloads" great-transport \
  --channel-id UC_x5XG1OV2P6uZZ5FSM9Ttw --limit 3 --sleep-seconds 5
```

For biliup authentication inside the container:
```bash
touch cookies.json
docker run --rm -it \
  -v "$PWD/cookies.json:/app/cookies.json" \
  --entrypoint biliup great-transport \
  --user-cookie /app/cookies.json login
```
