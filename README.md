# gateway-admin

Microservice responsible for routing and protecting back-office traffic for the [AlgaShop](https://github.com/jeanmalvessi/ems-algashop-meta) platform.

Built with **Spring Cloud Gateway** (WebFlux) reactive edge server, sitting in front of `ordering`, `product-catalog`, and `authorization-server` for the `admin` app.

## Responsibilities

- Routing admin requests to `ordering`, `product-catalog`, and `authorization-server` via Eureka-discovered (`lb://`) instances
- OAuth2 resource-server JWT validation on every `/api/**` route
- Rate limiting (Redis token bucket) on product/category management routes
- Circuit breaking, retries, and response-header deduplication on proxied calls
- CORS configuration for the `admin` Angular app

## Architecture

- **Gateway Layer:** Declarative predicate/filter route definitions (path/method matching, `RequestRateLimiter`, `CircuitBreaker`, `Retry`)
- **Security:** `GatewayAdminSecurityConfig` (JWT-authenticated `/api/**`, public `/actuator/**`), `RateLimitConfig` (per-authenticated-subject rate-limit key resolver)
- **Service Discovery:** `EurekaClientConfig` registers with and resolves peers through `service-registry`

## Tech Stack

- **Java 25**, Spring Boot 4.0.8
- **Spring Cloud Gateway Server WebFlux** (reactive routing, filters, predicates)
- **Spring Cloud Netflix Eureka Client** (service discovery, load-balanced routing)
- **Spring Security OAuth2 Resource Server** (JWT validation)
- **Spring Cloud Circuit Breaker** (Resilience4j, reactive)
- **Spring Data Redis Reactive** + Caffeine (request rate limiting, caching)
- **Spring Cloud AWS Secrets Manager / Parameter Store** (externalized configuration, mocked via LocalStack)
- **Spring Boot Actuator** (monitoring, gateway routes and circuit-breaker health endpoints)

## Routes

Base path: `/api/v1`

| Route | Path | Target | Notes |
|-------|------|--------|-------|
| orders-route | `/orders/**` | `ordering` | Circuit breaker |
| customers-route | `/customers/**` (GET) | `ordering` | |
| product-catalog-list-route | `/products` (GET) | `product-catalog` | Strips `shortDescription`/`mainImage` from list responses |
| product-catalog-route | `/products/**`, `/categories/**`, `/upload-requests/**` | `product-catalog` | Rate limited |
| user-management-route | `/users/**` | `authorization-server` | |

## Running

```bash
./gradlew bootRun
```

Default port: **9998** (development profile)

Requires `service-registry` running and reachable via Eureka, and Redis for rate limiting.
