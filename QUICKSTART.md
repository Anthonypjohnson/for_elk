# Quick Start Guide

## 1. First Time Setup

### Configure Settings
Edit `.env` and configure:
```bash
# Passwords
ELASTIC_PASSWORD=your_secure_password
KIBANA_SYSTEM_PASSWORD=your_secure_password
LOGSTASH_INTERNAL_PASSWORD=your_secure_password

# License (optional - defaults to basic/free)
LICENSE_TYPE=basic    # or 'trial' for 30-day all features
```

See [LICENSING.md](LICENSING.md) for license options.

### Increase Virtual Memory (macOS)
```bash
sudo sysctl -w vm.max_map_count=262144
```

## 2. Start the Stack

```bash
docker compose up -d
```

Wait 2-3 minutes for services to initialize.

## 3. Verify Services

```bash
# Check all services are running
docker compose ps

# Should show: elasticsearch, kibana, logstash, setup (exited - that's OK)
```

## 4. Access Kibana

Open browser to: **http://localhost:5601**

- Username: `elastic`
- Password: (your password from .env)

## 5. Add Hayabusa Logs

### For CSV files:
```bash
cp your_hayabusa_output.csv data/hayabusa/csv/
```

### For JSON files:
```bash
cp your_hayabusa_output.jsonl data/hayabusa/json/
```

Logstash will automatically process files (check logs with `docker compose logs -f logstash`)

## 6. Create Data View in Kibana

1. Go to **Stack Management** → **Data Views**
2. Click **Create data view**
3. Name: `Hayabusa Logs`
4. Index pattern: `hayabusa-*`
5. Timestamp field: `@timestamp`
6. Click **Save**

## 7. View Your Data

Go to **Discover** and select the `Hayabusa Logs` data view.

Search examples:
- `Level: "critical"` - Show only critical events
- `EventID: "4625"` - Show failed logon attempts
- `Computer: "DESKTOP-ABC"` - Filter by computer name
- `MitreTactics: *` - Show events with MITRE tactics

## Common Commands

```bash
# Start
docker compose up -d

# Stop
docker compose down

# View logs
docker compose logs -f logstash

# Restart after config changes
docker compose restart logstash

# Check Elasticsearch health
curl -k -u elastic:password https://localhost:9200/_cluster/health?pretty

# List indices
curl -k -u elastic:password https://localhost:9200/_cat/indices?v
```

## Troubleshooting

**Services won't start:**
```bash
# Increase vm.max_map_count
sudo sysctl -w vm.max_map_count=262144
```

**No data appearing:**
```bash
# Check Logstash logs
docker compose logs -f logstash

# Verify files are in correct directory
ls -la data/hayabusa/csv/
```

**Forgot password:**
- Edit `.env` file
- Run: `docker compose down && docker compose up -d`

## Next Steps

- Read [README.md](README.md) for full documentation
- Add more log types to `data/input/` directories
- Create visualizations and dashboards in Kibana
- Customize parsers in `logstash/pipeline/`
