# Fast API (vs Django)

Here’s a **comprehensive summary** of all concepts and interview-relevant material discussed in this chat, structured for quick review and note-taking:

---

## **Backend Engineering Concepts for 5+ Years Experience**

## **1. Python Frameworks: FastAPI, Django, Flask**

- **Django:** Full-stack, batteries-included, ORM, admin, forms, templating. Suited for large, database-driven web apps.
- **Flask:** Minimal, flexible micro-framework. Good for custom apps, prototypes. Extensible via plugins.
- **FastAPI:** Modern, async-first, API-focused. Leverages Python type hints and Pydantic models for validation, automatic docs (Swagger/ReDoc), dependency injection, superior async support.

## **Architectural Comparison Table**

| Feature/Aspect | **Django** | **Flask** | **FastAPI** |
| --- | --- | --- | --- |
| Type | Full-stack | Microframework | API-centric |
| ORM | Yes | No | No |
| Admin Interface | Yes | No | No |
| Async Support | Partial | None | Native |
| Data Validation | Forms | Manual/Extension | Pydantic/Type-hints |
| Auto API Docs | No | No | Yes (Swagger, ReDoc) |
| Built-in Security | Yes | No (manual) | OAuth/JWT modules |
| Best Use Cases | Web apps, CMS, ERP | Custom, prototyping | APIs, microservices |

---

## **2. Core Interview/Production Topics**

- **Backend scalability:** Building and deploying scalable services using Python (FastAPI, Flask, Django).
- **SQL Optimization:** Strong SQL design (PostgreSQL, MySQL), indexing, migrations, advanced queries.
- **API Design:** RESTful principles, versioning, pagination, filtering, error handling, idempotency.
- **Authentication:** JWT/OAuth implementation and security best practices.
- **Microservices Concepts:** Circuit breaker, Saga, API gateway, service discovery, decentralized data.
- **System Design:** Scaling for high requests, caching, monitoring, infrastructure (Docker, cloud deployments).
- **Real-world scenarios:** DB connection management, monolith-to-microservices migration, debugging prod issues.

---

## **3. FastAPI Unique Features**

- **Automatic validation & docs:** All endpoints typed—docs generated for free.
- **Pydantic integration:** Clean, explicit data validation (inputs/outputs).
- **Dependency injection:** Modular resource management using the `Depends` keyword.
- **Async-first:** Designed for high concurrency from start.
- **Minimal boilerplate:** Concise endpoint definitions.
- **Built-in security:** OAuth2, JWT, HTTP Basic easy to set up.

---

## **4. Dependency Injection in FastAPI**

- **Concept:** Share reusable logic/resources (e.g., DB sessions, authentication) by declaring dependencies with `Depends`.
- **Pattern:** Dependency function is called, result injected as argument into your route; enables easy testing, separation, code reuse.
- **Example:**
    
    `pythondef get_db(): 
        db = Session()
        try:
            yield db
        finally:
            db.close()
    @app.get("/")
    def endpoint(db: Session = Depends(get_db)):
        *# db is ready to use*`
    
- **Benefit:** No global state or manual wiring. Cleaner, modular, and easier to test or scale.

---

## **5. The `yield` Keyword in Python**

- **Purpose:** Used in generator functions to "yield" a value, pause execution, and resume later.
- **Benefits:** Efficient for memory; useful for streaming, handling large data one item at a time.
- **Use case in FastAPI:** Cleaning up resources—setup before yield, teardown after.
- **Examples:**
    
    `pythondef countdown(n):
        while n > 0:
            yield n
            n -= 1
    *# prints 3, 2, 1 in sequence via for loop*`
    

---

## **6. Real-life `yield` Examples**

- Reading files line-by-line.
- Streaming sequence generators (Fibonacci, countdown).
- Chunking lists for batch processing.
- Filtering lists (e.g., extract odd numbers).

---

## **7. When to Choose Django vs FastAPI**

**Choose Django When:**

- Need full-stack features (ORM, admin, forms, security).
- Building large, multi-feature web apps with lots of built-ins.
- Rapid prototyping and conventional structure with mature libraries.

**Choose FastAPI When:**

- Developing RESTful APIs/microservices.
- Need high performance and async capabilities.
- Want automatic documentation and validation.
- Prefer modular, modern codebases leveraging Python type hints.

---

## **8. Core Interview Questions to Prepare**

- Advanced FastAPI functionality and architecture.
- SQL optimization, schema design, migrations.
- API design patterns, security concerns.
- Authentication workflows, JWT/OAuth mechanisms.
- Microservices, scalability, system design scenarios.

---

**Visual Summary:**

See above for the comparative architecture table.

---

**How to use this summary:**

- Quick refresh before interviews or system-design deep dives.
- Visual reference for choosing frameworks/projects.
- Reference for implementation of generators, DI, and FastAPI features.
1. [https://www.instahyre.com/job-388212-full-stack-engineer-at-butter-money-bangalore/](https://www.instahyre.com/job-388212-full-stack-engineer-at-butter-money-bangalore/)
