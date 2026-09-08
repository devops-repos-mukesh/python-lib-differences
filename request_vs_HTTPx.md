# HTTPX vs Requests: Comprehensive Documentation

## Table of Contents
1. [Overview](#overview)
2. [Installation](#installation)
3. [Key Differences](#key-differences)
4. [Feature Comparison](#feature-comparison)
5. [Code Examples](#code-examples)
6. [Use Cases](#use-cases)
7. [Migration Guide](#migration-guide)
8. [Performance Considerations](#performance-considerations)

---

## Overview

### Requests
**Requests** is a popular, easy-to-use HTTP client library for Python. It abstracts the complexities of making HTTP requests behind a simple API, allowing developers to send HTTP/1.1 requests without manually adding query strings to URLs or form-encoding POST data.

- **First Released**: 2011
- **Maintainer**: Kenneth Reitz
- **Status**: Mature and stable
- **Python Support**: 3.7+

### HTTPX
**HTTPX** is a modern, fully-featured HTTP client for Python that combines the simplicity of Requests with advanced features needed for today's web development. It supports both synchronous and asynchronous operations, HTTP/2, and is built with best practices in mind.

- **First Released**: 2019
- **Maintainer**: Tom Christie (Starlette, FastAPI creator)
- **Status**: Production-ready
- **Python Support**: 3.7+

---

## Installation

### Requests
```bash
pip install requests
```

### HTTPX
```bash
# Basic installation
pip install httpx

# With HTTP/2 support
pip install httpx[http2]

# With all optional features
pip install httpx[all]
```

---

## Key Differences

| Feature | Requests | HTTPX |
|---------|----------|-------|
| **Async Support** | ❌ No | ✅ Yes (native) |
| **HTTP/2 Support** | ❌ No | ✅ Yes |
| **HTTP/3 Support** | ❌ No | ⚠️ Experimental |
| **Synchronous** | ✅ Yes | ✅ Yes |
| **Type Hints** | ⚠️ Partial | ✅ Full |
| **Request/Response Streaming** | ✅ Yes | ✅ Yes |
| **Cookie Jar Management** | ✅ Yes | ✅ Yes |
| **Auth Handling** | ✅ Basic, Digest, OAuth2 | ✅ Basic, Digest, Bearer |
| **Timeout Support** | ✅ Simple | ✅ Advanced (per-operation) |
| **Dependency Size** | Minimal | Slightly larger |
| **Learning Curve** | Very gentle | Gentle |
| **Active Development** | ✅ Maintenance mode | ✅ Active |

---

## Feature Comparison

### 1. Asynchronous Support

**Requests**: No native async support
```python
# Would need to use external libraries like asyncio + requests
import asyncio
import concurrent.futures
import requests

def fetch(url):
    return requests.get(url)

async def async_fetch(urls):
    with concurrent.futures.ThreadPoolExecutor() as executor:
        futures = [executor.submit(fetch, url) for url in urls]
        return [f.result() for f in futures]
```

**HTTPX**: Built-in async support
```python
import asyncio
import httpx

async def async_fetch(urls):
    async with httpx.AsyncClient() as client:
        tasks = [client.get(url) for url in urls]
        return await asyncio.gather(*tasks)

asyncio.run(async_fetch(urls))
```

### 2. HTTP/2 Support

**Requests**: Not supported

**HTTPX**: Full HTTP/2 support
```python
import httpx

# HTTP/2 is used automatically when available
with httpx.Client(http2=True) as client:
    response = client.get("https://www.example.com")
    print(response.http_version)  # "HTTP/2"
```

### 3. Type Hints

**Requests**: Minimal type hints (added gradually)

**HTTPX**: Comprehensive type hints
```python
import httpx
from typing import Optional

def make_request(
    url: str,
    headers: Optional[dict[str, str]] = None,
    timeout: float = 30.0
) -> httpx.Response:
    with httpx.Client() as client:
        return client.get(url, headers=headers, timeout=timeout)
```

### 4. Timeout Handling

**Requests**: Single timeout value
```python
import requests

# All operations use same timeout
response = requests.get(url, timeout=30)
```

**HTTPX**: Granular timeout control
```python
import httpx

# Connect, read, write, and pool timeouts separately
timeout = httpx.Timeout(
    timeout=30,
    connect=10.0,
    read=20.0,
    write=10.0,
    pool=5.0
)
with httpx.Client(timeout=timeout) as client:
    response = client.get(url)
```

### 5. Client Configuration

**Requests**: Session-based
```python
import requests

session = requests.Session()
session.headers.update({"User-Agent": "MyApp/1.0"})
response = session.get(url)
```

**HTTPX**: Client-based (similar but more modern)
```python
import httpx

with httpx.Client(
    headers={"User-Agent": "MyApp/1.0"},
    base_url="https://api.example.com"
) as client:
    response = client.get("/users")
```

---

## Code Examples

### Basic GET Request

**Requests**
```python
import requests

response = requests.get("https://api.github.com/users/octocat")
print(response.status_code)
print(response.json())
```

**HTTPX**
```python
import httpx

response = httpx.get("https://api.github.com/users/octocat")
print(response.status_code)
print(response.json())
```

### POST Request with Data

**Requests**
```python
import requests

data = {"name": "John", "email": "john@example.com"}
response = requests.post(
    "https://api.example.com/users",
    json=data,
    headers={"Authorization": "Bearer token"}
)
print(response.json())
```

**HTTPX**
```python
import httpx

data = {"name": "John", "email": "john@example.com"}
response = httpx.post(
    "https://api.example.com/users",
    json=data,
    headers={"Authorization": "Bearer token"}
)
print(response.json())
```

### Handling Sessions/Clients

**Requests**
```python
import requests

session = requests.Session()
session.headers.update({"Authorization": "Bearer token"})

# Reuse session for multiple requests
response1 = session.get("https://api.example.com/users")
response2 = session.get("https://api.example.com/posts")

session.close()
```

**HTTPX**
```python
import httpx

with httpx.Client(headers={"Authorization": "Bearer token"}) as client:
    response1 = client.get("https://api.example.com/users")
    response2 = client.get("https://api.example.com/posts")
    # Client is automatically closed
```

### Streaming Responses

**Requests**
```python
import requests

response = requests.get("https://example.com/large-file", stream=True)
with open("file.bin", "wb") as f:
    for chunk in response.iter_content(chunk_size=8192):
        f.write(chunk)
```

**HTTPX**
```python
import httpx

with httpx.stream("GET", "https://example.com/large-file") as response:
    with open("file.bin", "wb") as f:
        for chunk in response.iter_bytes(chunk_size=8192):
            f.write(chunk)
```

### Async Requests

**Requests**: Not supported natively

**HTTPX**
```python
import asyncio
import httpx

async def fetch_user(user_id):
    async with httpx.AsyncClient() as client:
        response = await client.get(f"https://api.example.com/users/{user_id}")
        return response.json()

async def main():
    results = await asyncio.gather(
        fetch_user(1),
        fetch_user(2),
        fetch_user(3)
    )
    print(results)

asyncio.run(main())
```

### Error Handling

**Requests**
```python
import requests
from requests.exceptions import RequestException, Timeout, ConnectionError

try:
    response = requests.get(url, timeout=5)
    response.raise_for_status()
except Timeout:
    print("Request timed out")
except ConnectionError:
    print("Connection error")
except RequestException as e:
    print(f"Error: {e}")
```

**HTTPX**
```python
import httpx

try:
    response = httpx.get(url, timeout=5)
    response.raise_for_status()
except httpx.TimeoutException:
    print("Request timed out")
except httpx.ConnectError:
    print("Connection error")
except httpx.HTTPError as e:
    print(f"Error: {e}")
```

---

## Use Cases

### Use Requests When:
- Building simple scripts or applications
- Working with synchronous-only code
- You need maximum compatibility
- HTTP/1.1 is sufficient for your needs
- Working on legacy projects
- You prefer a battle-tested, mature library
- You need the largest community support

### Use HTTPX When:
- Building modern async applications
- You need HTTP/2 support
- Type hints are important for your codebase
- You want fine-grained timeout control
- Building high-performance applications
- You prefer a more opinionated, modern design
- Working with async frameworks (FastAPI, Starlette, etc.)

---

## Migration Guide

### From Requests to HTTPX

The APIs are very similar, making migration straightforward:

```python
# Requests
import requests
response = requests.get(url)

# HTTPX equivalent
import httpx
response = httpx.get(url)
```

**Key changes to watch for:**

1. **Exception names**
   ```python
   # Requests
   from requests.exceptions import Timeout
   
   # HTTPX
   from httpx import TimeoutException
   ```

2. **Client context manager**
   ```python
   # Requests
   session = requests.Session()
   # ... use session ...
   session.close()
   
   # HTTPX (cleaner)
   with httpx.Client() as client:
       # ... use client ...
   ```

3. **Response streaming**
   ```python
   # Requests
   for chunk in response.iter_content():
       pass
   
   # HTTPX
   for chunk in response.iter_bytes():
       pass
   ```

4. **Async operations**
   ```python
   # HTTPX async
   async with httpx.AsyncClient() as client:
       response = await client.get(url)
   ```

---

## Performance Considerations

### Requests
- **Pros**: Lightweight, minimal dependencies
- **Cons**: No HTTP/2, no native async (requires thread pool for concurrent requests)
- **Best for**: Simple use cases, single requests

### HTTPX
- **Pros**: HTTP/2, native async, connection pooling optimized
- **Cons**: Slightly larger footprint, small learning curve for advanced features
- **Best for**: High-performance applications, concurrent requests, modern APIs

### Benchmark Example

**Concurrent requests to 10 URLs**

```python
# Using Requests (with ThreadPoolExecutor)
import requests
from concurrent.futures import ThreadPoolExecutor
import time

urls = [f"https://httpbin.org/delay/1" for _ in range(10)]
start = time.time()
with ThreadPoolExecutor(max_workers=10) as executor:
    responses = list(executor.map(requests.get, urls))
print(f"Requests: {time.time() - start:.2f}s")

# Using HTTPX (with async)
import asyncio
import httpx

async def fetch_all():
    async with httpx.AsyncClient() as client:
        tasks = [client.get(url) for url in urls]
        return await asyncio.gather(*tasks)

start = time.time()
asyncio.run(fetch_all())
print(f"HTTPX: {time.time() - start:.2f}s")
```

HTTPX typically shows better performance for concurrent operations due to native async support and HTTP/2.

---

## Conclusion

| Aspect | Requests | HTTPX |
|--------|----------|-------|
| **Maturity** | Mature | Production-ready |
| **Modern Features** | Limited | Excellent |
| **Async Support** | No | Yes |
| **HTTP/2** | No | Yes |
| **Learning Curve** | Minimal | Low |
| **Community** | Very large | Growing |
| **Maintenance** | Stable | Active |

**Choose Requests if** you need maximum simplicity and compatibility.
**Choose HTTPX if** you're building modern applications with async/HTTP/2 needs.

---

## References
- [Requests Documentation](https://requests.readthedocs.io/)
- [HTTPX Documentation](https://www.python-httpx.org/)
- [HTTPX GitHub Repository](https://github.com/encode/httpx)
- [HTTP/2 RFC 7540](https://tools.ietf.org/html/rfc7540)
