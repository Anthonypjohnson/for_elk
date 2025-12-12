# Version Compatibility

## Current Configuration

This ELK stack is configured for **Elasticsearch 9.2.2** and is compatible with the entire Elastic Stack 9.x series.

## Version Information

### Elastic Stack Version
- **Current**: 9.2.2
- **Configured in**: [.env](.env:2)

### Compatible Versions

#### Elasticsearch 9.x Series ✅
- Fully compatible with all 9.x releases
- Security features unchanged from 8.x
- Built-in users remain the same (`elastic`, `kibana_system`, `logstash_system`)
- SSL/TLS configuration compatible

#### Elasticsearch 8.x Series ✅
- Configuration works with 8.15.0+ (tested)
- Change `STACK_VERSION=8.15.0` in [.env](.env:2)
- No other configuration changes needed

#### Elasticsearch 7.x Series ❌
- **Not compatible** - security model changed significantly in 8.x
- 7.x does not require SSL/TLS by default
- Different certificate setup process

## Key Compatibility Notes

### Security (8.x → 9.x)
Based on [Elasticsearch 9.0 release notes](https://www.elastic.co/guide/en/elastic-stack/9.0/release-notes-elasticsearch-9.0.0.html):

✅ **No breaking changes for security**
- Built-in users (`elastic`, `kibana_system`, `logstash_system`) unchanged
- Certificate-based authentication works the same
- User password setup API remains identical

### SSL/TLS Changes (8.x → 9.x)

From the [Elasticsearch 9.0 release notes](https://www.elastic.co/guide/en/elastic-stack/9.0/release-notes-elasticsearch-9.0.0.html):

⚠️ **TLS Protocol Changes**:
- **TLSv1.1 disabled by default** in 9.x (was enabled in 8.x if JDK supported it)
- **Default protocols**: TLSv1.3, TLSv1.2
- **TLS_RSA ciphers removed** from default supported ciphers on JDK 24+

✅ **Our Configuration**: Uses TLSv1.2+ by default, so no changes needed

### Docker Compose Setup

Based on [deviantony/docker-elk](https://github.com/deviantony/docker-elk) and [Elastic Docker documentation](https://www.elastic.co/docs/deploy-manage/deploy/self-managed/install-elasticsearch-docker-compose):

✅ **Certificate generation** using `elasticsearch-certutil` works identically in 8.x and 9.x
✅ **Health checks** remain the same
✅ **Environment variables** unchanged
✅ **Service dependencies** work the same way

## Testing Different Versions

### Switch to Elasticsearch 8.15.0
```bash
# Edit .env
STACK_VERSION=8.15.0

# Restart stack
docker compose down -v
docker compose up -d
```

### Switch to Elasticsearch 9.0.0
```bash
# Edit .env
STACK_VERSION=9.0.0

# Restart stack
docker compose down -v
docker compose up -d
```

### Switch to Elasticsearch 9.2.2 (Current)
```bash
# Edit .env
STACK_VERSION=9.2.2

# Restart stack
docker compose down -v
docker compose up -d
```

## Breaking Changes to Watch

### Elasticsearch 9.0
From [Elasticsearch release notes](https://www.elastic.co/guide/en/elastic-stack/9.0/release-notes-elasticsearch-9.0.0.html):

1. **TLS Protocol**: TLSv1.1 disabled by default
   - **Impact**: None (we use TLSv1.2+)

2. **TLS_RSA Ciphers**: Removed on JDK 24+
   - **Impact**: None for most deployments

3. **Deprecation Logs**: New format in 9.0
   - **Impact**: None (doesn't affect functionality)

### No Breaking Changes For:
- ✅ Built-in users
- ✅ Security API
- ✅ Certificate management
- ✅ Docker deployment
- ✅ Logstash/Kibana integration

## Version-Specific Features

### Elasticsearch 9.x Advantages
- Improved performance
- Better memory management
- Enhanced security features
- Continued support and updates

### Elasticsearch 8.x Advantages
- More mature (longer release cycle)
- Wider community adoption
- More third-party documentation

## Recommendation

**Use Elasticsearch 9.2.2** (current configuration):
- Latest features and security patches
- No compatibility issues with this stack
- Future-proof for continued Elastic development

## Upgrade Path

### From 8.x to 9.x
```bash
# 1. Backup data (optional - forensics data)
# 2. Update version in .env
STACK_VERSION=9.2.2

# 3. Remove old containers and volumes
docker compose down -v

# 4. Start with new version
docker compose up -d

# 5. Verify services
docker compose ps
docker compose logs -f elasticsearch
```

**Note**: Using `-v` removes data volumes. For production, back up data before upgrading.

## References

- [Elasticsearch 9.0 Release Notes](https://www.elastic.co/guide/en/elastic-stack/9.0/release-notes-elasticsearch-9.0.0.html)
- [Built-in Users Documentation](https://www.elastic.co/docs/deploy-manage/users-roles/cluster-or-deployment-auth/built-in-users)
- [Docker Compose Setup Guide](https://www.elastic.co/docs/deploy-manage/deploy/self-managed/install-elasticsearch-docker-compose)
- [Elasticsearch Breaking Changes](https://www.elastic.co/docs/release-notes/elasticsearch/breaking-changes)
- [SSL/TLS Configuration](https://www.elastic.co/blog/configuring-ssl-tls-and-https-to-secure-elasticsearch-kibana-beats-and-logstash)
