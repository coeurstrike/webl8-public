# Database Caching for Website Categorizer

A modular database caching system for the Website Categorizer with PostgreSQL as the default backend and extensibility for Redis and MongoDB.

## 🚀 Quick Start

### 1. Setup PostgreSQL
```bash
# Run the automated setup script
chmod +x setup_postgresql.sh
./setup_postgresql.sh

# Or use Docker
docker run -d \
  --name postgres-cache \
  -e POSTGRES_PASSWORD=postgres \
  -e POSTGRES_DB=website_cache \
  -p 5432:5432 \
  postgres:15-alpine
```

### 2. Install Dependencies
```bash
pip install -r requirements.txt
```

### 3. Configure Environment
```bash
# Copy and edit .env file
cp .env.example .env
# Add your OpenRouter API key and PostgreSQL credentials
```

### 4. Use Cached Categorizer
```bash
# Analyze with caching
python cached_categorizer.py example.com --api-key YOUR_KEY

# View cache statistics
python cached_categorizer.py --stats
```

## 📦 Architecture

### Modular Design
```
database_cache.py
├── CacheInterface (Abstract Base Class)
│   ├── connect()
│   ├── disconnect()
│   ├── get()
│   ├── set()
│   ├── delete()
│   ├── get_by_domain()
│   ├── get_by_category()
│   ├── get_high_risk()
│   ├── search_phishing()
│   └── get_stats()
│
├── PostgreSQLCache (Default Implementation)
│   └── Full JSONB support with indexing
│
├── RedisCache (Stub - Future)
│   └── Placeholder for Redis implementation
│
└── MongoDBCache (Stub - Future)
    └── Placeholder for MongoDB implementation
```

### Database Schema (PostgreSQL)

```sql
-- Main cache table
CREATE TABLE website_cache (
    cache_key VARCHAR(255) PRIMARY KEY,
    domain VARCHAR(255) NOT NULL,
    model VARCHAR(100),
    analysis_data JSONB NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    expires_at TIMESTAMP NOT NULL,
    access_count INTEGER DEFAULT 0,
    last_accessed TIMESTAMP
);

-- Optimized indexes for JSON queries
CREATE INDEX idx_category ON website_cache((analysis_data->>'primary_category'));
CREATE INDEX idx_risk ON website_cache((analysis_data->'security_assessment'->>'risk_level'));
CREATE INDEX idx_phishing ON website_cache((analysis_data->'phishing_indicators'->>'likely_phishing'));
```

## 🔧 Usage Examples

### Basic Usage

```python
from database_cache import CacheManager
from cached_categorizer import CachedWebsiteCategorizer

# Initialize with PostgreSQL (default)
categorizer = CachedWebsiteCategorizer(
    api_key="your_openrouter_key",
    model="anthropic/claude-3.5-sonnet",
    cache_type="postgresql",
    cache_ttl_hours=24
)

# Analyze a domain (checks cache first)
result = categorizer.analyze_website("example.com")

# Force fresh analysis
result = categorizer.analyze_website("example.com", force_refresh=True)

# Get cache statistics
stats = categorizer.get_cache_stats()
print(f"Cache hit rate: {stats['session']['hit_rate']:.1f}%")
```

### Advanced Queries

```python
# Get high-risk domains
high_risk = categorizer.get_high_risk_domains(limit=10)

# Search for phishing sites
phishing = categorizer.search_phishing_domains(limit=20)

# Get all e-commerce sites
ecommerce = categorizer.get_cached_by_category("e-commerce", limit=50)

# Get domain analysis history
history = categorizer.get_domain_history("example.com")
```

### Direct Cache Access

```python
from database_cache import PostgreSQLCache

# Initialize cache directly
cache = PostgreSQLCache({
    'host': 'localhost',
    'port': 5432,
    'database': 'website_cache',
    'user': 'cacheuser',
    'password': 'cachepass'
})

# Connect
cache.connect()

# Store data
key = cache.generate_key('example.com', 'claude-3.5-sonnet')
cache.set(key, analysis_result, ttl_hours=24)

# Retrieve data
result = cache.get(key)

# Query operations
high_risk = cache.get_high_risk(limit=10)
phishing = cache.search_phishing(limit=10)
stats = cache.get_stats()

# Cleanup
cache.clear_expired()
cache.disconnect()
```

## 📊 Command Line Interface

### Analysis Commands
```bash
# Single domain analysis
python cached_categorizer.py example.com --api-key YOUR_KEY

# Force fresh analysis (bypass cache)
python cached_categorizer.py example.com --force-refresh

# Batch analysis
python cached_categorizer.py --batch domains.txt --api-key YOUR_KEY

# Skip DNS validation
python cached_categorizer.py internal.local --skip-validation
```

### Cache Query Commands
```bash
# View cache statistics
python cached_categorizer.py --stats

# Get high-risk domains
python cached_categorizer.py --high-risk --limit 10

# Search for phishing sites
python cached_categorizer.py --search-phishing --limit 20

# Get domains by category
python cached_categorizer.py --category e-commerce --limit 50

# Get domain history
python cached_categorizer.py --history example.com
```

### Cache Management
```bash
# Clear expired entries
python cached_categorizer.py --clear-expired

# Clear all cache (requires confirmation)
python cached_categorizer.py --clear-all

# Export results to JSON
python cached_categorizer.py --high-risk --output high_risk.json --json
```

## ⚙️ Configuration

### Environment Variables (.env)
```bash
# PostgreSQL Configuration
POSTGRES_HOST=localhost
POSTGRES_PORT=5432
POSTGRES_DB=website_cache
POSTGRES_USER=cacheuser
POSTGRES_PASSWORD=cachepass

# Cache Settings
CACHE_TYPE=postgresql  # Options: postgresql, redis, mongodb
CACHE_TTL_HOURS=24     # How long to cache results

# API Configuration
OPENROUTER_API_KEY=your_api_key_here
DEFAULT_MODEL=anthropic/claude-3.5-sonnet
```

### Connection Parameters
```python
# Custom PostgreSQL configuration
cache_config = {
    'host': 'db.example.com',
    'port': 5432,
    'database': 'custom_db',
    'user': 'custom_user',
    'password': 'secure_password'
}

categorizer = CachedWebsiteCategorizer(
    api_key="your_key",
    cache_type="postgresql",
    cache_config=cache_config,
    cache_ttl_hours=48
)
```

## 📈 Performance Benefits

### Without Cache
- Every domain analysis requires an API call
- Cost: ~$0.003 per domain (Claude 3.5 Sonnet)
- Time: 4-12 seconds per domain
- No historical data

### With Cache
- First analysis: 4-12 seconds (cached)
- Subsequent analyses: <100ms (from cache)
- Cost reduction: 80-95% for repeated domains
- Full history and analytics available
- Advanced queries on cached data

### Cache Hit Rates (Typical)
- Development: 20-40% hit rate
- Production (same domains): 60-80% hit rate
- Monitoring use case: 90%+ hit rate

## 🔍 Query Capabilities

### PostgreSQL JSONB Queries
```sql
-- Find all high-risk e-commerce sites
SELECT domain, analysis_data
FROM website_cache
WHERE analysis_data->>'primary_category' = 'e-commerce'
  AND analysis_data->'security_assessment'->>'risk_level' = 'high';

-- Find phishing sites impersonating specific brand
SELECT domain, analysis_data
FROM website_cache
WHERE analysis_data->'phishing_indicators'->>'impersonating_brand' = 'Amazon';

-- Get risk distribution
SELECT 
    analysis_data->'security_assessment'->>'risk_level' as risk_level,
    COUNT(*) as count
FROM website_cache
GROUP BY risk_level;
```

## 🔄 Extending to Other Databases

### Adding Redis Support
```python
class RedisCache(CacheInterface):
    def __init__(self, connection_params):
        import redis
        self.client = redis.Redis(**connection_params)
    
    def get(self, key):
        data = self.client.get(key)
        return json.loads(data) if data else None
    
    def set(self, key, data, ttl_hours=24):
        self.client.setex(
            key,
            timedelta(hours=ttl_hours),
            json.dumps(data)
        )
        return True
```

### Adding MongoDB Support
```python
class MongoDBCache(CacheInterface):
    def __init__(self, connection_params):
        from pymongo import MongoClient
        self.client = MongoClient(**connection_params)
        self.db = self.client.website_cache
        self.collection = self.db.analysis
    
    def get(self, key):
        doc = self.collection.find_one({'_id': key})
        return doc['data'] if doc else None
    
    def set(self, key, data, ttl_hours=24):
        self.collection.replace_one(
            {'_id': key},
            {
                '_id': key,
                'data': data,
                'expires_at': datetime.utcnow() + timedelta(hours=ttl_hours)
            },
            upsert=True
        )
        return True
```

## 📊 Monitoring & Maintenance

### View Cache Statistics
```python
stats = cache.get_stats()
# Returns:
{
    'total_entries': 1523,
    'active_entries': 1456,
    'expired_entries': 67,
    'categories': {'e-commerce': 234, 'news': 156, ...},
    'risk_levels': {'high': 23, 'medium': 145, 'low': 1288},
    'hit_rate': 76.5
}
```

### Automated Cleanup (Cron)
```bash
# Add to crontab for daily cleanup
0 3 * * * cd /path/to/project && python cached_categorizer.py --clear-expired
```

### Database Maintenance
```sql
-- Vacuum and analyze for performance
VACUUM ANALYZE website_cache;

-- Check table size
SELECT pg_size_pretty(pg_total_relation_size('website_cache'));

-- Remove old entries manually
DELETE FROM website_cache WHERE created_at < NOW() - INTERVAL '30 days';
```

## 🐳 Docker Deployment

### Docker Compose
```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: website_cache
      POSTGRES_USER: cacheuser
      POSTGRES_PASSWORD: cachepass
    ports:
      - "5432:5432"
    volumes:
      - postgres-data:/var/lib/postgresql/data

  app:
    build: .
    depends_on:
      - postgres
    environment:
      POSTGRES_HOST: postgres
      POSTGRES_PORT: 5432
      POSTGRES_DB: website_cache
      POSTGRES_USER: cacheuser
      POSTGRES_PASSWORD: cachepass
      OPENROUTER_API_KEY: ${OPENROUTER_API_KEY}
    command: python cached_categorizer.py --stats

volumes:
  postgres-data:
```

## 🛠️ Troubleshooting

### Connection Issues
```bash
# Test PostgreSQL connection
psql -h localhost -p 5432 -U cacheuser -d website_cache

# Check if tables exist
\dt

# View indexes
\di
```

### Cache Not Working
```python
# Debug mode
import logging
logging.basicConfig(level=logging.DEBUG)

# Test cache operations
from database_cache import PostgreSQLCache
cache = PostgreSQLCache(config)
if cache.connect():
    print("Connected successfully")
    test_key = "test:debug"
    cache.set(test_key, {"test": "data"})
    result = cache.get(test_key)
    print(f"Cache working: {result is not None}")
```

### Performance Issues
```sql
-- Check slow queries
SELECT query, calls, mean_exec_time
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 10;

-- Rebuild indexes
REINDEX TABLE website_cache;
```

## 📈 Best Practices

1. **TTL Strategy**
   - Short TTL (1-6 hours) for frequently changing sites
   - Medium TTL (24 hours) for general use
   - Long TTL (7-30 days) for stable sites

2. **Key Generation**
   - Include model in key for different analysis versions
   - Use consistent hashing for key distribution

3. **Batch Operations**
   - Use batch analysis for multiple domains
   - Implement connection pooling for high load

4. **Monitoring**
   - Track hit rates to optimize TTL
   - Monitor database size and performance
   - Set up alerts for high-risk domains

5. **Security**
   - Use environment variables for credentials
   - Implement read-only database user for queries
   - Regular backups of cache data

## 🎯 Future Enhancements

- [ ] Redis implementation for faster in-memory caching
- [ ] MongoDB implementation for document-based storage
- [ ] Multi-tier caching (Redis → PostgreSQL)
- [ ] Automatic cache warming for popular domains
- [ ] REST API for cache queries
- [ ] Web dashboard for visualization
- [ ] Export to various formats (CSV, Excel, PDF)
- [ ] Machine learning on cached data patterns

## 📄 License

MIT License - See LICENSE file for details
