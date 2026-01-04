# WebL8 AI Logging System

Comprehensive logging system with automatic rotation, multiple log types, and configuration integration.

## 🚀 Features

- **Automatic Daily Rotation** - Logs rotate at midnight automatically
- **Multiple Log Types** - Application, error, access, and security logs
- **Configuration Integration** - Uses settings from webl8ai.conf
- **Platform Support** - Works on Linux and Windows
- **Structured Logging** - JSON format for access and security logs
- **Fallback Handling** - Automatically uses user directory if no admin access

## 📍 Log File Locations

### Linux/Unix
```
/var/log/webl8ai/
├── webl8ai.log         # Main application log
├── error.log           # Error-only log
├── access.log          # API access log (JSON)
└── security.log        # Security events (JSON)
```

### Windows
```
C:\ProgramData\Webl8ai\logs\
├── webl8ai.log         # Main application log
├── error.log           # Error-only log
├── access.log          # API access log (JSON)
└── security.log        # Security events (JSON)
```

### User Fallback (No Admin)
- Linux: `~/.local/share/webl8ai/logs/`
- Windows: `%LOCALAPPDATA%\Webl8ai\logs\`

## ⚙️ Configuration

The logging system uses the `[logging]` section in webl8ai.conf:

```ini
[logging]
level = INFO                # Log level (DEBUG, INFO, WARNING, ERROR, CRITICAL)
file_logging = true         # Enable file logging
console_logging = true      # Enable console output
log_rotation = daily        # Rotation interval (hourly, daily, weekly, monthly)
max_log_files = 30          # Keep last 30 rotated files
log_format = %(asctime)s - %(name)s - %(levelname)s - %(message)s
```

## 📝 Log Types

### 1. Application Log (webl8ai.log)
General application events, info, warnings, and errors.

```
2024-01-03 10:15:32 - webl8ai.api - INFO - Analyzing domain: example.com with model: claude-3.5-sonnet
2024-01-03 10:15:35 - webl8ai.api - INFO - Analysis complete for example.com: status=success, time=2.50s, category=business, risk=low
```

### 2. Error Log (error.log)
Only ERROR and CRITICAL level messages.

```
2024-01-03 10:20:15 - webl8ai.api - ERROR - Analysis failed for test.com: Connection timeout
```

### 3. Access Log (access.log)
JSON-formatted API access records for audit trail.

```json
{
  "timestamp": "2024-01-03T10:15:35.123Z",
  "domain": "example.com",
  "customer_id": "CUST_20240103_A1B2",
  "api_key": "sk_abc123****xyz9",
  "status_code": 200,
  "response_time_ms": 2500,
  "error": null
}
```

### 4. Security Log (security.log)
JSON-formatted security events.

```json
{
  "timestamp": "2024-01-03T10:25:00.456Z",
  "event_type": "rate_limit_exceeded",
  "severity": "WARNING",
  "details": {
    "customer_id": "CUST_20240103_A1B2",
    "limit_type": "per_minute"
  }
}
```

## 🔄 Log Rotation

### Automatic Rotation
Logs rotate automatically based on configuration:

- **Daily** (default): Rotates at midnight
- **Hourly**: Rotates every hour
- **Weekly**: Rotates every Monday
- **Monthly**: Rotates approximately every 30 days

### Rotated File Names
```
webl8ai.log           # Current log
webl8ai.log.20240102  # Yesterday's log
webl8ai.log.20240101  # Day before
...
```

### Retention
- Keeps last 30 log files by default (configurable)
- Older files are automatically deleted

## 🛠️ Usage

### Basic Usage (Automatic)
The logging system initializes automatically when using webl8ai:

```python
# Logging is automatically configured
python webl8ai.py example.com
```

### Manual Integration

```python
from logging_system import initialize_logging, get_logger

# Initialize with configuration
from config_manager import get_config
config = get_config()
logging_system = initialize_logging(config)

# Get a logger for your module
logger = get_logger(__name__)

# Use the logger
logger.info("Starting process")
logger.warning("This is a warning")
logger.error("An error occurred", exc_info=True)
```

### Specialized Loggers

```python
from logging_system import APILogger, CacheLogger, CustomerLogger

# API logging
api_logger = APILogger()
api_logger.log_request("example.com", "claude-3.5-sonnet", start_time)
api_logger.log_response("example.com", "success", 2.5, "business", "low")
api_logger.log_error("example.com", "Connection timeout")

# Cache logging
cache_logger = CacheLogger()
cache_logger.log_hit("example.com:claude")
cache_logger.log_miss("test.com:claude")
cache_logger.log_set("example.com:claude", ttl=24)

# Customer logging
customer_logger = CustomerLogger()
customer_logger.log_customer_created("CUST_123", "Acme Corp")
customer_logger.log_rate_limit_exceeded("CUST_123", "per_minute")
customer_logger.log_authentication_failure("sk_invalid_key")
```

## 📊 Log Management

### View Statistics
```bash
python logging_system.py --stats

=== Logging Statistics ===
Log Directory: /var/log/webl8ai
Active Handlers: console, app, error, access, security
Active Loggers: 5
Log Files:
  webl8ai.log: 12.34 MB
  error.log: 0.56 MB
  access.log: 8.90 MB
```

### Clean Old Logs
```bash
# Delete logs older than 90 days
python logging_system.py --cleanup 90

Deleted 15 old log files
```

### Tail Logs
```bash
# Follow a specific log
python logging_system.py --tail app
python logging_system.py --tail error
python logging_system.py --tail access
```

### Test Logging
```bash
# Test all log levels and types
python logging_system.py --test

✓ Logging test complete - check log files
```

## 🔍 Log Analysis

### Search Logs
```bash
# Find errors in the last hour
grep "ERROR" /var/log/webl8ai/webl8ai.log | tail -100

# Find specific domain
grep "example.com" /var/log/webl8ai/access.log

# Find rate limit events
grep "rate_limit" /var/log/webl8ai/security.log
```

### Parse JSON Logs
```python
import json

# Read access log
with open('/var/log/webl8ai/access.log', 'r') as f:
    for line in f:
        entry = json.loads(line)
        if entry['status_code'] != 200:
            print(f"Error: {entry['domain']} - {entry['error']}")
```

### Analyze with jq
```bash
# Get average response time
cat /var/log/webl8ai/access.log | jq '.response_time_ms' | awk '{sum+=$1} END {print sum/NR}'

# Count errors by type
cat /var/log/webl8ai/access.log | jq -r '.error' | sort | uniq -c

# Find high-risk domains
cat /var/log/webl8ai/webl8ai.log | grep "risk=high"
```

## 🎯 Log Levels

| Level | Value | When to Use |
|-------|-------|------------|
| DEBUG | 10 | Detailed diagnostic info |
| INFO | 20 | General informational messages |
| WARNING | 30 | Warning messages |
| ERROR | 40 | Error messages |
| CRITICAL | 50 | Critical problems |

### Setting Log Level
```bash
# Via configuration
python config_manager.py --set logging.level DEBUG

# Via environment (temporary)
LOG_LEVEL=DEBUG python webl8ai.py example.com
```

## 📈 Monitoring Integration

### Logstash/Elasticsearch
```json
{
  "input": {
    "file": {
      "path": "/var/log/webl8ai/*.log",
      "codec": "json",
      "type": "webl8ai"
    }
  }
}
```

### Prometheus Metrics
Access log can be parsed for metrics:
- Request rate
- Response times
- Error rate
- Category distribution

### Grafana Dashboard
Create dashboards using:
- Log volume over time
- Error rate trends
- Response time percentiles
- Top domains by requests

## 🔒 Security Considerations

1. **Log Permissions**
   ```bash
   # Restrict log access
   chmod 640 /var/log/webl8ai/*.log
   chown root:webl8ai /var/log/webl8ai/*.log
   ```

2. **Sensitive Data**
   - API keys are automatically masked
   - Customer IDs are logged (not PII)
   - No passwords in logs

3. **Log Rotation Security**
   - Old logs are deleted, not just compressed
   - Configurable retention period

## 🐳 Docker Logging

### Docker Configuration
```dockerfile
# Mount log directory
docker run -v /var/log/webl8ai:/var/log/webl8ai webl8ai

# Or use Docker logging driver
docker run --log-driver=json-file \
  --log-opt max-size=10m \
  --log-opt max-file=3 \
  webl8ai
```

### Docker Compose
```yaml
services:
  webl8ai:
    image: webl8ai
    volumes:
      - webl8ai-logs:/var/log/webl8ai
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"

volumes:
  webl8ai-logs:
```

## 🆘 Troubleshooting

### Logs Not Created
```bash
# Check permissions
ls -la /var/log/webl8ai/

# Check configuration
python config_manager.py --get logging.file_logging

# Test with debug
LOG_LEVEL=DEBUG python webl8ai.py --test
```

### Logs Not Rotating
```bash
# Check rotation setting
python config_manager.py --get logging.log_rotation

# Manual rotation test
python logging_system.py --test
# Wait until midnight or change system time
```

### Disk Space Issues
```bash
# Check log sizes
du -sh /var/log/webl8ai/*

# Clean old logs
python logging_system.py --cleanup 7

# Reduce retention
python config_manager.py --set logging.max_log_files 7
```

## 📚 Best Practices

1. **Use Appropriate Log Levels**
   - DEBUG for development only
   - INFO for production
   - Avoid excessive logging

2. **Structured Logging**
   - Use JSON for machine-readable logs
   - Include context in log messages

3. **Regular Cleanup**
   - Set appropriate retention
   - Monitor disk space
   - Archive important logs

4. **Security**
   - Never log sensitive data
   - Mask API keys and passwords
   - Restrict file permissions

5. **Performance**
   - Async logging for high volume
   - Batch log writes
   - Use appropriate rotation interval

---

The logging system provides comprehensive monitoring and debugging capabilities while maintaining security and performance.
