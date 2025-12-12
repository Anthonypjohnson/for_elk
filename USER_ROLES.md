# Elasticsearch Built-in Users

This ELK stack uses Elasticsearch's built-in users with proper role separation.

## User Accounts

### 1. elastic (Superuser)
- **Role**: Superuser with full cluster access
- **Used for**:
  - Logging into Kibana web interface
  - Logstash pipeline operations (input/filter/output)
  - Administrative tasks
- **Password**: Set in `.env` as `ELASTIC_PASSWORD`
- **Security Note**: In production, create dedicated users for specific tasks instead of using elastic everywhere

### 2. kibana_system
- **Role**: Service account for Kibana
- **Used for**: Kibana's internal connection to Elasticsearch
- **Password**: Set in `.env` as `KIBANA_SYSTEM_PASSWORD`
- **Important**: Cannot be used to log into Kibana UI (use `elastic` for that)

### 3. logstash_system
- **Role**: Service account for Logstash monitoring
- **Used for**: Logstash monitoring data sent to Elasticsearch
- **Password**: Set in `.env` as `LOGSTASH_INTERNAL_PASSWORD`
- **Important**: Only for monitoring, NOT for pipeline data ingestion

## Configuration Locations

### docker-compose.yml
```yaml
# Setup service sets passwords for:
- kibana_system
- logstash_system

# Logstash uses:
- elastic (for pipeline output)
- logstash_system (for monitoring)

# Kibana uses:
- kibana_system (for service connection)
```

### logstash/config/logstash.yml
```yaml
# Monitoring configuration
xpack.monitoring.elasticsearch.username: "logstash_system"
xpack.monitoring.elasticsearch.password: "${LOGSTASH_INTERNAL_PASSWORD}"
```

### logstash/pipeline/9999-output-elasticsearch.conf
```ruby
# Pipeline output uses elastic superuser
elasticsearch {
  user => "elastic"
  password => "${ELASTIC_PASSWORD}"
}
```

## Production Best Practices

For production deployments, create custom users with specific roles:

### Create a Logstash Writer User
```bash
# In Kibana Dev Tools or via API
POST /_security/user/logstash_writer
{
  "password" : "your_secure_password",
  "roles" : [ "logstash_writer" ],
  "full_name" : "Logstash Writer",
  "email" : "logstash@example.com"
}

# Create role with necessary permissions
POST /_security/role/logstash_writer
{
  "cluster": ["monitor", "manage_index_templates", "manage_ilm"],
  "indices": [
    {
      "names": [ "hayabusa-*", "forensics-*" ],
      "privileges": ["create_index", "write", "read", "manage"]
    }
  ]
}
```

Then update [logstash/pipeline/9999-output-elasticsearch.conf](logstash/pipeline/9999-output-elasticsearch.conf:1) to use `logstash_writer` instead of `elastic`.

## Why This Matters

**Security Principle: Least Privilege**

- **kibana_system**: Only has permissions Kibana needs (can't access user data)
- **logstash_system**: Only has permissions to write monitoring data
- **elastic**: Full access (should be restricted to admins only)

Using the superuser `elastic` for Logstash pipelines in this forensic setup is acceptable because:
1. It's a single-purpose forensic analysis system
2. All services run in Docker on the same host
3. Easier initial setup and troubleshooting

For production or multi-tenant environments, always create dedicated service accounts with minimal required permissions.

## References

- [Elasticsearch Built-in Users Documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/built-in-users.html)
- [Logstash Security Configuration](https://www.elastic.co/guide/en/logstash/current/ls-security.html)
