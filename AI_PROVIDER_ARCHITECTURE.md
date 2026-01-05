# Modular AI Provider Architecture

## Overview

The OpenRouter AI API code has been separated into its own module (`openrouter_api.py`) for better modularity, maintainability, and flexibility. This allows easy swapping of AI providers without modifying the core categorization logic.

## 🏗️ Architecture

### Module Structure

```
website-categorizer-v2/
├── website_categorizer.py      # Core categorization logic
├── openrouter_api.py           # OpenRouter API module (separated)
├── ml_categorizer.py           # Machine Learning module (offline)
└── future_providers/           # Future AI providers
    ├── local_gemma.py         # Local Gemma 3 (placeholder)
    ├── ollama_provider.py     # Ollama integration (future)
    └── anthropic_direct.py    # Direct Anthropic API (future)
```

### Design Pattern

The system uses a **Provider Interface Pattern**:

```python
# Abstract Interface
class AIProviderInterface:
    def categorize_website(self, domain: str, **kwargs) -> Dict
    def test_connection(self) -> bool

# Concrete Implementations
class OpenRouterProvider(AIProviderInterface)    # Current
class LocalGemmaProvider(AIProviderInterface)    # Future
class OllamaProvider(AIProviderInterface)        # Future
```

## 📦 OpenRouter API Module

### Key Components

1. **OpenRouterAPI Class**
   - Handles all OpenRouter-specific logic
   - Manages API calls, retries, and error handling
   - Creates prompts and parses responses

2. **Provider Interface**
   - Standardized interface for all AI providers
   - Makes swapping providers seamless

3. **Factory Function**
   ```python
   provider = get_ai_provider(
       provider_type="openrouter",  # or "local_gemma", "ollama", etc.
       api_key="YOUR_KEY",
       model="anthropic/claude-3.5-sonnet"
   )
   ```

## 🔄 Benefits of Separation

### 1. **Modularity**
- AI provider code isolated from business logic
- Easy to test independently
- Clear separation of concerns

### 2. **Flexibility**
- Swap AI providers without changing core code
- Support multiple providers simultaneously
- Easy A/B testing between providers

### 3. **Maintainability**
- Provider-specific updates don't affect core
- Easier debugging and troubleshooting
- Cleaner codebase

### 4. **Extensibility**
- Add new providers easily
- Support for local models (Gemma, LLaMA, etc.)
- Custom provider implementations

## 🚀 Usage

### Basic Usage

```python
from website_categorizer import WebsiteCategorizer

# Uses OpenRouter by default
categorizer = WebsiteCategorizer(
    api_key="YOUR_KEY",
    model="anthropic/claude-3.5-sonnet"
)

result = categorizer.analyze_website("example.com")
```

### Specify Provider

```python
# Use OpenRouter (default)
categorizer = WebsiteCategorizer(
    api_key="YOUR_KEY",
    provider="openrouter"
)

# Future: Use local Gemma
categorizer = WebsiteCategorizer(
    model_path="/path/to/gemma",
    provider="local_gemma"
)

# Future: Use Ollama (local API)
categorizer = WebsiteCategorizer(
    provider="ollama",
    ollama_host="http://localhost:11434"
)
```

### Direct Provider Usage

```python
from openrouter_api import OpenRouterAPI

# Use provider directly
api = OpenRouterAPI(api_key="YOUR_KEY")
result = api.categorize_website("example.com")
api.close()
```

## 💻 Gemma 3 Requirements

### Hardware Requirements

#### Gemma 2B Model
- **Minimum RAM**: 8GB (CPU inference)
- **Minimum VRAM**: 6GB (GPU inference)
- **Disk Space**: ~5GB for model weights
- **CPU**: Modern x86_64 processor (AVX2 support recommended)

#### Gemma 7B Model
- **Minimum RAM**: 16GB (CPU inference)
- **Minimum VRAM**: 12GB (GPU inference)
- **Disk Space**: ~15GB for model weights
- **Recommended**: NVIDIA GPU with 16GB+ VRAM for good performance

#### Gemma 27B Model (if using)
- **Minimum RAM**: 64GB (CPU inference)
- **Minimum VRAM**: 40GB (GPU inference, requires multi-GPU)
- **Disk Space**: ~55GB for model weights
- **Recommended**: Multiple GPUs or high-RAM server

### Software Requirements

```bash
# Core requirements
pip install transformers>=4.38.0
pip install torch>=2.0.0  # or tensorflow>=2.15.0
pip install accelerate>=0.27.0
pip install sentencepiece>=0.1.99
pip install protobuf>=3.20.0

# Optional for better performance
pip install bitsandbytes  # For 8-bit quantization
pip install flash-attn    # For faster attention (requires CUDA)
```

### Performance Expectations

| Model | Hardware | Inference Speed | Memory Usage |
|-------|----------|----------------|--------------|
| Gemma 2B | CPU (8GB RAM) | 5-10 tokens/sec | ~6GB |
| Gemma 2B | RTX 3060 (12GB) | 30-50 tokens/sec | ~5GB VRAM |
| Gemma 7B | CPU (16GB RAM) | 1-3 tokens/sec | ~14GB |
| Gemma 7B | RTX 4090 (24GB) | 20-40 tokens/sec | ~14GB VRAM |

### Installation Steps

```bash
# 1. Install dependencies
pip install transformers torch accelerate sentencepiece

# 2. Download model (requires Hugging Face account)
from transformers import AutoModelForCausalLM, AutoTokenizer

model_id = "google/gemma-2b"  # or "google/gemma-7b"
tokenizer = AutoTokenizer.from_pretrained(model_id)
model = AutoModelForCausalLM.from_pretrained(
    model_id,
    device_map="auto",  # Automatically use GPU if available
    torch_dtype=torch.float16  # Use half precision for speed
)

# 3. Save locally for offline use
model.save_pretrained("./models/gemma-2b")
tokenizer.save_pretrained("./models/gemma-2b")
```

## 🐪 Ollama Integration

### What is Ollama?

Ollama is a tool that runs large language models locally and **provides an OpenAI-compatible API interface**. This makes it perfect for our architecture because:

1. **Local API Server** - Runs models locally but provides HTTP API access
2. **OpenAI Compatible** - Uses similar API format to OpenAI/OpenRouter
3. **Model Management** - Handles downloading, loading, and running models
4. **Multiple Models** - Supports Gemma, Llama, Mistral, and many others

### Ollama as an API Provider

```python
# Ollama runs a local API server on port 11434
# You make API calls to it just like OpenRouter, but it runs locally

class OllamaProvider(AIProviderInterface):
    def __init__(self, host="http://localhost:11434", model="gemma:2b"):
        self.host = host
        self.model = model
        self.api_url = f"{host}/api/generate"
    
    def categorize_website(self, domain, **kwargs):
        # Make API call to local Ollama server
        response = requests.post(self.api_url, json={
            "model": self.model,
            "prompt": f"Categorize website: {domain}",
            "stream": False
        })
        return self.parse_response(response.json())
```

### Installing and Using Ollama

```bash
# 1. Install Ollama
# macOS/Linux
curl -fsSL https://ollama.ai/install.sh | sh

# Windows
# Download from https://ollama.ai/download/windows

# 2. Pull Gemma model
ollama pull gemma:2b    # 2B parameter model
ollama pull gemma:7b    # 7B parameter model

# 3. Start Ollama server (usually auto-starts)
ollama serve

# 4. Test API endpoint
curl http://localhost:11434/api/generate -d '{
  "model": "gemma:2b",
  "prompt": "Categorize website: example.com"
}'
```

### Ollama API Example

```python
import requests

def categorize_with_ollama(domain: str):
    """Use Ollama's local API to categorize a website"""
    
    # Ollama provides an API endpoint locally
    response = requests.post(
        "http://localhost:11434/api/generate",
        json={
            "model": "gemma:2b",
            "prompt": create_categorization_prompt(domain),
            "stream": False,
            "options": {
                "temperature": 0.3,
                "top_p": 0.9
            }
        }
    )
    
    result = response.json()
    return parse_ollama_response(result['response'])

# Usage in our system
categorizer = WebsiteCategorizer(
    provider="ollama",
    ollama_host="http://localhost:11434",
    model="gemma:7b"
)
```

### Ollama Models Available

| Model | Size | RAM Required | Use Case |
|-------|------|--------------|----------|
| gemma:2b | 1.4GB | 8GB | Fast, general purpose |
| gemma:7b | 4.8GB | 16GB | Better accuracy |
| llama2:7b | 3.8GB | 8GB | Good general model |
| llama2:13b | 7.4GB | 16GB | High accuracy |
| mistral:7b | 4.1GB | 8GB | Fast and accurate |
| mixtral:8x7b | 26GB | 48GB | Very high accuracy |

### Ollama Advantages

1. **Simple API** - REST API similar to OpenAI
2. **Model Management** - Automatic downloading and updates
3. **Multiple Models** - Switch models with one command
4. **Resource Control** - Set memory and CPU limits
5. **No Python Required** - Runs as system service

### Ollama Configuration

```yaml
# ~/.ollama/config.yaml
models:
  default: gemma:2b
  options:
    num_gpu: 1
    num_thread: 8
    memory_limit: 8gb

api:
  host: 0.0.0.0:11434
  cors_origins: ["*"]
  
performance:
  keep_alive: 5m
  max_loaded_models: 2
```

## 🔌 Adding New Providers

### Step 1: Create Provider Class

```python
# my_provider.py
from openrouter_api import AIProviderInterface

class MyCustomProvider(AIProviderInterface):
    def __init__(self, config):
        self.config = config
    
    def categorize_website(self, domain, **kwargs):
        # Your implementation
        return results
    
    def test_connection(self):
        # Test provider availability
        return True
```

### Step 2: Register Provider

```python
# In openrouter_api.py get_ai_provider()
def get_ai_provider(provider_type, **kwargs):
    if provider_type == "my_custom":
        from my_provider import MyCustomProvider
        return MyCustomProvider(**kwargs)
```

### Step 3: Use Provider

```python
categorizer = WebsiteCategorizer(
    provider="my_custom",
    config=my_config
)
```

## 📊 Provider Comparison

| Provider | API Type | Cost | Privacy | Speed | RAM Needed | Setup |
|----------|----------|------|---------|-------|------------|-------|
| OpenRouter | Remote API | Per call | Cloud | 2-5s | Minimal | API key |
| ML Module | Local | Free | Local* | <1s | 2GB | Train models |
| Gemma Direct | Local | Free | Full | 1-3s | 8-64GB | Download model |
| Ollama+Gemma | Local API | Free | Full | 1-3s | 8-64GB | Install Ollama |
| Ollama+Llama | Local API | Free | Full | 1-3s | 8-48GB | Install Ollama |

*ML module still fetches HTML from websites

## 🔧 Configuration

### Via Config File

```ini
# webl8ai.conf
[ai_provider]
type = openrouter              # or local_gemma, ollama, ml
api_key = YOUR_KEY
model = anthropic/claude-3.5-sonnet

[ollama]
host = http://localhost:11434
model = gemma:7b
timeout = 60

[local_model]
path = /models/gemma-2b
device = cuda                  # or cpu
precision = float16            # or float32, int8
```

### Via Environment

```bash
export AI_PROVIDER=ollama
export OLLAMA_HOST=http://localhost:11434
export OLLAMA_MODEL=gemma:7b
```

## 🧪 Testing Providers

### Test OpenRouter

```bash
python openrouter_api.py
# Tests connection and categorization
```

### Test Ollama

```bash
# Start Ollama
ollama serve

# Test API
curl http://localhost:11434/api/generate -d '{
  "model": "gemma:2b",
  "prompt": "Test"
}'

# Use in our system
python test_ollama_provider.py
```

### Compare Providers

```python
from openrouter_api import get_ai_provider

providers = [
    ("openrouter", {"api_key": "KEY1"}),
    ("ml", {}),
    ("ollama", {"host": "http://localhost:11434", "model": "gemma:2b"})
]

for name, config in providers:
    provider = get_ai_provider(name, **config)
    result = provider.categorize_website("example.com")
    print(f"{name}: {result['primary_category']} ({result['response_time']}s)")
```

## 📈 Migration Path

### Current State
```
website_categorizer.py
└── OpenRouter API (embedded)
```

### New Architecture
```
website_categorizer.py
└── AIProviderInterface
    ├── OpenRouterProvider (current)
    ├── MLProvider (implemented)
    ├── OllamaProvider (local API)
    └── LocalGemmaProvider (direct)
```

### Benefits
- No breaking changes to existing code
- Gradual migration to new providers
- Test new providers alongside existing
- Use Ollama for easy local model management

## 🔒 Security Benefits

1. **API Key Isolation** - Provider credentials separate
2. **Provider Sandboxing** - Each provider isolated
3. **Local Options** - Full privacy with local models
4. **Audit Trail** - Clear provider usage tracking
5. **Ollama Benefits** - Runs as service with access control

## 📚 Example Workflows

### Cost Optimization with Ollama

```python
# Use local Ollama for common sites, OpenRouter for critical
def get_categorizer(domain: str):
    if is_critical_domain(domain):
        return WebsiteCategorizer(provider="openrouter")
    else:
        # Use local Ollama - free and private
        return WebsiteCategorizer(
            provider="ollama",
            model="gemma:2b"
        )
```

### Privacy-First with Gemma

```python
# Always use local models via Ollama
categorizer = WebsiteCategorizer(
    provider="ollama",
    ollama_host="http://localhost:11434",
    model="gemma:7b"
)
# No external API calls, complete privacy
```

### High Accuracy Hybrid

```python
# Try local first, fallback to cloud
try:
    # Try local Gemma via Ollama
    local = WebsiteCategorizer(provider="ollama", model="gemma:7b")
    result = local.analyze_website(domain)
    
    if result['confidence'] < 0.7:
        # Low confidence, use cloud
        cloud = WebsiteCategorizer(provider="openrouter")
        result = cloud.analyze_website(domain)
except:
    # Ollama not available, use cloud
    cloud = WebsiteCategorizer(provider="openrouter")
    result = cloud.analyze_website(domain)
```

## 🚦 Status

- ✅ **OpenRouter Module** - Separated and working
- ✅ **ML Module** - Implemented with --useML
- 🔄 **Ollama Provider** - Ready for implementation (API-based)
- 🔄 **Local Gemma** - Requirements documented
- 📋 **Direct APIs** - Planned

## 🎯 Quick Start Guide

### For Ollama + Gemma (Recommended)

```bash
# 1. Install Ollama
curl -fsSL https://ollama.ai/install.sh | sh

# 2. Pull Gemma model
ollama pull gemma:2b

# 3. Implement OllamaProvider (simple HTTP calls)
# 4. Use it
python webl8ai.py example.com --provider ollama
```

### For Direct Gemma (Advanced)

```bash
# 1. Check requirements
# Need: 8GB+ RAM, Python 3.8+

# 2. Install dependencies
pip install transformers torch accelerate

# 3. Download model
python download_gemma.py

# 4. Use it
python webl8ai.py example.com --provider local_gemma
```

---

The modular AI provider architecture makes the system flexible and ready for both direct local model integration (Gemma) and API-based local model servers (Ollama). Ollama is particularly attractive as it provides the simplicity of API calls while keeping everything local.
