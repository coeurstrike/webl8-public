# Website Categorization Tool v2.0

A powerful, production-ready website categorization and security analysis tool using OpenRouter.ai's LLM API. Features DNS validation, consistent JSON output, phishing detection, and comprehensive security assessment.

## 🚀 Quick Start

### Installation
```bash
# Clone or download the project
cd website-categorizer-complete

# Install dependencies
pip install -r requirements.txt

# Set your OpenRouter API key
export OPENROUTER_API_KEY="your_api_key_here"
```

### Basic Usage
```bash
# Analyze a single domain
python website_categorizer.py example.com --api-key YOUR_KEY

# Analyze with cheaper/faster model
python website_categorizer.py example.com --api-key YOUR_KEY --model anthropic/claude-3-haiku

# Skip DNS validation (for internal domains)
python website_categorizer.py internal.local --api-key YOUR_KEY --skip-validation

# Batch analysis
python website_categorizer.py --batch domains.txt --api-key YOUR_KEY --output results.json
```

## ✨ Key Features

### 1. **DNS & TCP Validation**
- Validates domain existence before analysis
- Checks A/AAAA DNS records
- Detects MX records for email capability
- Tests TCP connectivity (ports 443/80)
- Automatic www. subdomain fallback
- Saves API costs by avoiding non-existent domains

### 2. **Guaranteed Consistent JSON Output**
- Every response has identical structure
- Missing fields get sensible defaults
- Type validation with Pydantic
- No more defensive coding needed
- API version tracking for compatibility

### 3. **Comprehensive Analysis**
- Website categorization (18 categories)
- Phishing detection with confidence scoring
- Language identification (all languages)
- Complete redirect chain analysis
- Parked domain detection
- Security risk assessment
- Content analysis and suspicious patterns

### 4. **Production Features**
- FastAPI web service included
- WebSocket support for real-time analysis
- Batch processing capabilities
- Built-in caching
- Rate limiting
- Error recovery

## 📊 JSON Output Structure

Every response is **guaranteed** to have this exact structure:

```json
{
  "status": "success",
  "domain": "example.com",
  "resolved_domain": "www.example.com",
  "dns_records": ["93.184.216.34", "2606:2800:220:1:248:1893:25c8:1946"],
  "mx_records": ["10:mail.example.com", "20:mail2.example.com"],
  "has_mx": true,
  "original_url": "https://www.example.com",
  "final_url": "https://www.example.com",
  "fetch_error": null,
  "analysis": {
    "primary_category": "business",
    "subcategories": ["technology", "software"],
    "is_suspicious": false,
    "phishing_indicators": {
      "likely_phishing": false,
      "confidence": "high",
      "impersonating_brand": null,
      "suspicious_elements": []
    },
    "language": {
      "primary": "en",
      "detected_languages": ["en"]
    },
    "redirect_analysis": {
      "has_redirects": false,
      "redirect_count": 0,
      "redirect_chain": [],
      "final_destination": "www.example.com",
      "suspicious_redirects": false
    },
    "parked_domain_indicators": {
      "is_parked": false,
      "confidence": "high",
      "indicators": []
    },
    "security_assessment": {
      "risk_level": "low",
      "warnings": [],
      "recommendations": []
    },
    "content_analysis": {
      "has_content": true,
      "content_type": "Corporate website",
      "key_topics": ["business", "services"],
      "suspicious_patterns": []
    }
  },
  "model_used": "anthropic/claude-3.5-sonnet",
  "timestamp": "2024-12-23 10:30:45 UTC",
  "api_version": "2.0"
}
```

## 🎯 LLM Model Recommendations

### Best Overall: `anthropic/claude-3.5-sonnet`
- **Cost**: ~$3/1M tokens
- **Use for**: Production, high-accuracy requirements
- **Strengths**: Excellent pattern recognition, security analysis

### Best Value: `anthropic/claude-3-haiku`
- **Cost**: ~$0.25/1M tokens (12x cheaper)
- **Use for**: High-volume screening, initial passes
- **Strengths**: Fast, cost-effective, still accurate

### Recommended Strategy: Two-Tier System
```python
# Use Haiku for screening, Sonnet for suspicious sites
screening_model = "anthropic/claude-3-haiku"     # First pass
verification_model = "anthropic/claude-3.5-sonnet" # Suspicious only
```

## 🔧 API Service

### Start the API Service
```bash
# Basic start
python api_service.py

# With environment variables
export OPENROUTER_API_KEY="your_key"
export API_KEYS="client_key1,client_key2"  # Optional auth
export PORT=8000
python api_service.py

# Production with Gunicorn
gunicorn api_service:app -w 4 --worker-class uvicorn.workers.UvicornWorker
```

### API Endpoints

#### Analyze Single Domain
```bash
curl -X POST http://localhost:8000/analyze \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"domain": "example.com"}'
```

#### Batch Analysis
```bash
curl -X POST http://localhost:8000/analyze/batch \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"domains": ["google.com", "example.com", "suspicious-site.com"]}'
```

#### Quick Phishing Check
```bash
curl -X POST http://localhost:8000/classify/phishing \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"domain": "phishing-test.com"}'
```

#### WebSocket Real-time Analysis
```javascript
const ws = new WebSocket('ws://localhost:8000/ws/analyze');
ws.send(JSON.stringify({
  domain: 'example.com',
  api_key: 'YOUR_KEY'
}));
```

### API Documentation
- Interactive docs: `http://localhost:8000/docs`
- ReDoc: `http://localhost:8000/redoc`

## 🐳 Docker Deployment

### Dockerfile
```dockerfile
FROM python:3.10-slim

WORKDIR /app

COPY requirements_api.txt .
RUN pip install --no-cache-dir -r requirements_api.txt

COPY *.py .

ENV PYTHONUNBUFFERED=1

CMD ["uvicorn", "api_service:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Docker Commands
```bash
# Build
docker build -t website-categorizer .

# Run
docker run -d \
  --name webcategorizer \
  -p 8000:8000 \
  -e OPENROUTER_API_KEY="your_key" \
  -e API_KEYS="key1,key2" \
  --restart unless-stopped \
  website-categorizer
```

## 🛠️ Advanced Usage

### Python Integration
```python
from website_categorizer import WebsiteCategorizer

# Initialize
categorizer = WebsiteCategorizer(
    api_key="your_openrouter_api_key",
    model="anthropic/claude-3.5-sonnet"
)

# Single domain analysis
result = categorizer.analyze_website("example.com")

# With validation skip (for internal domains)
result = categorizer.analyze_website("internal.local", skip_validation=True)

# Batch analysis
domains = ["google.com", "phishing-test.com", "parked-domain.net"]
results = categorizer.analyze_batch(domains)

# Access specific fields (guaranteed to exist)
if result['status'] == 'success':
    category = result['analysis']['primary_category']
    is_phishing = result['analysis']['phishing_indicators']['likely_phishing']
    risk_level = result['analysis']['security_assessment']['risk_level']
    languages = result['analysis']['language']['detected_languages']
```

### Two-Tier Analysis System
```python
# Use cheaper model for screening, expensive for verification
def smart_analysis(domain):
    # First pass with Haiku
    screener = WebsiteCategorizer(api_key, "anthropic/claude-3-haiku")
    initial = screener.analyze_website(domain)
    
    # If suspicious, verify with Sonnet
    if initial['analysis']['is_suspicious'] or \
       initial['analysis']['phishing_indicators']['likely_phishing']:
        verifier = WebsiteCategorizer(api_key, "anthropic/claude-3.5-sonnet")
        return verifier.analyze_website(domain)
    
    return initial
```

## 📈 Performance & Costs

### Analysis Time
- DNS validation: ~0.5-2 seconds
- TCP check: ~0.5-2 seconds
- Website fetch: ~1-3 seconds
- LLM analysis: ~2-5 seconds
- **Total**: ~4-12 seconds per domain

### Cost Optimization
- Cache results (1-hour default TTL)
- Use Haiku for high-volume screening
- Implement two-tier analysis
- Batch processing for efficiency

### Rate Limits
- OpenRouter: Varies by model
- Recommended: 2-second delay between requests
- Batch processing: Max 50 domains per request

## 🔒 Security Features

### Input Validation
- DNS resolution before analysis
- TCP connectivity verification
- URL sanitization
- Timeout protection

### API Security
- Bearer token authentication
- Rate limiting support
- CORS configuration
- Input size limits

### Privacy
- No data persistence by default
- Optional caching (configurable TTL)
- No personal data collection
- Secure API communication

## 🧪 Testing

### Test DNS Validation
```python
# Test script included
python test_validation.py
```

### Test Categories
```bash
# E-commerce site
python website_categorizer.py amazon.com --api-key YOUR_KEY

# News site
python website_categorizer.py cnn.com --api-key YOUR_KEY

# Potential phishing (test carefully)
python website_categorizer.py suspicious-domain.test --api-key YOUR_KEY
```

## 📋 Categories Detected

- `e-commerce` - Online stores, marketplaces
- `news` - News websites, media outlets
- `social-media` - Social networks, forums
- `education` - Schools, learning platforms
- `technology` - Tech companies, IT services
- `finance` - Banks, financial services
- `health` - Medical, healthcare sites
- `entertainment` - Streaming, gaming, media
- `government` - Official government sites
- `business` - Corporate, B2B sites
- `personal-blog` - Individual blogs
- `forum` - Discussion boards
- `documentation` - Technical docs, wikis
- `tools-utilities` - Online tools, utilities
- `adult` - Adult content
- `gambling` - Betting, casino sites
- `parked-domain` - Placeholder pages
- `other` - Uncategorized

## ⚠️ Error Handling

### Common Errors and Solutions

#### DNS Resolution Failed
```json
{
  "status": "error",
  "fetch_error": "Domain validation failed",
  "validation_errors": ["No DNS A or AAAA records found"]
}
```
**Solution**: Check domain spelling, try with www., or use `--skip-validation`

#### API Key Issues
```bash
export OPENROUTER_API_KEY="your_actual_key_here"
```

#### Timeout Errors
- Increase timeout in code
- Check network connectivity
- Try again later

## 📚 Files Included

- `website_categorizer.py` - Main analysis tool with DNS validation
- `api_service.py` - FastAPI web service
- `requirements.txt` - Python dependencies
- `requirements_api.txt` - API service dependencies
- `README.md` - This documentation
- `test_validation.py` - DNS/TCP validation tests
- `.env.example` - Environment variables template
- `docker-compose.yml` - Docker composition
- `setup.sh` - Automated setup script

## 🤝 Contributing

Contributions welcome! Areas for improvement:
- Additional language detection
- Enhanced phishing patterns
- More categories
- Performance optimization
- Additional LLM providers

## 📄 License

MIT License - Free to use and modify

## 🆘 Support

For issues or questions:
1. Check the documentation
2. Review error messages
3. Test with `--skip-validation` flag
4. Verify API key is correct
5. Check OpenRouter service status

## 🔄 Version History

### v2.0 (Current)
- ✅ Guaranteed consistent JSON structure
- ✅ DNS and TCP validation
- ✅ Pydantic models for type safety
- ✅ FastAPI service included
- ✅ WebSocket support
- ✅ Two-tier analysis option

### v1.0
- Initial release
- Basic categorization
- Variable JSON structure

---

**Ready to use!** Start with a simple domain analysis:
```bash
python website_categorizer.py google.com --api-key YOUR_KEY
```
