---
description: "Comprehensive FastAPI + Supabase backend architecture guide covering directory structure, layered patterns, and development conventions"
---

# FastAPI + Supabase Architecture Guide

This guide establishes our standard patterns for FastAPI backends with Supabase. Follow these conventions for all projects to ensure consistency and maintainability.

## Directory Structure

```
app/
├── __init__.py            # Make directory a module
├── main.py                # Application entry point
├── config.py              # Pydantic configuration
├── logger.py              # Rich logging setup
├── exceptions.py          # Central error handling
├── supabase.py            # Supabase client setup
├── dependencies.py        # Dependency injection providers
├── routes/                # API endpoints
│   ├── __init__.py        # Make directory a module
│   └── [feature].py       # Feature-specific routes
├── services/              # Business logic
│   ├── __init__.py        # Make directory a module
│   └── [feature]_service.py
├── repositories/          # Data access layer
│   ├── __init__.py        # Make directory a module
│   ├── base_repository.py # Base CRUD operations
│   └── [feature]_repository.py
└── schemas/               # Pydantic models
    ├── __init__.py        # Make directory a module
    └── [feature].py
```

## Python Modules with `__init__.py`

Always include `__init__.py` files in every directory to:

1. Mark directories as Python packages
2. Enable clean absolute imports
3. Export specific components for cleaner imports

Example usage in `repositories/__init__.py`:

```python
from app.repositories.user_repository import UserRepository
from app.repositories.item_repository import ItemRepository

# This allows: from app.repositories import UserRepository
__all__ = ["UserRepository", "ItemRepository"]
```

## Decision Framework: Where Does My Code Go?

When adding new code, follow this decision process:

### 1. Is this data validation or API contracts?
If defining input/output data structures, place in `/schemas/[feature].py`.

**Examples:**
- Request validation models
- Response serialization models
- Shared data structures

**Conceptual pattern:**
```
# Base classes define common fields
BaseEntity:
  common_field1
  common_field2

# Request models extend base with input validation
CreateEntity(BaseEntity):
  required_field
  optional_field with validation rules

# Response models include system fields
EntityResponse(BaseEntity):
  id
  created_at
  updated_at
```

### 2. Is this data access code?
If code interacts with Supabase or other data sources, place in `/repositories/[feature]_repository.py`.

#### Repository Pattern: Good vs Bad

**❌ BAD: Repository with mixed concerns**
```python
UserRepository:
  get_user(id):
    user = query database for user
    if not user:
      log error
      send notification email
    apply business logic transformations
    return user
```

**✅ GOOD: Thin repository focused on data access**
```python
UserRepository extends BaseRepository:
  initialize with "users" table

  # Only data access concerns
  get_by_email(email):
    return query users where email = email

  # No business logic, logging, or side effects
```

#### Repository Error Handling

**❌ BAD: Returning error objects**
```python
get_user(id):
  try:
    result = query database for user
    if not result:
      return {"error": "User not found"}
    return result
  except DatabaseError:
    return {"error": "Database error"}
```

**✅ GOOD: Raising specific exceptions**
```python
get_user(id):
  try:
    result = query database for user
    if not result:
      raise NotFoundException(f"User with id {id} not found")
    return result
  except DatabaseError as e:
    # Transform external error to application exception
    raise DatabaseServiceException(f"Failed to get user: {str(e)}")
```

### 3. Is this business logic?
If implementing domain rules or coordinating multiple operations, place in `/services/[feature]_service.py`.

#### Service Layer: Good vs Bad

**❌ BAD: Service with direct data access**
```python
UserService:
  create_user(data):
    # Direct Supabase access
    supabase = get_supabase_client()
    return supabase.from_("users").insert(data)
```

**✅ GOOD: Service with repository abstraction**
```python
UserService:
  initialize with UserRepository

  create_user(data):
    # Business validations
    validate email format
    check password strength

    # Using repository for data access
    return repository.create(data)
```

#### Service Error Handling

**❌ BAD: Inconsistent error handling**
```python
UserService:
  get_user_dashboard(user_id):
    # Inconsistent approach to errors
    user = repository.get_user(user_id)
    if not user:
      return {"error": "User not found"}

    try:
      activity = activity_repository.get_activity(user_id)
    except Exception:
      activity = []

    preferences = preference_repository.get_preferences(user_id)
    if not preferences:
      preferences = default_preferences

    return {"user": user, "activity": activity, "preferences": preferences}
```

**✅ GOOD: Consistent error handling with graceful degradation**
```python
UserService:
  get_user_dashboard(user_id):
    # Critical data - fail if not available
    try:
      user = repository.get_user(user_id)
    except NotFoundException:
      # Let it propagate - can't proceed without user
      raise

    # Non-critical data - use fallbacks
    try:
      activity = activity_repository.get_activity(user_id)
    except Exception as e:
      logger.warning(f"Failed to get activity: {str(e)}")
      activity = []

    try:
      preferences = preference_repository.get_preferences(user_id)
    except Exception as e:
      logger.warning(f"Failed to get preferences: {str(e)}")
      preferences = default_preferences

    return {"user": user, "activity": activity, "preferences": preferences}
```

### 4. Is this an API endpoint?
If handling HTTP requests and responses, place in `/routes/[feature].py`.

#### Route Handler: Good vs Bad

**❌ BAD: Route with business logic**
```python
POST /users:
  # Business logic in route handler
  validate email format
  hash password
  check if user exists
  create user in database
  send welcome email
  return response
```

**✅ GOOD: Route delegating to service**
```python
POST /users:
  # Use schema for validation
  validate with UserCreate schema

  # Guard clause for basic validation
  if validation errors:
    return error response

  # Delegate all business logic to service
  return service.create_user(validated_data)
```

#### API Route Error Handling

**❌ BAD: Handling errors in each route**
```python
@router.get("/users/{user_id}")
async def get_user(user_id: str):
  try:
    user = await user_service.get_user(user_id)
    return user
  except NotFoundException:
    return JSONResponse(status_code=404, content={"error": "User not found"})
  except ValidationException:
    return JSONResponse(status_code=400, content={"error": "Invalid user ID"})
  except Exception:
    return JSONResponse(status_code=500, content={"error": "Internal server error"})
```

**✅ GOOD: Using global exception handlers**
```python
# In main.py - register global exception handlers
@app.exception_handler(NotFoundException)
async def not_found_handler(request, exc):
  return JSONResponse(status_code=404, content={"error": exc.message})

@app.exception_handler(ValidationException)
async def validation_handler(request, exc):
  return JSONResponse(status_code=400, content={"error": exc.message})

# In route handler - let exceptions propagate
@router.get("/users/{user_id}")
async def get_user(user_id: str):
  # No try/except - let global handlers handle exceptions
  return await user_service.get_user(user_id)
```

### 5. Is this dependency management or resource provisioning?
If setting up shared resources, managing object lifecycles, or wiring dependencies, place in `/dependencies.py`.

**Examples:**
- Database connection providers
- Service instantiation with dependency injection
- External client setup (Redis, S3, etc.)
- Authentication/authorization dependencies
- Resource cleanup with yield patterns

#### Dependency Injection: Good vs Bad

**❌ BAD: Manual dependency injection with None checks**
```python
ProcessingService:
  def __init__(self, calculation_service=None, email_service=None):
    self.calculation_service = calculation_service
    self.email_service = email_service

  async def process_data(self, data):
    if not self.calculation_service:
      logger.error("Calculation service not available")
      return False
    # Business logic with guard clauses everywhere
```

**✅ GOOD: FastAPI dependency injection**
```python
# In dependencies.py
def get_calculation_service() -> CalculationService:
  return CalculationService()

def get_processing_service(
  calc_service: CalculationService = Depends(get_calculation_service)
) -> ProcessingService:
  return ProcessingService(calculation_service=calc_service)

# In routes
@router.post("/process")
async def process_data(
  data: ProcessingRequest,
  service: ProcessingService = Depends(get_processing_service)
):
  # Dependencies guaranteed to exist, no None checks needed
  return await service.process_data(data)
```

**Schema-Aware Dependency Injection:**
```python
# app/dependencies.py
from app.config import get_settings
from app.repositories.user_repository import UserRepository

def get_user_repository() -> UserRepository:
    settings = get_settings()
    return UserRepository(
        supabase_client=await get_supabase_client(),
        schema=settings.DATABASE_SCHEMA
    )

def get_user_service(
    user_repo: UserRepository = Depends(get_user_repository)
) -> UserService:
    return UserService(user_repository=user_repo)
```

**Conceptual pattern:**
```
Dependencies create injection chain:
  Database Client + Schema -> Repository -> Service -> Route

FastAPI resolves automatically:
  route declares service dependency
  service declares repository dependency
  repository declares client + schema dependencies
  FastAPI provides all dependencies in correct order
```

## Async First Development

FastAPI is built on async foundations, and with Supabase's async client support, we must maintain async operations throughout our entire application stack. This "async all the way down" approach ensures maximum performance benefits.

### Must Be Async
- **Routes**: All endpoint handlers should use `async def`
- **Services**: Business logic functions that call repositories
- **Repositories**: All Supabase database operations
- **External API Calls**: Use async HTTP clients like `httpx` instead of `requests`
- **Dependencies**: Use `async def` for dependencies that perform I/O

### Don't Need to Be Async
- **Pydantic models/schemas**: Pure data structures with no I/O
- **Utility functions**: For data transformation with no I/O operations
- **Constants and configuration**: Static values and settings

### Maintaining the Async Chain

Breaking the async chain with a synchronous operation will block the entire event loop, negating FastAPI's performance benefits. Always ensure:

```python
# Route (async)
@router.get("/items/{item_id}")
async def get_item(
    item_id: str,
    item_service: ItemService = Depends(get_item_service)
):
    # Service call (async)
    return await item_service.get_item(item_id)

# Service (async)
async def get_item(self, item_id: str):
    # Repository call (async)
    return await self.item_repository.get_item(item_id)

# Repository (async)
async def get_item(self, item_id: str):
    # Supabase call (async)
    response = await self.supabase.from_("items").select("*").eq("id", item_id).execute()
    return response.data[0] if response.data else None
```

## Standardized Repository Pattern

To eliminate repetitive CRUD code, implement a `BaseRepository` that all entity repositories inherit from:

```python
# app/repositories/base_repository.py
from typing import Any, Dict, List, Optional
from supabase._async.client import AsyncClient

class BaseRepository:
    def __init__(self, supabase_client: AsyncClient, table_name: str, schema: str):
        self.supabase = supabase_client
        self.table_name = table_name
        self.schema = schema

    async def get_by_id(self, id: str) -> Optional[Dict[str, Any]]:
        response = await self.supabase.schema(self.schema).table(self.table_name)\
            .select("*")\
            .eq("id", id)\
            .execute()
        return response.data[0] if response.data else None

    async def list_all(self, limit: int = 100) -> List[Dict[str, Any]]:
        response = await self.supabase.schema(self.schema).table(self.table_name)\
            .select("*")\
            .limit(limit)\
            .execute()
        return response.data

    async def create(self, data: Dict[str, Any]) -> Dict[str, Any]:
        response = await self.supabase.schema(self.schema).table(self.table_name)\
            .insert(data)\
            .execute()
        return response.data[0]

    async def update(self, id: str, data: Dict[str, Any]) -> Dict[str, Any]:
        response = await self.supabase.schema(self.schema).table(self.table_name)\
            .update(data)\
            .eq("id", id)\
            .execute()
        return response.data[0]

    async def delete(self, id: str) -> bool:
        response = await self.supabase.schema(self.schema).table(self.table_name)\
            .delete()\
            .eq("id", id)\
            .execute()
        return len(response.data) > 0
```

Entity repositories then simply inherit this pattern:

```python
# app/repositories/user_repository.py
from app.repositories.base_repository import BaseRepository

class UserRepository(BaseRepository):
    def __init__(self, supabase_client, schema: str):
        super().__init__(supabase_client, "users", schema)

    # Only add custom methods beyond standard CRUD
    async def get_by_email(self, email: str):
        response = await self.supabase.schema(self.schema).table(self.table_name)\
            .select("*")\
            .eq("email", email)\
            .execute()
        return response.data[0] if response.data else None
```

## Supabase Integration

### Async Client Setup

Properly initialize the async Supabase client as a dependency:

```python
# app/supabase.py
from supabase._async.client import AsyncClient, create_client
from app.config import get_settings

_supabase_client: AsyncClient = None

async def get_supabase_client() -> AsyncClient:
    """Get or create Supabase client singleton"""
    global _supabase_client
    if _supabase_client is None:
        settings = get_settings()
        _supabase_client = await create_client(
            supabase_url=settings.SUPABASE_URL,
            supabase_key=settings.SUPABASE_SECRET_KEY.get_secret_value()
        )
    return _supabase_client

# app/main.py
from fastapi import FastAPI
from app.supabase import get_supabase_client

app = FastAPI()

@app.on_event("startup")
async def startup_event():
    # Initialize client on startup
    await get_supabase_client()
```

## Centralized Error Handling

To eliminate repetitive try/catch blocks, never nest try catch blocks in other try catch blocks, use centralized error handling:

### Minimal Exception Hierarchy

Define a minimal set of application exceptions that cover common error scenarios:

```python
# app/exceptions.py
class AppException(Exception):
    def __init__(self, message: str, status_code: int = 500):
        self.message = message
        self.status_code = status_code
        super().__init__(self.message)

class BadRequestException(AppException):
    def __init__(self, message: str = "Bad Request"):
        super().__init__(message, 400)

class UnauthorizedException(AppException):
    def __init__(self, message: str = "Unauthorized"):
        super().__init__(message, 401)

class ForbiddenException(AppException):
    def __init__(self, message: str = "Forbidden"):
        super().__init__(message, 403)

class NotFoundException(AppException):
    def __init__(self, message: str = "Not Found"):
        super().__init__(message, 404)

class ServiceException(AppException):
    def __init__(self, message: str = "Internal Server Error"):
        super().__init__(message, 500)

class DatabaseServiceException(ServiceException):
    def __init__(self, message: str = "Database Error"):
        super().__init__(message)

class ExternalServiceException(ServiceException):
    def __init__(self, message: str = "External Service Error"):
        super().__init__(message)
```

### Global Exception Handlers

```python
# app/main.py
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse
from app.exceptions import AppException

app = FastAPI()

@app.exception_handler(AppException)
async def app_exception_handler(request: Request, exc: AppException):
    return JSONResponse(
        status_code=exc.status_code,
        content={"error": exc.message}
    )
```

### Error Flow Through Layers

**Repository Layer:**
- Transform external errors into application exceptions
- Raise specific exceptions for common scenarios
- Don't catch exceptions unless you can handle them meaningfully

**Service Layer:**
- Implement business logic for handling errors
- Catch and handle non-critical errors with fallbacks
- Let critical errors propagate to the API layer

**API Layer:**
- Focus on input validation
- Let service exceptions propagate to global handlers

### When to Catch vs. When to Propagate

**Catch errors when:**
- You can provide a meaningful fallback
- The error is non-critical to the operation
- You need to transform the error type
- You need to add context to the error

**Let errors propagate when:**
- You can't provide a meaningful fallback
- The error is critical to the operation
- A higher layer has more context to handle it
- The error should be converted to an HTTP response

## Using Guard Clauses

Guard clauses help maintain clean code with minimal nesting:

**❌ BAD: Deeply nested conditionals**
```python
function process_item(item_id):
  if item_id valid:
    item = get_item(item_id)
    if item exists:
      if item is active:
        if user has permission:
          # Process the item
        else:
          # Error - no permission
      else:
        # Error - item not active
    else:
      # Error - item not found
  else:
    # Error - invalid item_id
```

**✅ GOOD: Guard clauses for early returns**
```python
function process_item(item_id):
  # Guard clauses first - exit early
  if not item_id valid:
    raise error "Invalid item ID"

  item = get_item(item_id)
  if not item exists:
    raise error "Item not found"

  if not item is active:
    raise error "Item not active"

  if not user has permission:
    raise error "No permission"

  # Happy path last without nesting
  # Process the item
```

## Comprehensive Logging with Rich

Rich provides beautiful, structured logging that makes debugging and monitoring easier:

```python
# app/logger.py
import logging
from rich.console import Console
from rich.logging import RichHandler
from app.config import get_settings

class LoggerFactory:
    @staticmethod
    def get_logger(name: str = None):
        settings = get_settings()

        # Configure logging with Rich
        console = Console()
        logging.basicConfig(
            level=settings.LOG_LEVEL,
            format="%(message)s",
            datefmt="[%X]",
            handlers=[RichHandler(console=console, rich_tracebacks=True)]
        )

        # Get or create logger
        logger = logging.getLogger(name or "app")
        return logger

# Usage in any module
from app.logger import LoggerFactory
logger = LoggerFactory.get_logger(__name__)

logger.info("Processing request")
logger.debug({"user_id": user_id, "action": "login"})
try:
    # Some operation
    pass
except Exception as e:
    logger.exception(f"Error processing request: {e}")
```

## Naming Conventions

1. **Files and Folders:** Lowercase with underscores (`user_service.py`)
2. **Classes:** PascalCase (`UserService`)
3. **Functions and Methods:** Lowercase with underscores (`get_user`)
4. **Variables:** Lowercase with underscores (`user_data`)
5. **Constants:** UPPERCASE with underscores (`MAX_ITEMS`)
6. **API Routes:** Plural nouns (`/users`, `/items`)

## Package Management

Use uv for dependency management and command execution:

### Basic Commands
```bash
# Install all dependencies
uv sync

# Add a new package
uv add package-name

# Run commands in the virtual environment
uv run uvicorn app.main:app --reload
uv run pytest
```

### Typical Workflow
1. **Setup**: `uv sync` to install dependencies
2. **Development**: Add packages with `uv add` as needed
3. **Run**: Use `uv run` for all command execution

## Project Configuration

Projects use PEP 621 format in `pyproject.toml`:

```toml
[project]
name = "your-project-name"
version = "0.1.0"
description = "FastAPI + Supabase Backend"
authors = [{name = "Your Name", email = "your.email@example.com"}]
requires-python = ">=3.9"
dependencies = [
    "fastapi",
    "uvicorn",
    "supabase",
    "pydantic",
    "pydantic-settings",
    "httpx",
    "rich",
]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.hatch.build.targets.wheel]
packages = ["app"]
```

**Important**: The `[tool.hatch.build.targets.wheel]` section is required to specify which directory contains your Python package.

## Environment Configuration

Use Pydantic's settings management for type-safe configuration across environments:

```python
# app/config.py
from pydantic import SecretStr
from pydantic_settings import BaseSettings, SettingsConfigDict
from functools import lru_cache
from enum import Enum

class Environment(str, Enum):
    DEVELOPMENT = "development"
    STAGING = "staging"
    PRODUCTION = "production"

class Settings(BaseSettings):
    # API Settings
    API_TITLE: str = "FastAPI Supabase API"
    API_VERSION: str = "v1"

    # Environment
    ENVIRONMENT: Environment = Environment.DEVELOPMENT
    DEBUG: bool = ENVIRONMENT == Environment.DEVELOPMENT

    # Supabase
    SUPABASE_URL: str
    SUPABASE_SECRET_KEY: SecretStr
    DATABASE_SCHEMA: str

    # Logging
    LOG_LEVEL: str = "DEBUG" if DEBUG else "INFO"

    # Set environment file based on environment variable
    model_config = SettingsConfigDict(env_file=".env")

@lru_cache()
def get_settings() -> Settings:
    """Return cached settings to avoid reading .env on every call"""
    return Settings()
```

### Health Check Endpoint

Add a simple health check endpoint that verifies Supabase connectivity:

```python
@app.get("/health")
async def health_check():
    """Simple health check endpoint that doesn't require Supabase"""
    try:
        # Just check if the application is running
        return {
            "status": "healthy",
            "timestamp": datetime.datetime.now().isoformat(),
            "version": "1.0.0"
        }
    except Exception as e:
        logger.error(f"Health check failed: {e}")
        return {
            "status": "unhealthy",
            "error": str(e),
            "timestamp": datetime.datetime.now().isoformat()
        }
```

If your Supabase instance doesn't have a `ping` RPC function, you can use an alternative lightweight query:

```python
# Alternative lightweight health check
result = await supabase.from_('some_small_table').select('count(*)', count=CountMethod.exact).limit(1).execute()
```