# Customer Management System for Website Categorizer

A complete customer management and rate limiting system for the Website Categorizer API, with SQLite database, web admin panel, and flexible rate limiting.

## ✨ Features

- **Customer Database** - SQLite-based customer data management
- **Rate Limiting** - Per second/minute/hour/day/month limits
- **API Key Management** - Secure key generation and validation  
- **Usage Tracking** - Real-time API call monitoring
- **Web Admin Panel** - Full-featured customer management interface
- **Optional Integration** - Works independently or integrated
- **Automatic Expiry** - Subscription-based access control

## 🚀 Quick Start

### 1. Enable Customer Management

```bash
# Set environment variable
export ENABLE_CUSTOMER_MANAGEMENT=true

# Or in your Python script
os.environ['ENABLE_CUSTOMER_MANAGEMENT'] = 'true'
```

### 2. Start Admin Panel

```bash
python customer_admin.py
```

Visit: http://localhost:5000
- Username: `admin`
- Password: `admin` (change immediately!)

### 3. Create a Customer

Via admin panel or programmatically:

```python
from customer_manager import CustomerManager

manager = CustomerManager()
customer = manager.create_customer(
    name="Acme Corp",
    expiry_days=30,
    limit_per_second=10,
    limit_per_minute=100,
    limit_per_hour=1000,
    limit_per_day=10000,
    limit_per_month=100000
)

print(f"API Key: {customer.api_key}")
```

### 4. Use with Rate Limiting

```python
from website_categorizer import WebsiteCategorizer
from rate_limiter import create_rate_limited_categorizer

# Create rate-limited version
RateLimitedCategorizer = create_rate_limited_categorizer(WebsiteCategorizer)

categorizer = RateLimitedCategorizer(
    api_key="your_openrouter_key",
    enable_rate_limiting=True
)

# Analyze with customer API key
result = categorizer.analyze_website_with_rate_limit(
    domain="example.com",
    api_key="customer_api_key_here"
)

if result['status'] == 'error' and result.get('error_type') == 'rate_limit':
    print(f"Rate limit exceeded: {result['error']}")
else:
    print(f"Analysis complete: {result}")
```

## 📊 Database Schema

### Customers Table
- `customer_id` - Unique identifier
- `name` - Customer name
- `api_key` - Unique API key
- `signup_date` - Registration date
- `expiry_date` - Subscription expiry
- `limit_per_*` - Rate limits
- `is_active` - Active status
- `total_calls` - Total API calls

### API Usage Table
- Tracks every API call
- Response times
- Status codes
- Timestamps

### Rate Limits Table
- Current window tracking
- Per-period counters

## 🔧 Configuration

### Environment Variables

```bash
# Customer Management
ENABLE_CUSTOMER_MANAGEMENT=true  # Enable/disable system
CUSTOMER_DB=customers.db         # Database file path

# Flask Admin
FLASK_SECRET_KEY=your-secret-key # Session secret
```

### Rate Limit Defaults

| Period | Default Limit |
|--------|--------------|
| Second | 10 calls |
| Minute | 100 calls |
| Hour | 1,000 calls |
| Day | 10,000 calls |
| Month | 100,000 calls |

## 🌐 Admin Panel Features

### Dashboard
- Customer overview
- Active/expired counts
- Total API usage
- Recent customers

### Customer Management
- Create new customers
- Edit rate limits
- Renew subscriptions
- View usage statistics
- Deactivate accounts

### Usage Analytics
- Real-time usage charts
- Per-period breakdowns
- Response time metrics
- Daily usage trends

### Settings
- Change admin password
- Create admin users
- Clean old data
- System information

## 🔑 API Key Validation

```python
from rate_limiter import check_customer_api_key

result = check_customer_api_key("sk_xxx...")

if result['valid']:
    print(f"Customer: {result['customer']['name']}")
else:
    print(f"Invalid: {result['message']}")
```

## 📈 Usage Statistics

```python
from customer_manager import CustomerManager

manager = CustomerManager()
stats = manager.get_usage_stats(customer_id, days=30)

print(f"Total calls: {stats['total_calls']}")
print(f"Avg response time: {stats['avg_response_time_ms']}ms")
print(f"Daily usage: {stats['daily_usage']}")
print(f"Current usage: {stats['current_usage']}")
```

## 🚫 Rate Limit Responses

When rate limit is exceeded:

```json
{
  "status": "error",
  "error": "Rate limit exceeded: 10 calls per second",
  "error_type": "rate_limit",
  "limits": {
    "per_second": 10,
    "per_minute": 100
  },
  "usage": {
    "second": 11,
    "minute": 45
  }
}
```

## 🔄 Without Customer Management

The website categorizer works independently when customer management is disabled:

```bash
# Disable customer management
export ENABLE_CUSTOMER_MANAGEMENT=false

# Run normally without rate limiting
python website_categorizer.py example.com --api-key YOUR_KEY
```

## 🧪 Testing

Run the complete demo:

```bash
python demo_customer_system.py
```

This will:
1. Create a demo customer
2. Test rate limiting
3. Show usage statistics
4. Demonstrate all features

## 📦 Files

### Core Modules
- `customer_manager.py` - Customer database management
- `rate_limiter.py` - Rate limiting middleware
- `customer_admin.py` - Flask web admin interface

### Templates
- `templates/base.html` - Base layout
- `templates/dashboard.html` - Main dashboard
- `templates/customers.html` - Customer list
- `templates/customer_view.html` - Customer details
- `templates/customer_new.html` - New customer form
- `templates/login.html` - Admin login
- `templates/settings.html` - Admin settings

### Integration
- `demo_customer_system.py` - Complete demo script

## 🎯 Use Cases

1. **SaaS API Service** - Monetize the categorization API
2. **Internal Rate Limiting** - Control usage across departments
3. **Partner Integration** - Provide controlled API access
4. **Trial Management** - Time-limited API access
5. **Usage Analytics** - Track and optimize API usage

## 🔒 Security

- Secure API key generation (cryptographically random)
- Password hashing for admin users
- Session timeout for admin panel
- SQL injection protection
- Rate limit bypass prevention

## 🚀 Production Deployment

1. **Change default admin password immediately**
2. Set strong `FLASK_SECRET_KEY`
3. Use HTTPS for admin panel
4. Regular database backups
5. Monitor usage patterns
6. Set appropriate rate limits

## 📝 Notes

- SQLite database is created automatically
- Customer management is completely optional
- Rate limits are checked before API calls
- Usage is tracked even for failed requests
- Admin panel requires Flask installation
- Works with all categorizer features (caching, MX records, etc.)

## 🆘 Troubleshooting

### "Customer management module not found"
- Install requirements: `pip install -r requirements.txt`

### "Rate limit exceeded" immediately
- Check rate limits are appropriate
- Verify time windows are calculating correctly
- Check customer hasn't exceeded monthly limit

### Admin panel not loading
- Ensure Flask is installed
- Check port 5000 is available
- Verify templates directory exists

### API key not working
- Check customer is active
- Verify subscription hasn't expired
- Ensure customer management is enabled

---

The customer management system provides enterprise-ready features for the Website Categorizer, enabling commercial deployment with full control over access and usage.
