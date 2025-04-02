# Training System API

A comprehensive AI-powered training content generation system that creates structured training courses based on source documents, templates, and language models.

## 🚀 Features

- ✅ **AI-Powered Content Generation**: Generate complete training courses using LLMs and source documents
- ✅ **Template System**: Use customizable templates for consistent output
- ✅ **Multilingual Support**: Generate content in multiple languages
- ✅ **Multiple Export Formats**: Export training as Markdown, HTML, PDF, or JSON
- ✅ **Vector Search**: Find relevant content across your document repository
- ✅ **Document Processing**: Automatic processing of source URLs and documents
- ✅ **API-First Design**: RESTful API for integration with other systems
- ✅ **GPU Acceleration**: CUDA support for faster processing

## 📋 Table of Contents

- [Quick Start](#-quick-start)
- [System Architecture](#-system-architecture)
- [API Reference](#-api-reference)
- [Templates](#-templates)
- [Configuration](#-configuration)
- [Development](#-development)
- [Troubleshooting](#-troubleshooting)

## 🏁 Quick Start

### Using Docker Compose

The simplest way to run the system is with Docker Compose:

```bash
# Clone the repository
git clone https://github.com/your-org/training-system.git
cd training-system

# Copy and edit the environment variables
cp .env.example .env
# Edit .env to configure your settings

# Start the system
docker-compose up -d

# Check the logs
docker-compose logs -f training-system-api
```

The system will be available at http://localhost:9099/docs for interactive API documentation and testing.

### Interactive API Documentation

Once the application is running, you can access:

- **Swagger UI**: http://localhost:9099/docs
  - Interactive API documentation with request/response examples
  - Try out API calls directly from the browser
  - Authentication support (click the "Authorize" button to set your API key)
  - Schema models for all request/response objects

- **ReDoc**: http://localhost:9099/redoc
  - Alternative documentation UI with a different layout
  - Better for reading and understanding the API structure

![Swagger UI Screenshot](https://fastapi.tiangolo.com/img/index/index-01-swagger-ui-simple.png)

### Prerequisites

- Docker and Docker Compose 
- NVIDIA GPU with CUDA support (for optimal performance)
- At least 16GB RAM
- Supabase instance (local or cloud)
- Ollama running (for LLM support)

## 🏗️ System Architecture

The system is built using the following components:

- **FastAPI**: API server
- **Qdrant**: Vector database for semantic search
- **Supabase**: PostgreSQL database with extensions
- **Ollama**: Local LLM inference server
- **PyTorch & Transformers**: Embedding and ML models

## 📘 API Reference

### Authentication

All API endpoints (except health checks) require API key authentication. Include your API key in one of the following ways:

- Header: `x-api-key: your-api-key`
- Query parameter: `?api_key=your-api-key`
- Cookie: `api_key=your-api-key`

### Templates API

Templates are the foundation for generating structured training content.

#### List Templates

```http
GET /templates?template_type=course
```

Query parameters:
- `template_type` (optional): Filter templates by type

Response:
```json
[{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "name": "Basic Course Template",
  "description": "A simple template for generating training courses",
  "template_type": "course",
  "content": "# {{course_title}}\n\n...",
  "parameters": {...},
  "created_at": "2023-06-15T14:30:00Z",
  "updated_at": "2023-06-15T14:30:00Z"
}]
```

#### Get Template

```http
GET /templates/{template_id}
```

Response:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "name": "Basic Course Template",
  "description": "A simple template for generating training courses",
  "template_type": "course",
  "content": "# {{course_title}}\n\n...",
  "parameters": {...},
  "created_at": "2023-06-15T14:30:00Z",
  "updated_at": "2023-06-15T14:30:00Z"
}
```

#### Create Template

```http
POST /templates
```

Request body:
```json
{
  "name": "New Course Template",
  "description": "My custom template",
  "template_type": "course",
  "content": "# {{course_title}}\n\n...",
  "parameters": {
    "course_title": {"type": "string", "description": "The title of the course"}
  }
}
```

#### Update Template

```http
PUT /templates/{template_id}
```

Request body:
```json
{
  "name": "Updated Template Name",
  "description": "Updated description",
  "content": "# {{course_title}}\n\n..."
}
```

#### Delete Template

```http
DELETE /templates/{template_id}
```

#### Generate Content from Template

```http
POST /templates/{template_id}/generate
```

Request body:
```json
{
  "course_title": "Data Privacy Training",
  "course_description": "Learn about data privacy regulations",
  "learning_objectives": [
    "Understand GDPR requirements",
    "Implement data protection measures",
    "Handle data breaches properly"
  ],
  "modules": [...]
}
```

### Training API

Generate complete training courses using AI and templates.

#### Generate Training Course

```http
POST /training/generate-training
```

Request body:
```json
{
  "region": "EU",
  "training_topic": "data_privacy",
  "template_type": "course",
  "language": "en",
  "export_format": "pdf",
  "model": "gemma:27b-q4",
  "meta_tags": {
    "compliance_level": "advanced",
    "audience": "IT staff"
  }
}
```

Response:
- For Markdown/HTML/JSON: Returns the content with appropriate MIME type
- For PDF: Returns the PDF file as a download

#### Generate From Document

```http
POST /training/generate-from-document/{document_id}?template_type=course&export_format=md&model=llama3
```

Path parameters:
- `document_id`: UUID of the source document

Query parameters:
- `template_type` (optional): Template type to use (default: course)
- `export_format` (optional): Export format (default: md)
- `model` (optional): LLM model to use

### Documents API

Manage source documents for training content.

#### List Documents

```http
GET /documents?region=EU&topic=data_privacy&limit=10
```

Query parameters:
- `region` (optional): Filter by region
- `topic` (optional): Filter by training topic
- `limit` (optional): Maximum number of documents to return

#### Get Document

```http
GET /documents/{document_id}?include_chunks=true
```

Query parameters:
- `include_chunks` (optional): Include document chunks in response (default: false)

#### Process Document

```http
POST /documents/process?document_id=550e8400-e29b-41d4-a716-446655440000
```

Query parameters:
- `document_id`: UUID of the document to process

### WebSocket API

Real-time communication for streaming updates.

```javascript
// Connect to WebSocket
const ws = new WebSocket('ws://localhost:9099/ws');

// Listen for messages
ws.onmessage = function(event) {
  const data = JSON.parse(event.data);
  console.log('Message from server:', data);
};

// Send data
ws.send(JSON.stringify({ type: 'request', action: 'status' }));
```

## 🧩 Templates

### Template Variables

Templates support a variety of variable types:

- **Simple Variables**: `{{variable_name}}`
- **Conditional Variables**: `{{#if condition}}...{{else}}...{{/if}}`
- **Loops**: `{{#each items}}{{this}}{{/each}}`
- **Optional Values**: `{?variable:default_value}`
- **Nested Properties**: `{{this.property}}`

### Example Template

```markdown
# {{course_title}}

## Course Overview
{{course_description}}

## Learning Objectives
{{#each learning_objectives}}
- {{this}}
{{/each}}

{{#each modules}}
## Module {{module_number}}: {{module_title}}

### Overview
{{module_description}}

{{#each topics}}
#### {{topic_title}}
{{topic_content}}
{{/each}}

### Summary
{{module_summary}}
{{/each}}
```

## ⚙️ Configuration

### Environment Variables

The system can be configured using environment variables in `.env` file. Key configurations:

| Variable | Description | Default |
|----------|-------------|---------|
| `API_KEY` | API key for authentication | - |
| `MODEL_NAME` | Default LLM model | `gemma:27b-q4` |
| `OLLAMA_BASE_URL` | URL for Ollama API | `http://ollama:11434` |
| `SUPABASE_URL` | URL for Supabase instance | `http://localhost:8000` |
| `SUPABASE_KEY` | Supabase service key | - |
| `QDRANT_URL` | Qdrant server URL | `http://localhost:6333` |
| `FIRECRAWL_API_KEY` | API key for Firecrawl service | - |
| `FIRECRAWL_URL` | API key for Firecrawl service | `http://host.docker.internal:3003` |
See `.env.example` for the complete list of environment variables.

## 🛠️ Development

### Project Structure

```
trainer/
├── app/                          # Application code
│   ├── api/                      # API layer
│   │   ├── routes/               # API routes
│   │   ├── models/               # API request/response models
│   │   └── dependencies.py       # FastAPI dependencies
│   ├── core/                     # Core application
│   │   ├── config/               # Configuration
│   │   ├── security/             # Authentication
│   │   └── logging/              # Logging configuration
│   ├── services/                 # Business logic
│   │   ├── documents/            # Document processing
│   │   ├── embeddings/           # Embedding models
│   │   ├── export/               # Export functionality
│   │   ├── llm/                  # LLM integration
│   │   ├── templates/            # Template processing
│   │   └── training/             # Training generation
│   └── db/                       # Database access
│       ├── migrations/           # Database migrations
│       └── repositories/         # Database repositories
├── data/                         # Data storage
│   ├── templates/                # Template storage
│   └── courses/                  # Generated course storage
├── logs/                         # Application logs
├── tests/                        # Test code
└── requirements/                 # Dependencies
```

### Adding New Features

1. Follow the existing architecture pattern
2. Add routes in the appropriate files under `app/api/routes/`
3. Add service logic in `app/services/`
4. Add models in `app/api/models/`
5. Update documentation as needed

### Running Tests

```bash
# Install dev dependencies
pip install -r requirements/dev.txt

# Run tests
pytest
```

## 🔧 Troubleshooting

### Common Issues

- **API Key Invalid**: Ensure your API key is correctly set in the request headers
- **Model Not Found**: Verify that Ollama is running and the requested model is available
- **Database Connection Failed**: Check your Supabase and Qdrant connection settings
- **Out of Memory**: Reduce batch sizes or use a smaller model

### Logs

Access logs at `logs/application.log` or via Docker:

```bash
docker-compose logs -f training-system-api
```

### Health Check

Check the system status:

```http
GET /health
```

Response:
```json
{
  "status": "ok",
  "environment": {
    "python_version": "3.11.5",
    "platform": "Linux-5.15.0-1041-azure-x86_64-with-glibc2.35"
  },
  "components": {
    "initialized": true,
    "qdrant": "ok",
    "supabase": "ok",
    "embedder": "ok"
  },
  "processing_time_ms": 45.32
}
```

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.
