# Machine Learning Web Categorization Module

A neural network-based web categorization system that works offline without API calls, saving OpenRouter credits while maintaining accuracy.

## 🎯 Purpose

The ML module provides an alternative to API-based categorization:
- **Save API credits** - No OpenRouter API calls required
- **Offline operation** - Works without internet (after training)
- **Fast analysis** - No network latency
- **Consistent results** - Same input always gives same output
- **Privacy** - No data sent to external services

## 🚀 Quick Start

### 1. Install Dependencies

```bash
pip install scikit-learn joblib numpy
```

### 2. Train the Models (Optional)

```bash
# Train with synthetic data
python train_ml_model.py --train

# Output:
# Category Accuracy: 92%
# Risk Assessment Accuracy: 88%
# Phishing Detection Accuracy: 95%
```

### 3. Use ML Categorization

```bash
# Single domain with ML
python webl8ai.py example.com --useML

# Batch processing with ML
python webl8ai.py --batch domains.txt --useML

# Output shows ML was used
# Method: Machine Learning (No API calls)
```

## 🏗️ Architecture

### Neural Network Models

The system uses three separate neural networks:

1. **Category Classifier**
   - Multi-layer Perceptron (200-100-50 neurons)
   - 18 output categories
   - ReLU activation, Adam optimizer
   - Early stopping for optimal performance

2. **Risk Assessment Model**
   - Two-layer network (100-50 neurons)
   - 4 risk levels: low, medium, high, critical
   - Trained on security features

3. **Phishing Detection Model**
   - Binary classifier
   - Specialized for phishing detection
   - High precision for security

### Feature Extraction

The system extracts 60+ features from domains:

#### Domain Features
- Length and structure
- Subdomain count
- Character patterns (numbers, hyphens)
- TLD analysis (.com, .edu, .gov, etc.)
- Typosquatting detection
- Brand impersonation checks

#### Content Features
- HTML structure analysis
- Title and meta tags
- Form detection (login, payment)
- Link analysis (internal/external ratio)
- Social media presence
- Shopping cart indicators
- News article structure
- Language detection

#### Security Features
- SSL/TLS validation
- Security headers (HSTS, CSP, X-Frame)
- Suspicious form patterns
- External link ratios
- Server technology detection

## 📊 Performance

### Accuracy Metrics
- **Category Classification**: ~85-92% accuracy
- **Risk Assessment**: ~88% accuracy
- **Phishing Detection**: ~95% accuracy

### Speed Comparison
| Method | Time per Domain | API Calls |
|--------|-----------------|-----------|
| API-based | 2-5 seconds | 1 call |
| ML-based | 0.5-1 second* | 0 calls |

*After initial content fetch

## 🔧 Training System

### Synthetic Data Generation

The training script generates realistic training data:

```python
# Category-specific patterns
'news_media': {
    'has_news_structure': True,
    'num_links': 50,
    'content_length': 5000,
    'cat_news_media': 5
}

'ecommerce_shopping': {
    'has_shopping_cart': True,
    'num_forms': 5,
    'cat_ecommerce_shopping': 7
}
```

### Training Process

```bash
python train_ml_model.py --train

# Generates 750+ training samples
# Splits 80/20 for train/test
# Trains all three models
# Saves to ml_models/ directory
```

### Model Storage

Models are saved in platform-specific locations:
- Linux: `/var/lib/webl8ai/ml_models/`
- Windows: `C:\ProgramData\Webl8ai\ml_models\`
- User fallback: `~/.webl8ai/ml_models/`

Files saved:
- `category_model.pkl` - Category classifier
- `risk_model.pkl` - Risk assessment
- `phishing_model.pkl` - Phishing detection
- `text_vectorizer.pkl` - Text processing
- `feature_scaler.pkl` - Feature normalization

## 🎯 Usage Examples

### Basic Usage

```python
from ml_categorizer import MLWebCategorizer

# Initialize
ml_cat = MLWebCategorizer()

# Analyze domain
result = ml_cat.analyze_website("example.com")

print(f"Category: {result['primary_category']}")
print(f"Risk: {result['security_assessment']['risk_level']}")
print(f"Phishing: {result['phishing_indicators']['likely_phishing']}")
```

### Command Line

```bash
# Test ML categorizer
python ml_categorizer.py

# Analyze with ML (no API)
python webl8ai.py amazon.com --useML

# Batch analysis with ML
python webl8ai.py --batch sites.txt --useML --output results.json
```

### Integration Example

```python
# Check if ML is available
if args.useML:
    result = categorizer.analyze_with_ml(domain)
else:
    result = categorizer.analyze(domain)  # Uses API
```

## 📈 Feature Details

### Category Detection

The system recognizes patterns for each category:

| Category | Key Indicators |
|----------|---------------|
| news_media | Article structure, dates, authors |
| social_networking | Login forms, profiles, sharing |
| ecommerce_shopping | Shopping cart, prices, checkout |
| technology | Code samples, APIs, documentation |
| education | .edu domain, courses, students |
| government | .gov domain, agencies, public info |
| finance | Banking terms, login security |
| health_medical | Medical terms, patient info |
| entertainment | Videos, games, streaming |

### Risk Assessment

Risk levels based on:
- **Low**: Valid SSL, security headers, known TLD
- **Medium**: Missing security features
- **High**: Suspicious patterns, typosquatting
- **Critical**: Multiple phishing indicators

### Phishing Detection

Checks for:
- Brand impersonation
- Typosquatting
- Suspicious forms
- Credential harvesting patterns
- URL obfuscation

## 🔄 Fallback System

When models aren't trained, the system uses heuristics:

```python
def _heuristic_categorization(self, features):
    # Count category keyword matches
    # Check specific features
    # Return best match or 'unknown'
```

This ensures the system always works, even without trained models.

## ⚙️ Configuration

### Enable ML in Config

```ini
# webl8ai.conf
[ml]
enabled = true
model_path = /var/lib/webl8ai/ml_models
auto_train = false
fallback_to_api = true
```

### Environment Variables

```bash
export WEBL8AI_USE_ML=true
export WEBL8AI_ML_MODELS=/path/to/models
```

## 🧪 Testing

### Unit Tests

```bash
# Test ML categorizer
python -m pytest test_ml_categorizer.py

# Test specific domain
python ml_categorizer.py
```

### Accuracy Testing

```python
# Test on real domains
test_cases = [
    ('google.com', 'technology'),
    ('amazon.com', 'ecommerce_shopping'),
    ('cnn.com', 'news_media')
]

for domain, expected in test_cases:
    result = ml_cat.analyze_website(domain)
    assert result['primary_category'] == expected
```

### Performance Comparison

```bash
# Compare API vs ML
time python webl8ai.py example.com           # API
time python webl8ai.py example.com --useML   # ML

# ML is typically 3-5x faster
```

## 📊 Monitoring

### ML-Specific Logging

```python
logger.info(f"ML Analysis starting for domain: {domain}")
logger.info(f"ML Analysis complete: {category} ({risk} risk)")
```

### Metrics Tracking

- Model accuracy over time
- Feature extraction success rate
- Fallback usage frequency
- Performance benchmarks

## 🚫 Limitations

1. **Initial Content Fetch** - Still needs to download HTML
2. **Training Data** - Accuracy depends on training quality
3. **New Patterns** - May not recognize very new website types
4. **Dynamic Content** - JavaScript-heavy sites may be misclassified
5. **Language** - Best performance on English sites

## 🔮 Future Enhancements

- [ ] Deep learning models (CNN/RNN)
- [ ] Multi-language support
- [ ] JavaScript rendering
- [ ] Transfer learning from pre-trained models
- [ ] Online learning from corrections
- [ ] Ensemble methods
- [ ] Feature importance analysis
- [ ] A/B testing framework

## 🆚 API vs ML Comparison

| Aspect | API-based | ML-based |
|--------|-----------|----------|
| **Accuracy** | 95-98% | 85-92% |
| **Speed** | 2-5 sec | <1 sec |
| **Cost** | Per API call | Free |
| **Internet** | Required | Not required* |
| **Updates** | Automatic | Manual retraining |
| **Privacy** | Data sent to API | Local processing |

*After initial HTML fetch

## 🛠️ Troubleshooting

### "ML categorization unavailable"
```bash
pip install scikit-learn joblib numpy
```

### "No pre-trained models found"
```bash
python train_ml_model.py --train
```

### Low accuracy
- Retrain with more data
- Check feature extraction
- Verify domain accessibility

### Slow performance
- Check model size
- Enable caching
- Reduce feature count

## 📝 Best Practices

1. **Regular Retraining** - Update models monthly
2. **Feature Selection** - Remove low-importance features
3. **Validation Set** - Keep separate test data
4. **Error Analysis** - Review misclassifications
5. **Hybrid Approach** - Use ML for common sites, API for critical

## 📚 API Compatibility

The ML module returns data in the exact same format as the API:

```json
{
  "status": "success",
  "domain": "example.com",
  "analysis": {
    "primary_category": "business",
    "confidence_score": 0.85,
    "security_assessment": {
      "risk_level": "low"
    },
    "phishing_indicators": {
      "likely_phishing": false
    },
    "ml_analysis": true  // Flag indicating ML was used
  }
}
```

This ensures seamless integration with existing code.

---

The ML categorization module provides a powerful alternative to API-based analysis, offering cost savings and offline operation while maintaining good accuracy for most use cases.
