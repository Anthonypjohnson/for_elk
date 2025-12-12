# Forensic ELK Stack for Digital Investigations

A Docker Compose-based ELK (Elasticsearch, Logstash, Kibana) stack designed specifically for digital forensic investigations with built-in support for Hayabusa logs and other forensic log sources.

## Features

- **Hayabusa Integration**: Pre-configured parsers for Hayabusa CSV and JSON/JSONL output formats
- **SOF-ELK Inspired Architecture**: Numbered pipeline organization (0000-9999) for predictable processing
- **Extensible Parsers**: Example parsers for Sysmon, PowerShell, and other forensic log types
- **Security Enabled**: Elasticsearch 8.x security with authentication and SSL/TLS
- **GeoIP Enrichment**: Automatic geographic data enrichment for IP addresses
- **MITRE ATT&CK Mapping**: Built-in support for MITRE tactics and techniques from Hayabusa
- **Daily Indices**: Time-based index rotation for easier data management

## Prerequisites

- Docker Desktop (or Docker Engine + Docker Compose)
- Minimum 8GB RAM allocated to Docker (4GB minimum, 16GB recommended for large datasets)
- At least 20GB free disk space

## Quick Start

### 1. Clone or Download This Repository

```bash
cd /Users/anthony/Documents/elk
```

### 2. Configure Passwords

Edit the [.env](.env) file and change the default passwords:

```bash
ELASTIC_PASSWORD=your_secure_password_here
KIBANA_SYSTEM_PASSWORD=your_secure_password_here
LOGSTASH_INTERNAL_PASSWORD=your_secure_password_here
```

**WARNING**: Never commit the `.env` file to version control!

### 3. Increase Virtual Memory (Linux/macOS)

Elasticsearch requires increased virtual memory limits:

**macOS:**
```bash
# Temporary (until reboot)
sudo sysctl -w vm.max_map_count=262144

# Permanent
echo "kern.sysv.shmmax=262144" | sudo tee -a /etc/sysctl.conf
```

**Linux:**
```bash
# Temporary
sudo sysctl -w vm.max_map_count=262144

# Permanent
echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf
```

### 4. Start the Stack

```bash
docker compose up -d
```

This will:
- Start Elasticsearch, Logstash, and Kibana
- Initialize security certificates
- Set up user accounts and passwords
- Begin monitoring for log files

### 5. Verify Services are Running

```bash
# Check service status
docker compose ps

# View logs
docker compose logs -f elasticsearch
docker compose logs -f logstash
docker compose logs -f kibana
```

### 6. Access Kibana

Open your browser to: **http://localhost:5601**

- **Username**: `elastic`
- **Password**: (the password you set in `.env`)

## Using the Stack

### Adding Hayabusa Logs

#### CSV Format

1. Run Hayabusa to generate CSV output:
   ```bash
   hayabusa csv-timeline -d /path/to/evtx -o output.csv
   ```

2. Copy the CSV file to the input directory:
   ```bash
   cp output.csv data/hayabusa/csv/
   ```

3. Logstash will automatically detect and process the file

#### JSON Format

1. Run Hayabusa to generate JSON output:
   ```bash
   hayabusa json-timeline -d /path/to/evtx -o output.jsonl
   ```

2. Copy the JSON file to the input directory:
   ```bash
   cp output.jsonl data/hayabusa/json/
   ```

### Adding Other Forensic Logs

Place log files in the appropriate directories:

- **Sysmon logs**: `data/input/sysmon/`
- **PowerShell logs**: `data/input/powershell/`
- **Windows Event Logs**: `data/input/evtx/`
- **Network logs**: `data/input/network/`

### Viewing Data in Kibana

1. Navigate to **Management > Stack Management > Data Views** (or Index Patterns)
2. Create a data view:
   - **Name**: `Hayabusa Logs`
   - **Index pattern**: `hayabusa-*`
   - **Time field**: `@timestamp`
3. Go to **Discover** to explore your data
4. Use filters to search by:
   - **Level**: critical, high, medium, low, info
   - **EventID**: Specific Windows event IDs
   - **RuleTitle**: Hayabusa detection rule titles
   - **Computer**: Hostname/computer name
   - **MitreTactics**: MITRE ATT&CK tactics

## Directory Structure

```
elk/
├── docker-compose.yml          # Docker orchestration
├── .env                         # Passwords and configuration
├── .gitignore                   # Git ignore rules
├── README.md                    # This file
│
├── elasticsearch/
│   └── config/
│       └── elasticsearch.yml    # Elasticsearch config
│
├── logstash/
│   ├── config/
│   │   └── logstash.yml        # Logstash main config
│   ├── pipeline/                # Processing pipelines
│   │   ├── 0000-input-file.conf
│   │   ├── 1000-preprocess.conf
│   │   ├── 6000-filter-hayabusa-csv.conf
│   │   ├── 6001-filter-hayabusa-json.conf
│   │   ├── 6100-filter-sysmon.conf
│   │   ├── 8000-enrichment.conf
│   │   └── 9999-output-elasticsearch.conf
│   ├── patterns/                # Custom grok patterns
│   │   ├── hayabusa.pattern
│   │   └── forensics.pattern
│   └── templates/               # Elasticsearch index templates
│       └── hayabusa-template.json
│
├── kibana/
│   └── config/
│       └── kibana.yml          # Kibana config
│
└── data/                        # Input data (not in git)
    ├── input/                   # General forensic logs
    │   ├── evtx/
    │   ├── sysmon/
    │   ├── powershell/
    │   └── network/
    └── hayabusa/
        ├── csv/                 # Hayabusa CSV files
        └── json/                # Hayabusa JSON files
```

## Pipeline Architecture

The Logstash pipeline follows SOF-ELK numbering conventions:

- **0000-0999**: Input configurations (file inputs)
- **1000-1999**: Preprocessing (tagging, metadata extraction)
- **6000-6999**: Parsing/filtering (log-type specific parsers)
- **8000-8999**: Enrichment (GeoIP, threat intelligence)
- **9000-9999**: Output (Elasticsearch routing)

## Indices

Data is automatically routed to daily indices:

- **hayabusa-YYYY.MM.DD**: Hayabusa logs
- **forensics-sysmon-YYYY.MM.DD**: Sysmon logs
- **forensics-powershell-YYYY.MM.DD**: PowerShell logs
- **forensics-evtx-YYYY.MM.DD**: Windows Event Logs
- **forensics-network-YYYY.MM.DD**: Network logs
- **forensics-general-YYYY.MM.DD**: Other forensic logs

## Management Commands

### Start Services
```bash
docker compose up -d
```

### Stop Services
```bash
docker compose down
```

### View Logs
```bash
docker compose logs -f [service_name]
# Examples:
docker compose logs -f elasticsearch
docker compose logs -f logstash
docker compose logs -f kibana
```

### Restart Services
```bash
docker compose restart
```

### Check Service Health
```bash
# Elasticsearch cluster health
curl -k -u elastic:your_password https://localhost:9200/_cluster/health?pretty

# Logstash pipeline stats
curl http://localhost:9600/_node/stats/pipelines?pretty

# List indices
curl -k -u elastic:your_password https://localhost:9200/_cat/indices?v
```

### Clear All Data (WARNING: Destructive!)
```bash
docker compose down -v
```

This removes all volumes including indexed data!

## Troubleshooting

### Elasticsearch Won't Start

**Error**: `bootstrap checks failed`

**Solution**: Increase `vm.max_map_count`:
```bash
sudo sysctl -w vm.max_map_count=262144
```

### Yellow Cluster Health

**Cause**: Replicas configured on single-node cluster

**Solution**: The Hayabusa index template already sets `number_of_replicas: 0`. For other indices:
```bash
curl -k -u elastic:password -X PUT https://localhost:9200/_all/_settings \
  -H 'Content-Type: application/json' \
  -d '{"index": {"number_of_replicas": 0}}'
```

### Logstash Not Processing Files

**Check 1**: Verify file permissions
```bash
ls -la data/hayabusa/csv/
```

**Check 2**: Check Logstash logs
```bash
docker compose logs -f logstash
```

**Check 3**: Verify pipeline configuration
```bash
docker compose exec logstash bin/logstash --config.test_and_exit -f /usr/share/logstash/pipeline/
```

### EventID Mapping Errors

**Error**: `mapper_parsing_exception` for EventID field

**Cause**: EventID incorrectly mapped as `long` instead of `keyword`

**Solution**: Delete the index and re-ingest:
```bash
curl -k -u elastic:password -X DELETE https://localhost:9200/hayabusa-*
# Then re-add your Hayabusa files
```

### Cannot Login to Kibana

**Error**: Invalid username or password

**Solution**: Verify password in `.env` matches what you're using. For Elasticsearch 8.x+, you must use the `elastic` user (or create a new superuser).

### Parse Failures

Parse failures are logged to: `/usr/share/logstash/logs/parse-failures-YYYY-MM-DD.log`

To view:
```bash
docker compose exec logstash cat logs/parse-failures-$(date +%Y-%m-%d).log
```

## Adding New Log Parsers

### Step 1: Add Input Configuration

Edit [logstash/pipeline/0000-input-file.conf](logstash/pipeline/0000-input-file.conf):

```ruby
file {
  path => "/usr/share/logstash/data/input/mylogtype/*.log"
  start_position => "beginning"
  sincedb_path => "/dev/null"
  type => "mylogtype"
  tags => ["mylogtype", "forensics"]
  mode => "read"
}
```

### Step 2: Create Parser

Create `logstash/pipeline/61XX-filter-mylogtype.conf`:

```ruby
filter {
  if [type] == "mylogtype" {
    # Add your parsing logic here
    grok {
      match => { "message" => "%{PATTERN}" }
    }

    # Parse timestamp
    date {
      match => ["timestamp_field", "ISO8601"]
      target => "@timestamp"
    }

    # Add tags
    mutate {
      add_tag => ["successfully_parsed"]
    }
  }
}
```

### Step 3: Restart Logstash

```bash
docker compose restart logstash
```

Logstash will automatically pick up the new configuration file.

## Security Considerations

### Production Deployment

Before deploying to production:

1. **Change all default passwords** in `.env`
2. **Generate secure encryption keys** for Kibana in `kibana/config/kibana.yml`
3. **Enable TLS for HTTP** (currently using SSL only for internal communication)
4. **Implement role-based access control (RBAC)** for users
5. **Configure firewall rules** to restrict access
6. **Enable audit logging**
7. **Regular security updates** for Docker images

### Data Privacy

For forensic investigations involving sensitive data:
- Consider field-level security for PII
- Implement data retention policies
- Use anonymization where required
- Document chain of custody

## Performance Tuning

### For Large Datasets

Edit [.env](.env):

```bash
# Increase Elasticsearch heap
ES_JAVA_OPTS=-Xms4g -Xmx4g
ES_MEM_LIMIT=8GB

# Increase Logstash heap
LS_JAVA_OPTS=-Xms2g -Xmx2g
LS_MEM_LIMIT=4GB
```

Then restart:
```bash
docker compose down
docker compose up -d
```

### Index Lifecycle Management

For managing disk space with large datasets, configure ILM policies in Kibana or via API.

## Resources

### Official Documentation
- [Elastic Stack Documentation](https://www.elastic.co/guide/index.html)
- [Logstash Reference](https://www.elastic.co/guide/en/logstash/current/index.html)
- [Hayabusa GitHub](https://github.com/Yamato-Security/hayabusa)

### Reference Implementations
- [SOF-ELK GitHub](https://github.com/philhagen/sof-elk) - Forensic ELK patterns
- [Hayabusa Wiki: Importing to Elastic Stack](https://github.com/Yamato-Security/hayabusa/wiki/Importing-Results-Into-Elastic-Stack)

## License

This project configuration is provided as-is for forensic investigation purposes.

## Support

For issues or questions:
1. Check the [Troubleshooting](#troubleshooting) section
2. Review Elasticsearch/Logstash/Kibana logs
3. Consult the official Elastic Stack documentation
4. Review Hayabusa documentation for log format questions

## Contributing

To extend this stack:
1. Add new parsers in `logstash/pipeline/` following the numbering convention
2. Add custom grok patterns in `logstash/patterns/`
3. Create index templates for new log types
4. Update this README with usage instructions
