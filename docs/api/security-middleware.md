---
title: SecurityMiddleware API - FastAPI Guard
description: Complete API reference for FastAPI Guard's SecurityMiddleware class and its configuration options
keywords: security middleware, fastapi middleware, api security, middleware configuration
---

# SecurityMiddleware

The `SecurityMiddleware` class is the core component of FastAPI Guard that handles all security features.

## Class Definition

```python
class SecurityMiddleware(BaseHTTPMiddleware):
    def __init__(
        self,
        app: Callable[[Request], Awaitable[Response]],
        config: SecurityConfig
    ):
        # ... initialization
```

## Methods

### dispatch

```python
async def dispatch(
    self,
    request: Request,
    call_next: Callable[[Request], Awaitable[Response]]
) -> Response:
    """
    Main method that processes each request through
    the security pipeline.
    """
```

### create_error_response

```python
async def create_error_response(
    self,
    status_code: int,
    default_message: str
) -> Response:
    """
    Creates standardized error responses.
    """
```

## Handler Integration
The middleware works with singleton handler instances:

- All handler classes (IPBanManager, CloudManager, etc.) use the singleton pattern
- The middleware initializes these existing instances conditionally based on configuration
- IPInfo is only initialized when country filtering and/or cloud blocking is enabled
- Cloud IP ranges are only loaded when cloud provider blocking is configured
- This selective loading improves performance when not all features are used

## Redis Configuration
Enable Redis in SecurityConfig:

```python
config = SecurityConfig(
    enable_redis=True,
    redis_url="redis://prod:6379/0",
    redis_prefix="prod_security:"
)
```

The middleware automatically initializes:
- CloudManager cloud provider ip ranges
- IPBanManager distributed banning
- IPInfoManager IP geolocation
- RateLimitManager rate limiting
- RedisManager Redis caching
- SusPatternsManager suspicious patterns

## Usage Example

```python
from fastapi import FastAPI
from guard.middleware import SecurityMiddleware
from guard.models import SecurityConfig
from guard.handlers.ipinfo_handler import IPInfoManager

app = FastAPI()

config = SecurityConfig(
    geographical_ip_manager=IPInfoManager("your_token"),  # NOTE: Required when using country blocking
    rate_limit=100
)

app.add_middleware(SecurityMiddleware, config=config)
```