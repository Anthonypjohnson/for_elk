# Elastic Stack Licensing Guide

## Current Configuration

This ELK stack is configured with the **Basic License** (free) by default.

**Configuration Location**: [.env](.env:18)
```bash
LICENSE_TYPE=basic
```

## Available License Types

### Basic License (Default) - FREE ✅

**What's Included:**
- ✅ Full text search
- ✅ Logging and metrics
- ✅ APM (Application Performance Monitoring)
- ✅ Maps
- ✅ Security features (authentication, TLS/SSL)
- ✅ Kibana visualization
- ✅ Canvas
- ✅ 1GB Machine Learning per node
- ✅ Elastic Stack monitoring
- ✅ Index lifecycle management (ILM)

**What's NOT Included:**
- ❌ Alerting
- ❌ Advanced ML features
- ❌ Graph analytics
- ❌ SIEM detection rules
- ❌ Transforms

**Cost**: Free forever

**Recommended For**:
- Forensic analysis (this use case)
- Log aggregation and search
- Development and testing
- Small to medium deployments

### Trial License - FREE for 30 days

**What's Included:**
- ✅ Everything in Basic
- ✅ Alerting
- ✅ Advanced Machine Learning
- ✅ Graph analytics
- ✅ SIEM features
- ✅ Advanced security (RBAC, field/document level security)
- ✅ Cross-cluster replication
- ✅ Transforms
- ✅ Searchable snapshots

**Duration**: 30 days, then reverts to Basic

**Recommended For**:
- Evaluating premium features
- Testing alerting capabilities
- Exploring ML anomaly detection

### Gold License - PAID

**Additional Features:**
- ✅ Alerting
- ✅ SQL access
- ✅ Canvas reports
- ✅ Custom branding

**Cost**: Paid subscription required

### Platinum License - PAID

**Additional Features:**
- ✅ Everything in Gold
- ✅ Advanced Machine Learning
- ✅ Graph analytics
- ✅ Advanced security features

**Cost**: Paid subscription required

### Enterprise License - PAID

**Additional Features:**
- ✅ Everything in Platinum
- ✅ Searchable snapshots
- ✅ SIEM detection rules
- ✅ Extended data retention
- ✅ Support for >100 nodes

**Cost**: Paid subscription required

## Changing the License Type

### Method 1: Environment Variable (Recommended)

Edit [.env](.env:18):

```bash
# For Basic (default, free)
LICENSE_TYPE=basic

# For 30-day Trial (all features)
LICENSE_TYPE=trial

# For Paid licenses (requires subscription)
LICENSE_TYPE=gold
LICENSE_TYPE=platinum
LICENSE_TYPE=enterprise
```

Then restart:
```bash
docker compose down
docker compose up -d
```

### Method 2: Via Kibana UI

1. Navigate to **Stack Management** → **License Management**
2. Click **Update license**
3. Upload your license file (for paid subscriptions)

### Method 3: Via API

```bash
# Check current license
curl -k -u elastic:changeme https://localhost:9200/_license

# Start trial (one-time, 30 days)
curl -k -u elastic:changeme -X POST https://localhost:9200/_license/start_trial?acknowledge=true

# Upload paid license (requires license.json file)
curl -k -u elastic:changeme -X PUT https://localhost:9200/_license \
  -H 'Content-Type: application/json' \
  -d @license.json
```

## Feature Comparison Matrix

| Feature | Basic | Trial | Gold | Platinum | Enterprise |
|---------|-------|-------|------|----------|------------|
| **Core Search & Analytics** |
| Full-text search | ✅ | ✅ | ✅ | ✅ | ✅ |
| Aggregations | ✅ | ✅ | ✅ | ✅ | ✅ |
| Geo search | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Visualization** |
| Kibana dashboards | ✅ | ✅ | ✅ | ✅ | ✅ |
| Canvas | ✅ | ✅ | ✅ | ✅ | ✅ |
| Maps | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Security** |
| TLS/SSL encryption | ✅ | ✅ | ✅ | ✅ | ✅ |
| Role-based access | ✅ | ✅ | ✅ | ✅ | ✅ |
| Field-level security | ❌ | ✅ | ❌ | ✅ | ✅ |
| Document-level security | ❌ | ✅ | ❌ | ✅ | ✅ |
| **Management** |
| Index lifecycle (ILM) | ✅ | ✅ | ✅ | ✅ | ✅ |
| Monitoring | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Advanced Features** |
| Alerting | ❌ | ✅ | ✅ | ✅ | ✅ |
| Machine Learning | 1GB/node | ✅ | ❌ | ✅ | ✅ |
| Graph analytics | ❌ | ✅ | ❌ | ✅ | ✅ |
| SIEM | ❌ | ✅ | ❌ | ❌ | ✅ |
| Transforms | ❌ | ✅ | ❌ | ✅ | ✅ |
| Searchable snapshots | ❌ | ✅ | ❌ | ❌ | ✅ |

## For Forensic Investigations

### Recommended: Basic License (Current)

For digital forensic investigations, the **Basic license is sufficient** because:

✅ **You Get:**
- Full search capabilities across all forensic data
- Kibana dashboards for timeline analysis
- Index lifecycle management for data retention
- Security features (important for chain of custody)
- Maps for IP geolocation
- 1GB Machine Learning per node (enough for anomaly detection)

❌ **You Don't Need:**
- Alerting (forensics is retrospective analysis)
- Advanced ML (basic ML covers anomaly detection)
- Graph analytics (nice-to-have but not essential)

### When to Upgrade to Trial

Consider the 30-day trial if you want to:
- **Test alerting** for automated detection in ongoing investigations
- **Explore advanced ML** for sophisticated pattern detection
- **Use SIEM features** for threat hunting
- **Evaluate transforms** for data aggregation

### When to Consider Paid Licenses

Consider paid licenses if:
- Running a **SOC or threat hunting team** (Enterprise for SIEM)
- Need **advanced RBAC** for multi-team access (Platinum)
- Require **long-term data retention** with searchable snapshots (Enterprise)
- Need **commercial support** from Elastic

## License Expiration

### Basic License
- **Never expires**
- No feature loss
- Free forever

### Trial License
- **Expires after 30 days**
- Automatically reverts to Basic
- All data remains accessible
- Premium features become unavailable

### Paid Licenses
- Expire based on subscription
- Grace period before feature loss
- Notifications in Kibana before expiration

## Checking Your Current License

### Via Kibana
Navigate to **Stack Management** → **License Management**

### Via API
```bash
curl -k -u elastic:changeme https://localhost:9200/_license | jq
```

### Expected Output (Basic):
```json
{
  "license": {
    "status": "active",
    "uid": "...",
    "type": "basic",
    "issue_date": "...",
    "issue_date_in_millis": 0,
    "max_nodes": null,
    "issued_to": "forensics-elk",
    "issuer": "elasticsearch",
    "start_date_in_millis": -1
  }
}
```

## Compliance and Legal

### Basic License
- **License**: Elastic License 2.0 (ELv2) and Server Side Public License (SSPL)
- **Commercial use**: Allowed
- **Modification**: Allowed
- **Restrictions**: Cannot offer as a managed service

### Usage in Forensics
- ✅ Internal investigations
- ✅ Law enforcement use
- ✅ Corporate security operations
- ✅ Evidence analysis
- ❌ Offering ELK as a service to others (requires commercial agreement)

## Cost Estimation (Paid Licenses)

For reference, paid Elastic subscriptions are typically priced:
- **Gold**: ~$95/month per node
- **Platinum**: ~$125/month per node
- **Enterprise**: ~$175/month per node

**Note**: Prices vary by region and subscription term. Contact Elastic for accurate pricing.

For a single-node forensic setup, Basic is free and sufficient.

## Frequently Asked Questions

### Can I use Basic for commercial forensic work?
Yes, the Basic license allows commercial use for internal purposes.

### Do I lose my data if trial expires?
No, all data remains accessible. Only premium features become unavailable.

### Can I switch between license types?
Yes, you can switch at any time. Premium features activate/deactivate based on license.

### Is there a difference between self-managed and cloud licensing?
Yes, this guide covers self-managed (Docker). Elastic Cloud has different pricing and features.

### What happens if I don't configure a license?
Elasticsearch uses Basic license by default.

## References

- [Elastic Subscriptions](https://www.elastic.co/subscriptions)
- [License Management API](https://www.elastic.co/guide/en/elasticsearch/reference/current/licensing-apis.html)
- [Elastic License FAQ](https://www.elastic.co/pricing/faq/licensing)
- [Elastic License 2.0 (ELv2)](https://www.elastic.co/licensing/elastic-license)

## Configuration Files

- License type setting: [.env](.env:18)
- Elasticsearch config: [elasticsearch.yml](elasticsearch/config/elasticsearch.yml:42)
- Docker compose: [docker-compose.yml](docker-compose.yml:85)
