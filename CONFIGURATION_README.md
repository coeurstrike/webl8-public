# WebL8 AI Configuration System

Centralized configuration management for all WebL8 AI modules using a single configuration file.

## 📍 Configuration File Location

The system uses a platform-specific configuration file:

- **Linux/Unix**: `/etc/webl8ai/webl8ai.conf`
- **Windows**: `C:\ProgramData\Webl8ai\webl8ai.conf`
- **User Fallback**: `~/.config/webl8ai/webl8ai.conf` (Linux) or `%LOCALAPPDATA%\Webl8ai\webl8ai.conf` (Windows)

## 🚀 Quick Setup

### 1. Install the System

```bash
# Run the installer
sudo python install.py

# Or without sudo (installs to user directory)
python install.py
```

### 2. Configure API Key

```bash
# Set your OpenRouter API key
python config_manager.py --set api.openrouter_api_key YOUR_KEY_HERE
```

### 3. Enable Features

```bash
# Enable caching
python config_manager.py --set cache.enable_cache true

# Enable customer management
python config_manager.py --set customer_management.enabled true

# Enable admin panel
python config_manager.py --set admin_panel.enabled true
```

### 4. Run the System

```bash
# Use the configured system
python webl8ai.py example.com
```

## 📝 Configuration File Structure

The configuration file uses INI format with the following sections:

### [general]
General application settings
```ini
app_name = WebL8 AI Website Categorizer
version = 2.0
debug = false
```

### [api]
OpenRouter API configuration
```ini
openrouter_api_key = YOUR_KEY_HERE
default_model = anthropic/claude-3.5-sonnet
timeout = 30
max_retries = 3
verify_ssl = true
```

### [dns]
DNS validation settings
```ini
enable_validation = true
check_mx_records = true
dns_timeout = 5
tcp_timeout = 5
check_ports = 443,80
```

### [cache]
Database caching configuration
```ini
enable_cache = false
cache_type = postgresql  # Options: postgresql, redis, mongodb
cache_ttl_hours = 24
max_cache_size_mb = 1000
```

### [postgresql]
PostgreSQL connection settings (if cache_type = postgresql)
```ini
host = localhost
port = 5432
database = website_cache
user = cacheuser
password = cachepass
pool_size = 5
```

### [customer_management]
Customer management system
```ini
enabled = false
database_file = customers.db
require_api_key = false
allow_anonymous = true
default_trial_days = 30
```

### [rate_limits]
Default rate limiting settings
```ini
default_per_second = 10
default_per_minute = 100
default_per_hour = 1000
default_per_day = 10000
default_per_month = 100000
anonymous_per_hour = 100
```

### [admin_panel]
Web admin panel settings
```ini
enabled = false
host = 0.0.0.0
port = 5000
secret_key = (auto-generated)
session_timeout = 3600
default_username = admin
default_password = admin
```

### [logging]
Logging configuration
```ini
level = INFO
file_logging = true
console_logging = true
log_rotation = daily
max_log_files = 30
```

## 🛠️ Configuration Management Tool

The `config_manager.py` tool provides easy configuration management:

### View Configuration

```bash
# Show all configuration
python config_manager.py --show

# Show configuration status
python config_manager.py --status

# Get specific value
python config_manager.py --get api.openrouter_api_key
```

### Modify Configuration

```bash
# Set a value
python config_manager.py --set section.key value

# Examples:
python config_manager.py --set api.openrouter_api_key sk_xxxxx
python config_manager.py --set cache.enable_cache true
python config_manager.py --set postgresql.host db.example.com
python config_manager.py --set customer_management.enabled true
python config_manager.py --set rate_limits.default_per_day 50000
```

### Create Default Configuration

```bash
# Create default configuration file
python config_manager.py --create-default
```

## 🔄 Environment Variable Overrides

Environment variables can override configuration file settings:

| Environment Variable | Configuration Setting |
|---------------------|----------------------|
| `WEBL8AI_API_KEY` | api.openrouter_api_key |
| `OPENROUTER_API_KEY` | api.openrouter_api_key |
| `WEBL8AI_MODEL` | api.default_model |
| `WEBL8AI_CACHE_ENABLED` | cache.enable_cache |
| `WEBL8AI_CACHE_TYPE` | cache.cache_type |
| `POSTGRES_HOST` | postgresql.host |
| `POSTGRES_PORT` | postgresql.port |
| `POSTGRES_DB` | postgresql.database |
| `POSTGRES_USER` | postgresql.user |
| `POSTGRES_PASSWORD` | postgresql.password |
| `REDIS_HOST` | redis.host |
| `REDIS_PORT` | redis.port |
| `WEBL8AI_CUSTOMER_ENABLED` | customer_management.enabled |
| `WEBL8AI_ADMIN_ENABLED` | admin_panel.enabled |
| `FLASK_SECRET_KEY` | admin_panel.secret_key |

Example:
```bash
# Override API key temporarily
WEBL8AI_API_KEY=sk_temp_key python webl8ai.py example.com
```

## 🎯 Usage Examples

### Basic Usage (No Optional Features)

```ini
# webl8ai.conf - Basic configuration
[api]
openrouter_api_key = YOUR_KEY

[cache]
enable_cache = false

[customer_management]
enabled = false
```

```bash
python webl8ai.py example.com
```

### With PostgreSQL Caching

```ini
# webl8ai.conf - With caching
[api]
openrouter_api_key = YOUR_KEY

[cache]
enable_cache = true
cache_type = postgresql
cache_ttl_hours = 24

[postgresql]
host = localhost
port = 5432
database = website_cache
user = cacheuser
password = cachepass
```

### With Customer Management

```ini
# webl8ai.conf - With customer management
[api]
openrouter_api_key = YOUR_KEY

[customer_management]
enabled = true
database_file = customers.db

[rate_limits]
default_per_second = 10
default_per_day = 10000

[admin_panel]
enabled = true
port = 5000
```

```bash
# Start admin panel
python customer_admin.py

# Use with customer API key
python webl8ai.py example.com --customer-key sk_customer_xxxxx
```

### Full Enterprise Setup

```ini
# webl8ai.conf - Full enterprise configuration
[api]
openrouter_api_key = YOUR_KEY
default_model = anthropic/claude-3.5-sonnet

[cache]
enable_cache = true
cache_type = postgresql
cache_ttl_hours = 48

[postgresql]
host = db.company.com
port = 5432
database = webl8ai_cache
user = webl8ai_user
password = secure_password

[customer_management]
enabled = true
database_file = /var/lib/webl8ai/customers.db
require_api_key = true
allow_anonymous = false

[rate_limits]
default_per_second = 20
default_per_minute = 200
default_per_hour = 2000
default_per_day = 20000
default_per_month = 200000

[admin_panel]
enabled = true
host = 0.0.0.0
port = 8080
secret_key = (auto-generated-secure-key)

[logging]
level = INFO
file_logging = true
log_rotation = daily
max_log_files = 90

[security]
enable_cors = true
allowed_origins = https://app.company.com
enable_rate_limiting = true
max_request_size_mb = 10
```

## 📂 Directory Structure

### Linux/Unix

```
/etc/webl8ai/
├── webl8ai.conf         # Main configuration file
└── webl8ai.conf.backup  # Backup configuration

/var/lib/webl8ai/
├── customers.db         # Customer database
└── cache/               # Cache data

/var/log/webl8ai/
├── webl8ai.log         # Application logs
├── access.log          # API access logs
└── error.log           # Error logs

/var/cache/webl8ai/     # Temporary cache files
```

### Windows

```
C:\ProgramData\Webl8ai\
├── webl8ai.conf        # Main configuration file
├── customers.db        # Customer database
├── logs\               # Log files
├── data\               # Application data
└── cache\              # Cache files
```

## 🔧 Programmatic Access

### Python Integration

```python
from config_manager import get_config

# Get configuration instance
config = get_config()

# Access configuration values
api_key = config.get('api', 'openrouter_api_key')
cache_enabled = config.get_bool('cache', 'enable_cache')
rate_limits = config.get_section('rate_limits')

# Get pre-configured settings
api_config = config.get_api_config()
cache_config = config.get_cache_config()
customer_config = config.get_customer_config()

# Modify configuration
config.set('api', 'default_model', 'gpt-4')
config.save_config()
```

### Using Configured Categorizer

```python
from webl8ai import ConfiguredWebsiteCategorizer

# Initialize with configuration
categorizer = ConfiguredWebsiteCategorizer()

# Analyze website (uses all configured features)
result = categorizer.analyze("example.com")

# With customer API key (if customer management enabled)
result = categorizer.analyze(
    "example.com",
    customer_api_key="sk_customer_xxx"
)
```

## 🐳 Docker Configuration

For Docker deployments, mount the configuration file:

```dockerfile
# Dockerfile
FROM python:3.9

COPY . /app
WORKDIR /app

RUN pip install -r requirements.txt

# Create config directory
RUN mkdir -p /etc/webl8ai

# Use configuration
CMD ["python", "webl8ai.py", "--config", "/etc/webl8ai/webl8ai.conf"]
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  webl8ai:
    build: .
    volumes:
      - ./webl8ai.conf:/etc/webl8ai/webl8ai.conf:ro
      - webl8ai-data:/var/lib/webl8ai
      - webl8ai-logs:/var/log/webl8ai
    environment:
      - WEBL8AI_API_KEY=${OPENROUTER_API_KEY}

volumes:
  webl8ai-data:
  webl8ai-logs:
```

## 🔒 Security Considerations

1. **Configuration File Permissions**
   ```bash
   # Linux: Restrict access to config file
   sudo chmod 600 /etc/webl8ai/webl8ai.conf
   sudo chown root:root /etc/webl8ai/webl8ai.conf
   ```

2. **Sensitive Values**
   - Store passwords and API keys securely
   - Consider using environment variables for production
   - Rotate API keys regularly

3. **Admin Panel Security**
   - Change default admin password immediately
   - Use HTTPS in production
   - Configure firewall rules for port access

## 🆘 Troubleshooting

### Configuration Not Found

```bash
# Check configuration location
python config_manager.py --status

# Create default configuration
python config_manager.py --create-default
```

### Permission Denied

```bash
# Linux: Use sudo for system directories
sudo python install.py

# Or use user directory
python install.py  # Will use ~/.config/webl8ai/
```

### Module Not Using Configuration

```python
# Ensure configuration is initialized
from config_manager import initialize_config
config = initialize_config()

# Check if features are enabled
print(f"Cache enabled: {config.is_cache_enabled()}")
print(f"Customer management: {config.is_customer_management_enabled()}")
```

## 📚 Migration from Environment Variables

If migrating from environment variables:

```bash
# Export existing environment to config
python config_manager.py --set api.openrouter_api_key "$OPENROUTER_API_KEY"
python config_manager.py --set postgresql.host "$POSTGRES_HOST"
python config_manager.py --set postgresql.password "$POSTGRES_PASSWORD"

# Verify configuration
python config_manager.py --show
```

## ✅ Benefits of Centralized Configuration

1. **Single Source of Truth** - All settings in one place
2. **Platform Compatibility** - Works on Linux and Windows
3. **Easy Management** - Simple CLI tool for configuration
4. **Override Support** - Environment variables can override
5. **Modular Control** - Enable/disable features easily
6. **Production Ready** - Proper paths for system deployment
7. **User Friendly** - Fallback to user directory if needed
8. **Secure** - Sensitive data in protected location

---

The centralized configuration system makes WebL8 AI easier to deploy, manage, and maintain across different environments and platforms.
