# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Building and Development
- **Build frontend**: `cd web/default && npm install && npm run build`
- **Build backend**: `go mod download && go build -ldflags "-s -w" -o one-api`
- **Run locally**: `./one-api --port 3000 --log-dir ./logs`
- **Build all web themes**: `cd web && ./build.sh` (reads themes from `web/THEMES` file)
- **Run with Docker**: `docker run --name one-api -d --restart always -p 3000:3000 -e TZ=Asia/Shanghai -v /home/ubuntu/data/one-api:/data justsong/one-api`

### Testing and Quality
- **Run tests**: `go test ./...`
- **Format code**: `go fmt ./...`
- **Vet code**: `go vet ./...`

## Architecture Overview

### Core Components

**Main Application (main.go)**
- Entry point that initializes all services
- Sets up database, Redis, HTTP server with Gin framework
- Manages configuration through environment variables and command flags
- Supports both SQLite (default) and MySQL/PostgreSQL databases

**Router System (router/)**
- `SetRouter()` configures four main route groups:
  - API routes for management operations
  - Dashboard routes for admin interface
  - Relay routes for AI model requests
  - Web routes for frontend static files

**Model Management (model/)**
- Database models for users, channels, tokens, logs
- Channel and user caching system for performance
- Support for both main database and separate log database

**Relay System (relay/)**
- Core request routing and load balancing
- Adaptor pattern for different AI providers
- Request/response transformation between formats
- Retry logic with automatic failover

### AI Provider Adaptors

The system uses an adaptor pattern (`relay/adaptor/interface.go`) to support multiple AI providers:

**Key Adaptors Include:**
- **OpenAI**: Reference implementation with full feature support
- **Anthropic**: Claude models with message format conversion
- **Google**: Gemini/PaLM models with safety settings
- **AWS**: Bedrock service integration (Claude, Llama3)
- **Azure**: OpenAI-compatible endpoints
- **Chinese Providers**: Baidu, Alibaba, Tencent, ByteDance, etc.
- **Other**: Cohere, Mistral, Groq, Ollama, etc.

Each adaptor implements:
- Request URL generation
- Header setup and authentication
- Request/response format conversion
- Model list management

### Key Features

**Multi-tenant System**
- User groups with different rate limits and pricing
- Channel groups for load distribution
- Token-based access control with quotas

**Monitoring and Reliability**
- Request success rate tracking
- Automatic channel disabling for failed providers
- Load balancing across multiple channels
- Retry logic with smart failover

**Caching Strategy**
- Redis support for distributed deployments
- In-memory caching for single-node setups
- Channel cache for fast model availability checks

### Configuration

**Environment Variables (commonly used):**
- `SQL_DSN`: Database connection string (required for production)
- `REDIS_CONN_STRING`: Redis connection for caching
- `SESSION_SECRET`: Fixed session key for multi-instance deployments
- `NODE_TYPE`: Set to "slave" for multi-node deployments
- `SYNC_FREQUENCY`: Config sync interval (seconds) for cached deployments

**Multi-node Deployment Requirements:**
- Shared MySQL/PostgreSQL database (not SQLite)
- Same `SESSION_SECRET` across all nodes
- Redis setup on each node for optimal performance
- Master node handles admin interface, slaves handle API requests

### Development Notes

**Database Migrations**: Automatic on startup - no manual schema changes needed

**Theme System**: Multiple frontend themes in `web/` directory, configurable via `THEME` environment variable

**Billing System**: Token-based quota system with support for different model pricing rates

**API Compatibility**: Maintains OpenAI API compatibility while supporting provider-specific features through adaptors