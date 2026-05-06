# Rollcage BFF SDK

Backend-for-Frontend SDK for Node.js applications.

## Features

### Core Capabilities
- HTTP Client with retry, timeout, and circuit breaker
- Multi-layer caching (Redis, in-memory, hybrid)
- GraphQL client with batching support
- Middleware for JWT auth, rate limiting, logging, CORS
- Structured logging and health checks
- Type-safe configuration with Zod validation
- **Common GraphQL schema** (`commonTypeDefs` / `commonResolvers` — `_health` & `_meta`)

### Reliability
- Circuit breakers and retry logic
- Stateless design for horizontal scaling
- JWT token pass-through
- Comprehensive error handling

## Dependency reference

### `dependencies` (always installed)

| Package               | Purpose                                                                            |
| --------------------- | ---------------------------------------------------------------------------------- |
| `axios`               | HTTP client used by `AxiosClientFactory` and outbound calls                        |
| `cors`                | CORS middleware (mounted by `createBFFServer`)                                     |
| `helmet`              | Sets secure HTTP response headers                                                  |
| `compression`         | gzip/brotli response compression                                                   |
| `ioredis`             | Redis client used by the Redis cache provider                                      |
| `node-cache`          | In-memory cache provider (default)                                                 |
| `pino`                | Structured JSON logger (production fast path)                                      |
| `pino-pretty`         | Human-readable log formatting in development                                       |
| `express-rate-limit`  | Token-bucket rate limiting middleware                                              |

### `peerDependencies` (provided by the consuming app)

| Package    | Purpose                                                                                |
| ---------- | -------------------------------------------------------------------------------------- |
| `express`  | The HTTP framework whose types/runtime the SDK extends. Range: `>=4.18.0` (works with Express 5) |
| `graphql`  | Optional — only needed if you use `commonTypeDefs` / `formatGraphQLError`              |

### `optionalDependencies` (installed only if available — degrade gracefully)

| Package                                          | Purpose                                                       |
| ------------------------------------------------ | ------------------------------------------------------------- |
| `@opentelemetry/api`                             | OpenTelemetry tracing API surface                             |
| `@opentelemetry/sdk-node`                        | OpenTelemetry Node.js SDK (auto trace/metric pipeline)        |
| `@opentelemetry/auto-instrumentations-node`      | Automatic instrumentations for HTTP, Express, Pino, etc.      |
| `@opentelemetry/exporter-zipkin`                 | Zipkin exporter (use OTLP exporters for other backends)       |

### `devDependencies` (only for SDK contributors)

| Package                          | Purpose                                                                  |
| -------------------------------- | ------------------------------------------------------------------------ |
| `typescript`                     | Compiler                                                                 |
| `@types/*`                       | Type definitions for `node`, `express`, `cors`, `compression`, `jsonwebtoken`, `jest`, `supertest` |
| `express`                        | Local Express install for type-checking the SDK source                   |
| `jest` + `ts-jest`               | Unit-test runner                                                         |
| `supertest`                      | HTTP assertions for integration tests                                    |
| `eslint` + `typescript-eslint` + `@eslint/js` + `eslint-config-prettier` | Linting (ESLint 9 flat config) |
| `prettier`                       | Code formatter                                                           |

## 📦 Installation

```bash
npm install @rollcage/bff-sdk
```

## 🚀 Quick Start

### Basic Setup

```typescript
import { createBFFSDK } from 'rollcage-bff-sdk'

// Initialize SDK
const sdk = createBFFSDK({
  baseURL: 'https://api.example.com',
  environment: 'production',
  cache: {
    provider: 'redis',
    defaultTTL: 300
  }
})

// Simple API call with JWT pass-through
app.get('/api/users/:id', async (req, res) => {
  const token = req.headers.authorization?.replace('Bearer ', '')
  const user = await sdk.request('GET', `/users/${req.params.id}`, { token })
  res.json(user)
})
```

### Express Integration

```typescript
import express from 'express'
import { 
  createBFFSDK, 
  authMiddleware, 
  loggingMiddleware,
  errorHandler 
} from 'rollcage-bff-sdk'

const app = express()
const sdk = createBFFSDK({
  baseURL: process.env.API_BASE_URL,
  environment: process.env.NODE_ENV,
  cache: { provider: 'hybrid' }
})

// Apply middleware
app.use(loggingMiddleware())
app.use(authMiddleware({ secret: process.env.JWT_SECRET }))

// Health check
app.get('/health', async (req, res) => {
  const health = await sdk.healthCheck()
  res.status(health.status === 'healthy' ? 200 : 503).json(health)
})

// Error handling
app.use(errorHandler())
```

## 🏗️ Architecture

### Modular Design
```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   HTTP Client   │    │  Cache Manager   │    │   Middleware    │
│                 │    │                  │    │                 │
│ • Fetch-based   │    │ • Redis          │    │ • JWT Auth      │
│ • Retry logic   │    │ • Memory         │    │ • Rate Limit    │
│ • Circuit break │    │ • Hybrid         │    │ • Logging       │
└─────────────────┘    └──────────────────┘    └─────────────────┘
         │                       │                       │
         └───────────────────────┼───────────────────────┘
                                 │
                    ┌─────────────────────┐
                    │     BFF SDK Core    │
                    │                     │
                    │ • Configuration     │
                    │ • Health Checks     │
                    │ • Error Handling    │
                    │ • Observability     │
                    └─────────────────────┘
```

## 📚 Core Features

### HTTP Client with Resilience
- Native fetch API for optimal performance
- Automatic retry with exponential backoff
- Circuit breaker pattern for fault tolerance
- Request/response interceptors
- JWT token pass-through

### Multi-Layer Caching
- **Memory Cache**: Fast L1 cache with LRU eviction
- **Redis Cache**: Distributed L2 cache for scalability  
- **Hybrid Cache**: Best of both worlds
- Cache decorators for easy integration
- Intelligent cache invalidation

### GraphQL Support
- Full GraphQL client with custom headers
- Request batching and data loading
- Error handling and retries
- Token injection support

### Enterprise Middleware
- JWT authentication with role-based access
- Rate limiting (memory/Redis backed)
- Structured request logging
- CORS and security headers
- Error boundary handling

### Observability
- Structured logging with Pino
- Health checks with custom providers
- Metrics collection (Prometheus compatible)
- Performance monitoring
- Request tracing

## 🛠️ CLI Tools

Generate new BFF services:
```bash
npx bff-cli generate --name my-service --port 3000
```

Check service health:
```bash
npx bff-cli health --url http://localhost:3000
```

## Performance

- Native fetch API with minimal abstractions
- HTTP/2 and connection reuse
- Multi-layer caching with invalidation
- Request batching to reduce API calls
- Circuit breakers to prevent failures

## Security

- JWT token pass-through
- CORS protection
- Rate limiting per IP/route
- Security headers via Helmet
- Input validation with Zod

## Deployment

- Stateless design for scaling
- Redis clustering support
- Graceful shutdown
- Health check endpoints
- Metrics collection