# WebL8 AI Response Codes Documentation

## Overview

WebL8 AI uses standardized numeric response codes for all API responses, making it easy for machines to parse and handle different conditions without string matching.

## Response Format

All API responses include a numeric `code` field:

```json
{
  "status": "success|error|warning|info",
  "code": 1000,
  "message": "Human-readable message",
  "data": { ... }
}
```

## Code Ranges

| Range | Category | Description |
|-------|----------|-------------|
| 1000-1999 | SUCCESS | Successful operations |
| 2000-2099 | DNS_ERROR | DNS and network errors |
| 2100-2199 | VALIDATION_ERROR | Input validation errors |
| 3000-3099 | AUTH_ERROR | Authentication errors |
| 4000-4099 | RATE_LIMIT_ERROR | Rate limiting errors |
| 5000-5099 | API_ERROR | External API errors |
| 6000-6099 | CACHE_ERROR | Cache system errors |
| 7000-7099 | SYSTEM_ERROR | Internal system errors |
| 8000-8999 | WARNING | Warning conditions |
| 9000-9999 | INFO | Informational messages |

## Common Response Codes

### Success Codes (1xxx)
- `1000` - Analysis completed successfully
- `1001` - Result served from cache
- `1010` - Customer created successfully
- `1020` - Authentication successful
- `1030` - Batch processing complete

### DNS/Network Errors (20xx)
- `2000` - Domain does not exist (NXDOMAIN)
- `2001` - No DNS A/AAAA records found
- `2002` - DNS resolution timeout
- `2010` - TCP connection failed
- `2011` - TCP connection timeout

### Authentication Errors (30xx)
- `3000` - No API key provided
- `3001` - Invalid API key
- `3002` - API key expired
- `3003` - Account suspended
- `3004` - Account inactive

### Rate Limit Errors (40xx)
- `4001` - Rate limit exceeded (per second)
- `4002` - Rate limit exceeded (per minute)
- `4003` - Rate limit exceeded (per hour)
- `4004` - Rate limit exceeded (per day)
- `4005` - Rate limit exceeded (per month)
- `4010` - Anonymous rate limit exceeded

### API Errors (50xx)
- `5000` - OpenRouter API error
- `5001` - No response from API
- `5003` - API call timeout
- `5004` - OpenRouter rate limit
- `5006` - Invalid OpenRouter API key

### Warnings (8xxx)
- `8002` - High risk domain detected
- `8003` - Phishing indicators detected
- `8004` - Suspicious patterns detected
- `8020` - Approaching rate limit
- `8021` - Subscription expiring soon

## Example Responses

### Successful Analysis
```json
{
  "status": "success",
  "code": 1000,
  "message": "Analysis completed successfully",
  "data": {
    "domain": "example.com",
    "category": "business",
    "risk_level": "low"
  }
}
```

### Rate Limit Error
```json
{
  "status": "error",
  "code": 4003,
  "message": "Rate limit exceeded: 1000 calls per hour",
  "details": {
    "limit": 1000,
    "used": 1001,
    "reset_in": 1800
  },
  "retry_after": 1800
}
```

### Authentication Error
```json
{
  "status": "error",
  "code": 3002,
  "message": "API key expired on 2024-01-01",
  "details": {
    "customer_id": "CUST_123",
    "expiry_date": "2024-01-01"
  }
}
```

### DNS Error
```json
{
  "status": "error",
  "code": 2000,
  "message": "Domain does not exist",
  "details": {
    "domain": "nonexistent.example",
    "dns_error": "NXDOMAIN"
  }
}
```

### Warning with Successful Response
```json
{
  "status": "success",
  "code": 1000,
  "message": "Analysis completed",
  "data": {
    "domain": "suspicious.com",
    "category": "business",
    "risk_level": "high"
  },
  "warnings": [
    {
      "code": 8003,
      "message": "Phishing indicators detected"
    }
  ]
}
```

## Client Implementation Examples

### Python
```python
import requests

response = requests.post('https://api.webl8ai.com/analyze', 
                         json={'domain': 'example.com'})
result = response.json()

# Check by code instead of parsing strings
if result['code'] == 1000:
    # Success
    print(f"Category: {result['data']['category']}")
elif result['code'] == 4003:
    # Rate limit - wait and retry
    time.sleep(result.get('retry_after', 60))
elif 3000 <= result['code'] < 3100:
    # Authentication error - check API key
    print("Authentication failed")
elif 2000 <= result['code'] < 2100:
    # DNS error - domain issue
    print("Domain validation failed")
```

### JavaScript
```javascript
const response = await fetch('https://api.webl8ai.com/analyze', {
  method: 'POST',
  body: JSON.stringify({domain: 'example.com'})
});

const result = await response.json();

switch(true) {
  case result.code === 1000:
    // Success
    console.log(`Category: ${result.data.category}`);
    break;
    
  case result.code >= 4000 && result.code < 4100:
    // Rate limit error
    setTimeout(() => retry(), result.retry_after * 1000);
    break;
    
  case result.code >= 3000 && result.code < 3100:
    // Auth error
    console.error('Authentication failed');
    break;
    
  default:
    console.error(`Error ${result.code}: ${result.message}`);
}
```

### Error Handling by Category
```python
def handle_response(result):
    code = result.get('code', 0)
    
    # Success
    if 1000 <= code < 2000:
        return process_success(result['data'])
    
    # DNS/Network errors - domain issue
    elif 2000 <= code < 2100:
        log_domain_error(result)
        return None
    
    # Validation errors - fix input
    elif 2100 <= code < 2200:
        raise ValueError(result['message'])
    
    # Auth errors - check credentials
    elif 3000 <= code < 3100:
        refresh_api_key()
        raise AuthError(result['message'])
    
    # Rate limit - wait and retry
    elif 4000 <= code < 4100:
        wait_time = result.get('retry_after', 60)
        time.sleep(wait_time)
        return retry_request()
    
    # API errors - external issue
    elif 5000 <= code < 5100:
        log_api_error(result)
        return use_fallback()
    
    # System errors - contact support
    elif 7000 <= code < 7100:
        notify_admin(result)
        raise SystemError(result['message'])
```

## Benefits of Numeric Codes

1. **Language Independent** - No string parsing in different languages
2. **Faster Processing** - Numeric comparison is faster than string matching
3. **Consistent Handling** - Same code always means same condition
4. **Easier Logging** - Group and filter by code ranges
5. **Version Safe** - Messages can change, codes remain stable
6. **Automation Friendly** - Easy to build automated handlers

## Code Checking Functions

The response_codes module provides helper functions:

```python
from response_codes import (
    is_success_code,
    is_error_code, 
    is_warning_code,
    get_code_description,
    get_code_category
)

code = 4003

if is_error_code(code):
    category = get_code_category(code)  # "RATE_LIMIT_ERROR"
    description = get_code_description(code)  # "Rate limit exceeded (per hour)"
```

## Migration from String Parsing

### Before (String Parsing)
```python
if 'rate limit' in result['error'].lower():
    if 'per hour' in result['error']:
        wait_time = 3600
    elif 'per minute' in result['error']:
        wait_time = 60
```

### After (Code Checking)
```python
if result['code'] == 4003:  # Rate limit per hour
    wait_time = result.get('retry_after', 3600)
elif result['code'] == 4002:  # Rate limit per minute
    wait_time = result.get('retry_after', 60)
```

## Best Practices

1. **Always check the code first**, then use message for logging
2. **Handle codes by range** for category-level handling
3. **Use retry_after** field when provided for rate limits
4. **Log both code and message** for debugging
5. **Document which codes your application handles**
6. **Set up monitoring** for specific error code patterns

---

The response code system makes WebL8 AI easier to integrate, more reliable to parse, and simpler to monitor in production environments.
