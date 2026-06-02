# FastAPI Dependency Injection (Depends) - Comprehensive Summary

# 1. What is `Depends`?

- **`Depends` is FastAPI's Dependency Injection (DI) marker class**
- Signals to FastAPI: "Execute this function as a dependency and resolve its parameters automatically"
- Not optional - you can't just pass a function reference
- Works by inspecting function signatures and building a dependency resolution tree

### Key Syntax:

```python
# ❌ Wrong - just a function reference
router.dependencies = [verify_s2s_token]

# ✅ Correct - wrapped in Depends
router.dependencies = [Depends(verify_s2s_token)]

```

---

## 2. How Dependencies Work - The Complete Flow

### Router-Level Dependencies (Your Code Example):

```python
# Line 184-283: Dependency function
async def verify_s2s_token(
    x_s2s_token: Optional[str] = Header(None, alias="x-s2s-token"),
    x_atwork_token: Optional[str] = Header(None, alias="x-atwork-access-token"),
):
    # Validates JWT tokens
    # Checks user roles (SUPER_ADMIN, DEVELOPER)
    # Raises HTTPException if invalid

# Line 285-287: Applied to all endpoints in router
router.dependencies = [Depends(verify_s2s_token)]

```

### Execution Flow:

```
┌─────────────────────────────────────┐
│ 1. HTTP Request Arrives             │
│    GET /analytics/logs              │
│    Headers: x-s2s-token: eyJ...     │
└─────────────────────────────────────┘
            ↓
┌─────────────────────────────────────┐
│ 2. FastAPI Finds Route              │
│    Matches endpoint                 │
└─────────────────────────────────────┘
            ↓
┌─────────────────────────────────────┐
│ 3. Check Router Dependencies        │
│    Found: Depends(verify_s2s_token) │
└─────────────────────────────────────┘
            ↓
┌─────────────────────────────────────┐
│ 4. Execute Dependency               │
│    - Extract headers from request   │
│    - Call verify_s2s_token()        │
│    - Validate JWT                   │
│    - Check roles                    │
└─────────────────────────────────────┘
            ↓
      ┌─────┴─────┐
      │           │
  ✅ Pass      ❌ Fail
      │           │
      │           └─→ HTTPException (401/403)
      │               Return error
      │               ENDPOINT NEVER RUNS
      ↓
┌─────────────────────────────────────┐
│ 5. Resolve Endpoint Parameters      │
│    Extract Query/Body/Path params   │
└─────────────────────────────────────┘
            ↓
┌─────────────────────────────────────┐
│ 6. Execute Endpoint Function        │
│    Your business logic runs         │
└─────────────────────────────────────┘
            ↓
┌─────────────────────────────────────┐
│ 7. Return Response                  │
└─────────────────────────────────────┘

```

---

## 3. Who Calls Your Functions? (Critical Concept)

### Answer: **FastAPI's Request Handler** - NOT YOU!

**At Startup:**

```
FastAPI initialization
    ↓
Scans all routes and dependencies
    ↓
Inspects function signatures
    ↓
Discovers parameters and their types (Query, Header, Body, etc.)
    ↓
Builds dependency resolution tree
    ↓
Stores execution plan

```

**At Runtime (Per Request):**

```
FastAPI Request Handler (pseudocode):
    1. Parse incoming request
    2. Extract headers, query params, body
    3. Resolve dependencies in order
    4. Call dependency functions with extracted values
    5. Call endpoint function with resolved parameters
    6. Return response

```

### You Never Call These Functions Directly:

```python
# ❌ You DON'T write:
token = await verify_s2s_token(
    request.headers.get("x-s2s-token"),
    request.headers.get("x-atwork-access-token")
)

# ✅ FastAPI does it automatically:
router.dependencies = [Depends(verify_s2s_token)]

```

---

## 4. Parameter Types - Query, Body, Header

### `Query()` - Extract from URL Query String

```python
@router.get("/analytics/inference-report")
async def get_inference_report(
    format: str = Query("txt"),  # ← Not a default value!
    start_date_time: str = Query(None),  # ← Dependency instruction!
):

```

**What it means:**

- Extract from URL: `?format=json&start_date_time=2024-01-01`
- Used with: GET, DELETE requests
- Individual parameters

**Request Example:**

```
GET /analytics/inference-report?format=json&start_date_time=2024-01-01

```

**Inside Function (receives actual values):**

```python
print(format)  # "json" (string)
print(start_date_time)  # "2024-01-01" (string)

```

### `Body()` - Extract from Request Body

```python
@router.post("/analytics/pipeline")
async def run_analytics_pipeline(
    request: dict = Body(default=None),  # ← Entire JSON body
):

```

**What it means:**

- Extract from request body as JSON
- Used with: POST, PUT, PATCH requests
- Returns entire parsed dict

**Request Example:**

```
POST /analytics/pipeline
Content-Type: application/json

{
  "start_date_time": "2024-01-01T00:00:00Z",
  "last_24_hours": true
}

```

**Access Pattern:**

```python
last_24_hours = request.get("last_24_hours", False) if request else False

```

### `Header()` - Extract from HTTP Headers

```python
async def verify_s2s_token(
    x_s2s_token: Optional[str] = Header(None, alias="x-s2s-token"),
):

```

**What it means:**

- Extract from HTTP headers
- `alias` maps header name (can include hyphens)

### Comparison Table:

| Type | Source | HTTP Methods | Structure | Access |
| --- | --- | --- | --- | --- |
| `Query()` | URL query string | GET, DELETE | Individual params | Direct variable |
| `Body()` | Request body | POST, PUT, PATCH | Single dict | `request.get()` |
| `Header()` | HTTP headers | Any | Individual values | Direct variable |
| `Path()` | URL path | Any | Individual values | Direct variable |

---

## 5. The "Magic Trick" - How Query() Actually Works

### What You Write (Declaration):

```python
async def get_inference_report(
    format: str = Query("txt"),  # ← Looks like default value
):

```

### What FastAPI Interprets:

```python
# "This parameter comes from query string named 'format'
#  Default value if missing: 'txt'
#  Type: str"

```

### What Your Function Receives (Runtime):

```python
# format is NOT a Query object!
# It's an actual string: "json"
format: str = "json"  # ← Real value from ?format=json

# You work with normal Python values:
if format == "json":  # This works!
    return JSONResponse(...)

```

### Visual Representation:

```
Definition Time:
    format: str = Query("txt")
            ↓
    [FastAPI reads Query instruction]
            ↓
    [Builds: "Extract 'format' from query string, default='txt'"]

Runtime:
    Request: ?format=json
            ↓
    [FastAPI extracts "json"]
            ↓
    format: str = "json"  # ← Your function receives this

```

---

## 6. Flow Without Router Dependencies (Insecure)

```
HTTP Request
    ↓
Route Matching ✅
    ↓
Check Router Dependencies → EMPTY! ⚠️
    ↓
verify_s2s_token() NEVER CALLED ❌
    ↓
No Authentication ⚠️
No Token Validation ❌
Headers Ignored ⚠️
    ↓
Extract Query Parameters ✅
    ↓
Execute Endpoint ✅ (Anyone can access!)
    ↓
Return Data 🚨 (Public endpoint!)

```

### Key Differences:

- ✅ Route matching works
- ✅ Query/Body extraction works
- ❌ NO security checks
- ❌ Headers ignored
- ⚠️ Endpoint executes regardless

---

## 7. Dependency Chains (Powerful Feature)

Dependencies can depend on other dependencies:

```python
# Chain of dependencies
async def get_token(x_token: str = Header(...)):
    return x_token

async def get_user(token: str = Depends(get_token)):  # ← Depends on get_token
    return jwt.decode(token, ...)

async def get_permissions(user: dict = Depends(get_user)):  # ← Depends on get_user
    return db.permissions.find({"user_id": user['id']})

@router.get("/admin")
async def admin_panel(
    permissions: list = Depends(get_permissions)  # ← Chains all 3!
):
    # FastAPI automatically: get_token → get_user → get_permissions
    pass

```

**Execution Order:**

```
get_token() → get_user(token) → get_permissions(user) → admin_panel(permissions)

```

---

## 8. Other Use Cases of `Depends`

### Database Connections:

```python
async def get_db():
    db = mongodb.db
    yield db  # Cleanup possible

@router.get("/users")
async def get_users(db = Depends(get_db)):
    return await db["users"].find({})

```

### Pagination (Reusable Pattern):

```python
class PaginationParams:
    def __init__(
        self,
        page: int = Query(1, ge=1),
        limit: int = Query(100, ge=1, le=1000),
    ):
        self.page = page
        self.limit = limit
        self.skip = (page - 1) * limit

@router.get("/users")
async def get_users(pagination: PaginationParams = Depends()):
    # Reusable pagination logic!
    return db.users.find().skip(pagination.skip).limit(pagination.limit)

```

### Rate Limiting:

```python
async def check_rate_limit(x_api_key: str = Header(...)):
    if await redis.get(f"rate:{x_api_key}") > 100:
        raise HTTPException(429, "Rate limit exceeded")
    return True

@router.get("/api")
async def api_endpoint(_: bool = Depends(check_rate_limit)):
    # Automatically rate-limited!
    pass

```

### Logging/Monitoring:

```python
async def log_request(request: Request):
    start = time.time()
    logger.info(f"Started: {request.method} {request.url}")
    yield
    logger.info(f"Finished in {time.time() - start:.2f}s")

router.dependencies = [Depends(log_request)]  # All endpoints logged!

```

### Feature Flags:

```python
async def check_feature_enabled(
    feature: str,
    user: dict = Depends(get_current_user),
):
    if not is_enabled(feature, user):
        raise HTTPException(403, "Feature not available")
    return True

```

---

## 9. Real Benefits of Using `Depends`

### 1. 🧪 **Testability** (HUGE!)

```python
# Easy to mock in tests
def test_endpoint():
    app.dependency_overrides[get_db] = lambda: MockDB()
    response = client.get("/users")
    # Test with mock database!

```

### 2. ♻️ **Code Reusability (DRY)**

```python
# Instead of repeating in every endpoint:
analytics_service = AnalyticsService(mongodb.db)  # Repeated everywhere ❌

# Write once, use everywhere:
def get_analytics_service():
    return AnalyticsService(mongodb.db)

# Use in multiple endpoints:
async def endpoint(service = Depends(get_analytics_service)): ...  # ✅

```

### 3. 🐛 **Better Debugging**

- Centralized error handling
- Add logging at dependency level
- Affects all endpoints using that dependency

### 4. 🔄 **Easy Swapping**

```python
def get_email_service():
    if settings.ENV == "production":
        return RealEmailService()
    else:
        return FakeEmailService()  # Dev/test

```

### 5. 🎯 **Separation of Concerns**

```python
# Without Depends - mixed concerns:
async def endpoint():
    # Auth + rate limit + logging + business logic all mixed ❌

# With Depends - clean separation:
async def endpoint(
    user = Depends(get_user),        # Auth
    _ = Depends(check_rate_limit),   # Rate limiting
    _ = Depends(log_request),        # Logging
):
    # Only business logic here! ✅

```

### 6. 🔒 **Consistent Security**

- Apply auth once at router level
- All endpoints automatically protected
- Can't forget to add auth

### 7. 📊 **Performance**

```python
@lru_cache()  # Cache result
def get_settings():
    return Settings()  # Loaded only once!

```

### 8. 🔧 **Maintainability**

- Change logic in one place
- Affects all endpoints using it
- Easy to update/refactor

---

## 10. Key Concepts Summary

### Router-Level vs Endpoint-Level:

```python
# Router-level (applies to ALL endpoints):
router.dependencies = [Depends(verify_s2s_token)]

# Endpoint-level (applies to one endpoint):
@router.get("/endpoint")
async def endpoint(token = Depends(verify_s2s_token)):
    pass

```

### Parameter Resolution Order:

```
1. Router dependencies (security)
2. Endpoint parameter dependencies (Query, Body, Path)
3. Endpoint function execution

```

### The Dependency Inversion Principle:

- High-level code (endpoints) doesn't depend on low-level details
- Both depend on abstractions (the `Depends` mechanism)
- Loose coupling, high cohesion

---

## 11. Your Code Refactoring Opportunity

**Current Pattern (Repetitive):**

```python
@router.get("/analytics/logs")
async def get_chat_logs(...):
    analytics_service = AnalyticsService(mongodb.db)  # Repeated

@router.post("/analytics/summary")
async def get_summary(...):
    analytics_service = AnalyticsService(mongodb.db)  # Repeated

```

**Improved Pattern (DRY):**

```python
async def get_analytics_service():
    return AnalyticsService(mongodb.db)

async def get_excluded_person_ids():
    # Extract common logic
    var_doc = await dynamic_variables.find_one(...)
    return var_doc["value"] if var_doc else []

@router.get("/analytics/logs")
async def get_chat_logs(
    service: AnalyticsService = Depends(get_analytics_service),
):
    # Clean and reusable!

```

---

## Quick Reference Card

| Concept | Syntax | Purpose |
| --- | --- | --- |
| Dependency marker | `Depends(func)` | Tell FastAPI to execute func as dependency |
| Query parameter | `Query(default)` | Extract from URL query string |
| Body parameter | `Body(default)` | Extract from request body (JSON) |
| Header parameter | `Header(default)` | Extract from HTTP headers |
| Router dependencies | `router.dependencies = [...]` | Apply to all endpoints |
| Endpoint dependency | `param = Depends(func)` | Apply to one endpoint |
| Dependency chain | `Depends(func_with_depends)` | Dependencies can depend on others |

---

**Remember:** `Depends` enables automatic dependency injection - FastAPI handles all the plumbing, you focus on business logic! 🚀
