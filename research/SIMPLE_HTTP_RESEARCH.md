# simple_http Research Report

**Date:** 2025-12-08
**Library:** simple_http (HTTP Client Wrapper)
**Current Version:** 1.0.0 (initial implementation)

---

## Executive Summary

This research report follows a systematic seven-step process to evaluate the current `simple_http` library implementation and provide recommendations for enhancement. The library currently provides basic HTTP request functionality wrapping EiffelStudio's `NET_HTTP_CLIENT`, offering GET, POST, PUT, DELETE, HEAD, and PATCH methods with configurable timeouts.

The research reveals significant gaps compared to modern HTTP client standards, particularly in areas of resilience (retry logic, circuit breakers), developer experience (fluent API, type-safe responses), and common use cases (authentication, cookie management, redirect handling). While the current implementation provides a functional foundation, substantial enhancements are recommended to achieve feature parity with contemporary HTTP clients.

---

## Step 1: Specifications Research

### HTTP/1.1 (RFC 7230-7235) - Now Superseded by RFC 9110-9114

The HTTP/1.1 specification suite was originally published as six separate RFCs in June 2014, but has been consolidated and updated:

#### Original RFC 7230-7235 Series:
1. **RFC 7230 - Message Syntax and Routing**: HTTP architecture, terminology, URI schemes (http/https)
2. **RFC 7231 - Semantics and Content**: Request methods, status codes, headers, content negotiation
3. **RFC 7232 - Conditional Requests**: ETags, If-Modified-Since, conditional GET/PUT operations
4. **RFC 7233 - Range Requests**: Partial content (Range/Content-Range headers), resume downloads
5. **RFC 7234 - Caching**: Cache-Control, Expires, Age headers, cache validation
6. **RFC 7235 - Authentication**: WWW-Authenticate, Authorization headers, Basic/Digest auth

#### Updated Specifications (June 2022):
**RFC 9110** consolidates and obsoletes RFC 7231, 7232, 7233, 7235, and portions of 7230, providing unified HTTP semantics applicable across all protocol versions.

**Key HTTP/1.1 Characteristics for Client Implementation:**
- Text-based protocol (headers in plain text)
- Persistent connections (Connection: keep-alive)
- Request pipelining (multiple requests without waiting for responses)
- Chunked transfer encoding
- Content negotiation (Accept headers)
- Conditional requests (ETags reduce bandwidth)
- Authentication framework (Bearer tokens, Basic auth)

### HTTP/2 (RFC 7540, RFC 9113)

Published May 2015, derived from Google's SPDY protocol.

**Key Improvements Over HTTP/1.1:**
- **Binary protocol**: More efficient parsing, reduced errors
- **Multiplexing**: Multiple concurrent requests/responses over single TCP connection
- **Header compression**: HPACK algorithm reduces overhead
- **Server push**: Proactive resource delivery
- **Stream prioritization**: Important requests complete faster
- **Per-stream flow control**: Better resource management

**Limitation:**
- TCP-level head-of-line blocking: Lost packets block all multiplexed streams

### HTTP/3 (RFC 9114)

Published June 2022, uses QUIC transport over UDP instead of TCP.

**Key Improvements Over HTTP/2:**
- **No head-of-line blocking**: Stream-level reliability, lost packets only affect one stream
- **Faster connection establishment**: 0-RTT or 1-RTT handshake (vs 3-way TCP + TLS)
- **Built-in encryption**: TLS 1.3 integrated into QUIC transport
- **Connection migration**: Survives network changes (Wi-Fi to cellular)
- **QPACK compression**: Header compression optimized for QUIC
- **Better congestion control**: Per-stream flow control

**Security Considerations:**
- Encrypted headers prevent MitM attacks
- 0-RTT replay attack risks (needs application-level protection)

### REST API Best Practices

**Resource Design:**
- Use nouns, not verbs (`/users` not `/getUsers`)
- Plural resource names (`/products`, `/orders`)
- Hierarchical relationships (`/users/123/orders`)
- Filter/sort/paginate with query params (`?status=active&sort=created_at&limit=20`)

**HTTP Method Semantics:**
- GET: Retrieve (safe, idempotent, cacheable)
- POST: Create (non-idempotent)
- PUT: Replace (idempotent)
- PATCH: Partial update (idempotent)
- DELETE: Remove (idempotent)

**Status Code Standards:**
- 2xx Success: 200 OK, 201 Created, 204 No Content
- 3xx Redirect: 301 Moved Permanently, 302 Found, 304 Not Modified
- 4xx Client Error: 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 429 Too Many Requests
- 5xx Server Error: 500 Internal Server Error, 502 Bad Gateway, 503 Service Unavailable

**Security Best Practices:**
- Always use HTTPS
- OAuth 2.0 or JWT for authentication
- Rate limiting with 429 + Retry-After header
- Input sanitization to prevent injection attacks

**API Design Principles:**
- Version APIs (`/v1/users` or `Accept: application/vnd.api+json;version=1`)
- Consistent error format (RFC 7807 Problem Details)
- HATEOAS for discoverability (optional)
- Compression (Accept-Encoding: gzip, br)
- Pagination for large datasets
- Field filtering/sparse fieldsets (`?fields=id,name,email`)

**Sources:**
- [RFC 9110: HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html)
- [RFC 7540: HTTP/2](https://datatracker.ietf.org/doc/html/rfc7540)
- [RFC 9114: HTTP/3](https://datatracker.ietf.org/doc/html/rfc9114)
- [Updated HTTP specifications (Axway)](https://blog.axway.com/learning-center/software-development/updated-http-specifications)
- [Azure API Design Best Practices](https://learn.microsoft.com/en-us/azure/architecture/best-practices/api-design)
- [REST API Best Practices (Stack Overflow)](https://stackoverflow.blog/2020/03/02/best-practices-for-rest-api-design/)

---

## Step 2: Tech-Stack Library Analysis

### Go: net/http (Standard) vs Resty

#### net/http (Standard Library)
**Characteristics:**
- Robust, well-suited for many use cases
- Lightweight, stable, complete control
- Verbose - manual JSON encoding/decoding, timeout handling
- Foundation for all HTTP in Go

**When to Use:**
- Small projects avoiding dependencies
- Need complete low-level control
- Standard library preference

#### Resty
**Key Features:**
1. **Chainable API**: Readable method chaining
   ```go
   client.R().
       SetHeader("Accept", "application/json").
       SetBody(user).
       Post("https://api.example.com/users")
   ```

2. **Automatic JSON Handling**: Auto marshal/unmarshal to Go structs
3. **Built-in Retry Support**: Configurable retry logic
4. **HTTP/2 Support**: Advanced features out-of-box
5. **Middleware**: Request/response middlewares, hooks
6. **Client Settings**: Timeout, RedirectPolicy, Proxy, TLSClientConfig
7. **Multipart Upload**: Easy file uploads
8. **Response to File**: Save directly like `curl -o`

**Trade-offs:**
- Slightly heavier than net/http
- External dependency
- Ideal for most projects needing quick setup

**Sources:**
- [Resty GitHub](https://github.com/go-resty/resty)
- [HTTP Requests in Go: Comparison](https://medium.com/@praveendayanithi/http-requests-in-go-a-comparison-of-client-libraries-97ab5a8cc51c)
- [Best Golang HTTP Clients](https://hayageek.com/best-golang-http-clients/)

### Rust: reqwest (High-Level) vs hyper (Low-Level)

#### hyper
**Characteristics:**
- Foundation library (reqwest built on hyper)
- Asynchronous by design
- Both client and server APIs
- Production-proven, battle-tested
- Building block, not end-user library

**Recommendation:** Use reqwest unless building custom HTTP infrastructure.

#### reqwest
**Key Features:**
1. **Async and Blocking APIs**: Both `reqwest::Client` (async/Tokio) and `reqwest::blocking`
2. **Automatic Connection Pooling**:
   - Client holds connection pool internally (uses Arc, shareable)
   - Reuses connections to same host
   - Managed by underlying hyper library

3. **Customizable Pool Settings** (via ClientBuilder):
   - `pool_max_idle_per_host()`: Max idle connections per host
   - `pool_idle_timeout()`: Idle timeout before closing
   - `timeout()`: Request timeout
   - `connect_timeout()`: Connection timeout
   - `tcp_keepalive()`: TCP keep-alive interval
   - `tcp_nodelay()`: Disable Nagle's algorithm

4. **Request Body Support**: Plain text, JSON, URL-encoded, multipart forms
5. **Customizable Redirect Policy**: Configure max redirects, policies
6. **HTTP Proxy Support**: With authentication
7. **TLS Encryption by Default**: Secure by default
8. **Cookie Management**: Optional cookie store

**Best Practice:**
- Create ONE Client instance and reuse (already uses Arc internally)
- Connection pooling automatic, no manual management needed
- Avoids overhead of establishing new connections

**Sources:**
- [reqwest GitHub](https://github.com/seanmonstar/reqwest)
- [reqwest docs](https://docs.rs/reqwest/)
- [Using connection pooling in Rust](https://app.studyraid.com/en/read/11242/350323/using-connection-pooling)
- [How to choose the right Rust HTTP client](https://blog.logrocket.com/best-rust-http-client/)

### Python: requests (Classic) vs httpx (Modern)

#### requests
**Strengths:**
- Simple, easy to use
- Purely synchronous
- Large user base, extensive tutorials
- Mature ecosystem
- Default choice for straightforward HTTP

**Limitations:**
- No native async support
- No HTTP/2 support
- Limited for high-concurrency applications

#### httpx
**Key Features:**
1. **Async AND Sync APIs**: Works with both paradigms
   ```python
   # Sync
   response = httpx.get("https://api.example.com")

   # Async
   async with httpx.AsyncClient() as client:
       response = await client.get("https://api.example.com")
   ```

2. **HTTP/2 Support**: Multiplexing multiple requests over single connection
3. **Better Performance**:
   - Faster than requests for sync operations
   - Order of magnitude faster for async/concurrent operations
   - 50 GET requests: httpx ~1.22s, requests ~1.5s

4. **Modern Features**:
   - Type annotations (better IDE support)
   - Automatic JSON/format handling
   - Built-in mocking (works for sync and async)
   - Streaming responses
   - Connection pooling
   - Customizable timeout settings
   - Authentication, redirects, proxies

5. **Async Environment Support**: Works with asyncio and trio

**Behavioral Differences:**
- httpx doesn't auto-follow redirects (must opt-in)
- User-agent: "httpx" vs "Python-Requests"
- Better SSL cipher suite compatibility
- Requires Python 3.9+

**When to Use:**
- **requests**: Simple synchronous needs, maximum compatibility
- **httpx**: Modern apps, async support needed, HTTP/2 required, high concurrency

**Sources:**
- [HTTPX vs Requests (Oxylabs)](https://oxylabs.io/blog/httpx-vs-requests-vs-aiohttp)
- [Requests vs HTTPX (ScrapingAnt)](https://scrapingant.com/blog/requests-vs-httpx)
- [Best Python HTTP Clients 2025](https://proxyway.com/guides/the-best-python-http-clients)
- [Python HTTPX Documentation](https://www.python-httpx.org/)

### Node.js: axios vs got vs Fetch API

#### Fetch API (Native)
**Characteristics:**
- Built-in browser and Node.js (v18+)
- Lightweight, efficient, no dependencies
- Smaller bundle size
- Streaming response support
- Manual JSON parsing, error handling
- Native in most JS runtimes

#### axios
**Key Features:**
1. **Promise-based**: Easy async/await usage
2. **Interceptors**: Add custom logic to requests/responses
   ```javascript
   axios.interceptors.request.use(config => {
       config.headers.Authorization = `Bearer ${token}`;
       return config;
   });
   ```

3. **Automatic JSON Handling**: Serializes/parses automatically
4. **Data Transformation**: Transform request/response data
5. **Request Cancellation**: Built-in mechanisms
6. **Form Serialization**: Auto serialize to multipart/form-data or x-www-form-urlencoded
7. **XSRF Protection**: Client-side protection
8. **HTTP/2 Support**: Experimental (v1.13.0+)
9. **Fetch Adapter**: Available (v1.7.0+)

**Use Cases:**
- Need interceptors for auth/logging
- Automatic transforms
- Broader browser compatibility
- Established codebases

#### got
**Characteristics:**
- Lightweight, feature-rich for Node.js
- Powerful error handling
- High performance
- Promise-based API
- Retry mechanisms, hooks, custom agents
- Maintains strong position in Node.js ecosystem

#### 2025 Trends
- Increased Edge Computing adoption favors native Fetch
- Growing streaming response support
- Enhanced TypeScript integration
- Better framework integration

**Sources:**
- [Axios vs Fetch 2025](https://blog.logrocket.com/axios-vs-fetch-2025/)
- [Exploring HTTP Clients](https://blog.platformatic.dev/exploring-http-clients-axios-requests-and-node-fetch)
- [axios GitHub](https://github.com/axios/axios)
- [Fetch API vs Axios vs Alova](https://www.freecodecamp.org/news/fetch-api-vs-axios-vs-alova/)

### Summary: Common Features Across Modern HTTP Clients

| Feature | Go Resty | Rust reqwest | Python httpx | Node.js axios |
|---------|----------|--------------|--------------|---------------|
| Connection Pooling | ✅ | ✅ Auto | ✅ | ✅ |
| Retry Logic | ✅ Built-in | ⚠️ Manual | ⚠️ Manual | ⚠️ Manual/Library |
| Interceptors/Middleware | ✅ | ⚠️ Manual | ⚠️ Manual | ✅ Built-in |
| Async/Await | ✅ | ✅ | ✅ Both | ✅ |
| Auto JSON | ✅ | ✅ | ✅ | ✅ |
| HTTP/2 | ✅ | ✅ | ✅ | ⚠️ Experimental |
| Timeout Config | ✅ | ✅ Granular | ✅ | ✅ |
| Redirect Control | ✅ | ✅ | ⚠️ Manual | ✅ |
| Cookie Management | ✅ | ✅ Optional | ✅ | ⚠️ Manual |
| Type Safety | ✅ Structs | ✅ Structs | ⚠️ Annotations | ⚠️ TypeScript |

---

## Step 3: Eiffel Ecosystem Analysis

### ISE http_client Library

**Location:** `$ISE_LIBRARY/contrib/library/network/http_client/http_client.ecf`

**Description:** Simple web client to send HTTP requests (GET, POST, etc.) and receive responses.

**Two Implementations:**
1. **libcurl-based**: Uses Eiffel cURL wrapper (C library dependency)
2. **EiffelNET-based**: Pure Eiffel using EiffelNET and EiffelNET SSL extension

**Recent Improvements (EiffelStudio 18.01):**
- Fixed query and form data encoding (correct handling of '+', ' ')
- Fixed file uploading (various issues in libcurl and net implementation)
- Fixed form multipart encoding (correct boundary usage)
- Added support for multiple files in form data

**Key Classes:**
- `NET_HTTP_CLIENT`: Main client class
- `NET_HTTP_CLIENT_SESSION`: Session management
- `HTTP_CLIENT_RESPONSE`: Response wrapper
- Various protocol and header handling classes

### EiffelNet (Standard Library)

**Location:** `$ISE_LIBRARY/library/net/net.ecf`

**Capabilities:**
- Socket manipulation (TCP/UDP)
- Network addresses (IPv4, IPv6)
- Basic protocol implementations
- `HTTP_PROTOCOL`: Highest-level HTTP class in EiffelNet

**Limitations:**
- Basic, low-level networking
- Limited high-level HTTP features
- No built-in retry, pooling, or resilience patterns

### Eiffel-Loop HTTP Client Library

**Features:**
- HTTP download to file
- GET, POST, HEAD methods
- Higher-level abstractions than EiffelNet
- Not part of standard ISE distribution

### Current simple_http Implementation Analysis

**Architecture:**
- Wraps `NET_HTTP_CLIENT` from ISE http_client library
- Delegates to `NET_HTTP_CLIENT_SESSION` for requests
- Wraps responses in `SIMPLE_HTTP_RESPONSE`

**What It Has:**
```eiffel
class SIMPLE_HTTP
feature
    -- Configuration
    timeout: INTEGER
    connect_timeout: INTEGER
    set_timeout (a_seconds: INTEGER)
    set_connect_timeout (a_seconds: INTEGER)

    -- HTTP Methods
    get (a_url: STRING): SIMPLE_HTTP_RESPONSE
    post (a_url: STRING; a_data: detachable STRING): SIMPLE_HTTP_RESPONSE
    put (a_url: STRING; a_data: detachable STRING): SIMPLE_HTTP_RESPONSE
    delete (a_url: STRING): SIMPLE_HTTP_RESPONSE
    head (a_url: STRING): SIMPLE_HTTP_RESPONSE
    patch (a_url: STRING; a_data: detachable STRING): SIMPLE_HTTP_RESPONSE

    -- Convenience
    last_response: detachable SIMPLE_HTTP_RESPONSE
    is_success: BOOLEAN
    has_error: BOOLEAN
```

```eiffel
class SIMPLE_HTTP_RESPONSE
feature
    -- Response Data
    status: INTEGER
    body: detachable READABLE_STRING_8
    raw_header: READABLE_STRING_8
    headers: HASH_TABLE [STRING, STRING]
    error_message: detachable READABLE_STRING_8
    error_occurred: BOOLEAN

    -- Convenience
    header (a_name: STRING): detachable STRING
    content_type: detachable STRING
    content_length: INTEGER
    body_string: STRING

    -- Status Helpers
    is_success: BOOLEAN          -- 2xx
    is_redirect: BOOLEAN         -- 3xx
    is_client_error: BOOLEAN     -- 4xx
    is_server_error: BOOLEAN     -- 5xx
    has_error: BOOLEAN
```

**Design Strengths:**
1. Clean, simple API surface
2. Proper Eiffel contracts (require/ensure)
3. Good response status helpers
4. Header parsing with case-insensitive access
5. Separation of concerns (client vs response)
6. Wraps complexity of underlying ISE library

**Design Weaknesses:**
1. Creates new session per request (no session reuse/pooling)
2. No header customization (User-Agent, Accept, Authorization)
3. No query parameter helpers (manual URL construction)
4. No request body helpers (JSON, form data)
5. No configuration persistence (timeout applied per request)
6. No redirect control (uses ISE defaults)
7. No cookie management
8. No authentication helpers
9. No resilience patterns (retry, circuit breaker)

**Sources:**
- [Eiffel Networking Solutions](https://www.eiffel.org/doc/solutions/Networking)
- [Eiffel-Loop HTTP Client](http://www.eiffel-loop.com/library/http-client.html)
- [EiffelStudio 18.01 Release Notes](https://dev.eiffel.com/EiffelStudio_18.01_Releases)

---

## Step 4: Developer Pain Points

### Common HTTP Client Frustrations

#### 1. Error Handling Complexity
**Pain Points:**
- Distinguishing transient vs permanent errors
- Network timeouts vs server errors
- SSL/TLS handshake failures
- Connection reset errors
- Proper error messages for debugging

**Common Issues:**
- Connection timeout errors (host/service/intermediary failed to establish connection)
- Read timeout on idle SSL connections (connection appears stale, breaks keep-alive)
- SSL handshake timeout (failure to complete TLS handshake in time)
- Connection reset (host closed connection before request complete)
- Expired/self-signed certificates
- Mismatched SSL/TLS versions

**Developer Expectations:**
- Clear error types (network, timeout, HTTP, auth)
- Actionable error messages
- Automatic retry for transient errors
- Context preservation (which request failed)

#### 2. Timeout Configuration
**Challenges:**
- Multiple timeout types (connection, read, write, total)
- Appropriate defaults (AWS SDK: 2s in-region, 3s cross-region too low)
- Different needs for different endpoints
- Streaming vs single-shot requests

**Common Mistakes:**
- Setting timeout too low (excessive failures)
- No timeout (hangs indefinitely)
- Same timeout for all requests (one size doesn't fit all)
- Forgetting idle timeout for streaming

**Developer Needs:**
- Sensible defaults (30s request, 10s connect)
- Easy override per request
- Separate connect/read/total timeouts
- Documentation of recommendations

#### 3. SSL/TLS Issues
**Frequent Problems:**
- Certificate verification failures
- Mismatched protocol versions (client TLS 1.3, server TLS 1.0)
- Cipher suite incompatibility
- Expired certificates
- Self-signed certificates (development)
- SNI (Server Name Indication) misconfiguration
- Incomplete certificate chains

**Prevention Strategies:**
- Support latest TLS (1.2+, preferably 1.3)
- Strong cipher suite defaults
- Clear certificate verification options
- Easy disable for development (with warnings)
- Helpful error messages (which cert failed, why)

#### 4. Redirect Handling
**Complications:**
- Max redirect limits (prevent infinite loops)
- Cross-domain redirects (security implications)
- Preserving HTTP method (POST → GET on 302?)
- Authorization header stripping (security)
- Cookie handling across redirects
- Relative vs absolute redirect URLs

**Common Bugs:**
- Authorization lost on redirect (cleared by default in .NET)
- Cookies not sent to redirect target (domain mismatch)
- POST becoming GET (historical 302 behavior)
- Infinite redirect loops
- Mixed HTTP/HTTPS redirects

**Developer Expectations:**
- Configurable max redirects (default 10)
- Follow/don't follow option
- Preserve method option
- Access redirect chain
- Clear redirect policies

#### 5. Cookie Management
**Challenges:**
- Cookie persistence across requests
- Domain/path matching rules
- Secure/HttpOnly flags
- Expiration handling
- Cookie jars (manual vs automatic)
- SameSite attribute (CSRF protection)

**Common Issues:**
- Cookies lost on redirects (domain/path mismatch)
- Manual cookie headers not cleared on redirect
- Race conditions (redirect before cookie stored)
- Multiple cookie standards (Netscape, RFC 2109, RFC 2965)
- httpOnly cookies can't be read by JS (but sent by browser)

**Developer Needs:**
- Automatic cookie jar
- Manual cookie override
- Easy session management
- Clear cookie inspection
- Policy-driven cookie handling

#### 6. Authentication Patterns
**Common Schemes:**
- Bearer tokens (OAuth 2.0, JWT)
- Basic authentication (username:password)
- API keys (header or query param)
- OAuth 2.0 flows
- Digest authentication
- Client certificates (mTLS)

**Pain Points:**
- Token refresh logic
- Authorization header management
- Storing credentials securely
- Multiple auth schemes per app
- Auth persistence across requests
- Credential injection (not hardcoded)

**Developer Expectations:**
- Easy Bearer token setup
- Basic auth helper
- API key injection
- OAuth 2.0 helpers (separate library okay)
- Clear auth documentation

#### 7. Request/Response Body Handling
**Content Types:**
- JSON (most common)
- Form-encoded (application/x-www-form-urlencoded)
- Multipart form data (file uploads)
- XML
- Plain text
- Binary data

**Pain Points:**
- Manual JSON serialization
- Form encoding edge cases (spaces, +, special chars)
- Multipart boundaries (complex to construct)
- Large file uploads (memory issues)
- Streaming responses (large downloads)
- Content-Type header mismatch

**Developer Needs:**
- Auto JSON serialize/deserialize
- Form builder helper
- Multipart upload helper
- Streaming support
- Automatic Content-Type setting

#### 8. Connection Pooling
**Why It Matters:**
- TCP handshake expensive (3-way)
- TLS handshake very expensive (multiple round-trips)
- Reusing connections improves performance 10x+
- Prevents socket exhaustion

**Common Mistakes:**
- Creating new client per request (no pooling)
- Not reusing client instance
- Pool size too small (contention)
- Pool size too large (resource waste)
- No idle timeout (stale connections)
- No max connections per host (unbalanced)

**Best Practices:**
- Create ONE client instance, reuse it
- Automatic pooling (like reqwest, Resty)
- Configurable pool size
- Idle timeout (5 minutes default)
- Per-host connection limits
- Connection health checks

#### 9. Retry Logic
**When to Retry:**
- Network errors (connection reset, timeout)
- 429 Too Many Requests (with Retry-After)
- 503 Service Unavailable (transient)
- 502/504 Gateway errors (intermediary issues)

**When NOT to Retry:**
- 400 Bad Request (client error)
- 401 Unauthorized (auth error)
- 403 Forbidden (permission error)
- 404 Not Found (resource doesn't exist)
- 4xx client errors (generally)

**Retry Strategies:**
- Fixed delay (simple but inefficient)
- Linear backoff (predictable)
- Exponential backoff (recommended)
- Exponential backoff with jitter (best - prevents thundering herd)

**Configuration Needs:**
- Max retry attempts (default 3)
- Initial delay (default 100ms)
- Max delay (default 30s)
- Retry predicate (which errors/status codes)
- Backoff strategy

#### 10. Interceptors/Middleware
**Use Cases:**
- Logging (request/response)
- Authentication (inject tokens)
- Metrics (latency, status codes)
- Retry logic
- Caching
- Request/response transformation
- Error handling

**Developer Expectations:**
- Pre-request hooks
- Post-response hooks
- Error hooks
- Chainable/composable
- Easy to add/remove
- Access to full request/response

**Sources:**
- [SSL/TLS Handshake Errors (Sectigo)](https://www.sectigo.com/blog/tls-ssl-handshake-errors-how-to-fix-them)
- [Troubleshoot HTTP Status Codes & SSL Errors](https://squareops.com/blog/comprehensive-guide-to-http-errors-in-devops-causes-scenarios-and-troubleshooting-steps/)
- [AWS SDK Troubleshooting FAQs](https://docs.aws.amazon.com/sdk-for-java/latest/developer-guide/troubleshooting.html)
- [HttpClient Cookie Guide](https://hc.apache.org/httpclient-legacy/cookies.html)
- [Cookie Authentication Issues (.NET)](https://github.com/dotnet/runtime/issues/16983)

---

## Step 5: Innovation Opportunities

### What Would Make simple_http Exceptional?

Based on research of modern HTTP clients and developer pain points, here are innovation opportunities for the Eiffel ecosystem:

#### 1. Fluent Request Builder API
**Problem:** Current API requires multiple calls, no request customization
**Solution:** Chainable request builder

```eiffel
-- Current (limited)
response := http.post (url, data)

-- Enhanced (fluent)
response := http.request
    .url ("https://api.example.com/users")
    .method ("POST")
    .header ("Authorization", "Bearer " + token)
    .header ("Accept", "application/json")
    .json_body (user_object)
    .timeout (60)
    .execute
```

**Benefits:**
- Discoverable API (IDE autocomplete)
- Type-safe configuration
- Optional parameters clear
- Readable, self-documenting code

#### 2. Smart Response Type Inference
**Problem:** Manual parsing of JSON/XML responses
**Solution:** Content-type aware parsing helpers

```eiffel
class SIMPLE_HTTP_RESPONSE
feature
    json: detachable JSON_OBJECT
        -- Parsed JSON if Content-Type is application/json

    json_array: detachable JSON_ARRAY
        -- Parsed JSON array

    xml: detachable XML_DOCUMENT
        -- Parsed XML if Content-Type is application/xml or text/xml

    as_json_object: JSON_OBJECT
        -- Parse body as JSON object (regardless of Content-Type)
        require body_not_empty: attached body
```

**Benefits:**
- Automatic parsing based on Content-Type
- Lazy evaluation (parse on first access)
- Optional manual override
- Clear error if parse fails

#### 3. Built-in Retry with Exponential Backoff
**Problem:** No resilience for transient errors
**Solution:** Configurable retry policy

```eiffel
http.set_retry_policy (policy)
    -- policy: SIMPLE_HTTP_RETRY_POLICY

create policy.make_default  -- 3 retries, exponential backoff, jitter
create policy.make_custom (5, 100, 30000, True)  -- max_retries, initial_ms, max_ms, jitter

-- Or per-request
response := http.request
    .url (url)
    .retry (3)  -- Simple: just max retries
    .retry_on_status (<<503, 429>>)  -- Which status codes to retry
    .execute
```

**Smart Defaults:**
- Retry on network errors (connection timeout, reset)
- Retry on 429 (with Retry-After header respect)
- Retry on 503 Service Unavailable
- Don't retry 4xx client errors
- Don't retry non-idempotent methods (POST) by default
- Exponential backoff with jitter
- Max 3 retries default

**Benefits:**
- Resilience out-of-box
- Prevents thundering herd (jitter)
- Respects server hints (Retry-After)
- Configurable but sensible defaults

#### 4. Connection Pooling and Reuse
**Problem:** New session per request (expensive)
**Solution:** Automatic connection pooling

```eiffel
class SIMPLE_HTTP
feature {NONE}
    client: NET_HTTP_CLIENT
        -- Reused client instance

    sessions: HASH_TABLE [NET_HTTP_CLIENT_SESSION, STRING]
        -- Session pool keyed by host

    get_or_create_session (a_url: STRING): NET_HTTP_CLIENT_SESSION
        -- Reuse session for same host, or create new
```

**Configuration:**
```eiffel
http.set_pool_config (config)
    config.max_connections_per_host := 10
    config.max_idle_connections := 50
    config.idle_timeout := 300  -- seconds
    config.connection_lifetime := 600  -- max age
```

**Benefits:**
- 10x+ performance improvement (reuse TCP/TLS)
- Automatic management (no manual pooling)
- Configurable limits (prevent resource exhaustion)
- Stale connection detection

#### 5. Authentication Helpers
**Problem:** Manual header construction, token management
**Solution:** Built-in auth methods

```eiffel
-- Bearer token (OAuth 2.0, JWT)
http.set_bearer_token (token)
http.set_auth_header ("Bearer " + token)  -- Equivalent

-- Basic auth
http.set_basic_auth (username, password)
    -- Automatically encodes to "Basic base64(username:password)"

-- API key (header)
http.set_api_key ("X-API-Key", key)

-- API key (query param)
http.set_api_key_query ("api_key", key)

-- Per-request override
response := http.request
    .url (url)
    .bearer_token (different_token)
    .execute
```

**Benefits:**
- Reduces boilerplate
- Prevents encoding errors
- Clear, self-documenting
- Secure defaults

#### 6. Query Parameter Builder
**Problem:** Manual URL construction, encoding issues
**Solution:** Type-safe query builder

```eiffel
response := http.request
    .url ("https://api.example.com/users")
    .query ("status", "active")
    .query ("role", "admin")
    .query ("limit", "20")
    .query ("created_after", "2025-01-01")
    .execute
    -- GET https://api.example.com/users?status=active&role=admin&limit=20&created_after=2025-01-01

-- Or batch
params: HASH_TABLE [STRING, STRING]
params.put ("active", "status")
params.put ("admin", "role")
response := http.request.url (base_url).queries (params).execute
```

**Benefits:**
- Automatic URL encoding
- Type-safe (integer, boolean converters)
- Array parameter support
- No manual concatenation

#### 7. Request/Response Interceptors
**Problem:** No hooks for cross-cutting concerns
**Solution:** Interceptor pattern

```eiffel
class SIMPLE_HTTP_INTERCEPTOR
    request_interceptor (req: SIMPLE_HTTP_REQUEST)
        -- Called before request sent
        -- Can modify request (add headers, log, etc.)

    response_interceptor (req: SIMPLE_HTTP_REQUEST; resp: SIMPLE_HTTP_RESPONSE)
        -- Called after response received
        -- Can modify response (log, cache, etc.)

    error_interceptor (req: SIMPLE_HTTP_REQUEST; error: EXCEPTION)
        -- Called on error
        -- Can retry, log, transform error
end

-- Usage
create logger_interceptor
http.add_interceptor (logger_interceptor)

-- Built-in interceptors
http.add_interceptor (create {SIMPLE_HTTP_LOGGER_INTERCEPTOR})
http.add_interceptor (create {SIMPLE_HTTP_METRICS_INTERCEPTOR})
http.add_interceptor (create {SIMPLE_HTTP_RETRY_INTERCEPTOR}.make (3))
```

**Benefits:**
- Separation of concerns
- Reusable cross-cutting logic
- Easy logging/metrics
- Testable (mock interceptors)

#### 8. JSON Body Helpers
**Problem:** Manual JSON serialization
**Solution:** Automatic JSON handling (requires simple_json integration)

```eiffel
-- Send JSON
user_data: JSON_OBJECT
user_data.put ("John Doe", "name")
user_data.put ("john@example.com", "email")

response := http.request
    .url (url)
    .method ("POST")
    .json_body (user_data)  -- Automatically sets Content-Type: application/json
    .execute

-- Receive JSON
if attached response.json as json_obj then
    if attached {JSON_STRING} json_obj.item ("name") as name then
        print (name.item)
    end
end

-- Or more convenient
if attached response.json_value ("name") as name then
    print (name)
end
```

**Benefits:**
- No manual serialization
- Automatic Content-Type
- Type-safe access
- Integration with simple_json

#### 9. Form Data Helpers
**Problem:** Complex form encoding, file uploads
**Solution:** Form builder

```eiffel
-- URL-encoded form
response := http.request
    .url (url)
    .form_field ("username", "john")
    .form_field ("password", "secret")
    .execute
    -- Content-Type: application/x-www-form-urlencoded
    -- Body: username=john&password=secret

-- Multipart form (file upload)
response := http.request
    .url (url)
    .multipart_field ("title", "My Document")
    .multipart_file ("document", "/path/to/file.pdf", "application/pdf")
    .execute
    -- Content-Type: multipart/form-data; boundary=...
```

**Benefits:**
- Correct encoding
- File upload simplified
- Automatic boundary generation
- Memory-efficient streaming

#### 10. Redirect Control
**Problem:** No control over redirect behavior
**Solution:** Configurable redirect policy

```eiffel
-- Global setting
http.set_max_redirects (10)  -- Default
http.set_max_redirects (0)   -- Don't follow redirects
http.set_follow_redirects (False)

-- Per-request
response := http.request
    .url (url)
    .follow_redirects (False)
    .execute

-- Access redirect chain
if response.was_redirected then
    across response.redirect_chain as url_cursor loop
        print (url_cursor.item + "%N")
    end
    print ("Final URL: " + response.final_url)
end
```

**Benefits:**
- Control redirect behavior
- Inspect redirect chain
- Prevent infinite loops
- Security (prevent open redirects)

#### 11. Timeout Granularity
**Problem:** Only single timeout value
**Solution:** Multiple timeout types

```eiffel
http.set_connect_timeout (10)  -- Time to establish connection
http.set_read_timeout (30)     -- Time to read response
http.set_write_timeout (30)    -- Time to write request
http.set_total_timeout (60)    -- Total request time (overrides others)

-- Per-request
response := http.request
    .url (url)
    .connect_timeout (5)
    .read_timeout (120)  -- Long-running API
    .execute
```

**Benefits:**
- Fine-grained control
- Different needs for different endpoints
- Streaming support (long read timeout)
- Clear timeout failures (which timeout?)

#### 12. Cookie Jar
**Problem:** No automatic cookie management
**Solution:** Optional cookie jar

```eiffel
-- Enable automatic cookie handling
http.enable_cookies

-- Cookies automatically stored and sent on subsequent requests
response := http.get ("https://example.com/login")  -- Receives Set-Cookie
response := http.get ("https://example.com/profile") -- Sends Cookie

-- Manual cookie access
cookies := http.cookies_for_url ("https://example.com")

-- Clear cookies
http.clear_cookies
http.clear_cookies_for_domain ("example.com")

-- Disable cookies
http.disable_cookies
```

**Benefits:**
- Session management
- Automatic (like browser)
- Manual override available
- Security (domain isolation)

---

## Step 6: Design Strategy

### Core Design Principles

#### 1. Simple by Default, Powerful When Needed
**Philosophy:** The 80% case should be one-liner; 20% case should be possible.

**Simple (80% case):**
```eiffel
create http.make
response := http.get ("https://api.example.com/users")
if response.is_success then
    print (response.body_string)
end
```

**Powerful (20% case):**
```eiffel
response := http.request
    .url ("https://api.example.com/users")
    .header ("Authorization", "Bearer " + token)
    .query ("status", "active")
    .timeout (60)
    .retry (3)
    .execute
```

#### 2. Progressive Disclosure
**Layers of Complexity:**
1. **Minimal:** Basic GET/POST with defaults
2. **Intermediate:** Headers, query params, timeouts
3. **Advanced:** Retry, interceptors, pooling config
4. **Expert:** Custom session management, connection control

**Benefit:** Users learn incrementally, not overwhelmed.

#### 3. Fail-Fast with Clear Errors
**Principles:**
- Contracts at API boundaries (require/ensure)
- Clear exception types (SIMPLE_HTTP_ERROR hierarchy)
- Actionable error messages
- Include context (URL, method, status if available)

**Error Types:**
```eiffel
SIMPLE_HTTP_ERROR (abstract)
    SIMPLE_HTTP_NETWORK_ERROR (connection failed, timeout)
    SIMPLE_HTTP_SSL_ERROR (certificate, handshake)
    SIMPLE_HTTP_REDIRECT_ERROR (too many redirects, loop)
    SIMPLE_HTTP_PARSE_ERROR (invalid JSON, XML)
    SIMPLE_HTTP_STATUS_ERROR (4xx, 5xx if configured to throw)
```

#### 4. Eiffel-Idiomatic Design
**Eiffel Strengths:**
- Design by Contract (require/ensure)
- Query/Command separation
- Strong typing
- Once functions (lazy initialization)
- Agents (callbacks)

**Design Patterns:**
- Builder pattern (request builder)
- Strategy pattern (retry policy)
- Chain of responsibility (interceptors)
- Factory pattern (session creation)
- Singleton (client instance reuse)

### What simple_http SHOULD Be

1. **Thin Wrapper:** Abstracts ISE http_client complexity, not reimplementing HTTP
2. **Developer-Friendly:** Intuitive API, minimal boilerplate
3. **REST-Oriented:** Optimized for REST API consumption
4. **Resilient:** Built-in retry, timeout handling
5. **Modern:** JSON support, auth helpers, connection pooling
6. **Testable:** Mockable, interceptors for testing
7. **Documented:** Rich examples, clear contracts
8. **Extensible:** Interceptors, custom policies

### What simple_http Should NOT Be

1. **HTTP Server:** Only client-side (simple_web handles server)
2. **Complete Replacement:** Still uses ISE http_client underneath
3. **WebSocket Client:** Different protocol (could be simple_websocket)
4. **GraphQL Client:** Different query paradigm (separate library)
5. **Browser:** No DOM, JavaScript, rendering
6. **Low-Level Socket:** Use EiffelNet for that
7. **Async Runtime:** Eiffel not async-native (blocking okay)
8. **OAuth 2.0 Flow Handler:** Complex flows (separate library simple_oauth)
9. **HTTP/2 or HTTP/3 Pusher:** Depends on ISE library capabilities

### API Surface Design

#### Minimal API (Most Users)
```eiffel
class SIMPLE_HTTP
create
    make

feature -- Basic Operations
    get (url: STRING): SIMPLE_HTTP_RESPONSE
    post (url: STRING; body: detachable STRING): SIMPLE_HTTP_RESPONSE
    put (url: STRING; body: detachable STRING): SIMPLE_HTTP_RESPONSE
    delete (url: STRING): SIMPLE_HTTP_RESPONSE
    head (url: STRING): SIMPLE_HTTP_RESPONSE
    patch (url: STRING; body: detachable STRING): SIMPLE_HTTP_RESPONSE

feature -- Configuration
    set_timeout (seconds: INTEGER)
    set_connect_timeout (seconds: INTEGER)
    set_bearer_token (token: STRING)
    set_basic_auth (username, password: STRING)

feature -- State
    last_response: detachable SIMPLE_HTTP_RESPONSE
    is_success: BOOLEAN
    has_error: BOOLEAN
end
```

#### Intermediate API (Power Users)
```eiffel
feature -- Request Builder
    request: SIMPLE_HTTP_REQUEST_BUILDER
        -- Fluent request builder

feature -- Advanced Configuration
    set_max_redirects (count: INTEGER)
    set_retry_policy (policy: SIMPLE_HTTP_RETRY_POLICY)
    add_interceptor (interceptor: SIMPLE_HTTP_INTERCEPTOR)
    enable_cookies
    set_default_headers (headers: HASH_TABLE [STRING, STRING])
```

#### Advanced API (Library Authors)
```eiffel
feature -- Connection Management
    set_pool_config (config: SIMPLE_HTTP_POOL_CONFIG)
    close_idle_connections

feature -- Low-Level Access
    underlying_client: NET_HTTP_CLIENT
        -- Access to ISE client for advanced needs
```

### Versioning Strategy

**Semantic Versioning:** MAJOR.MINOR.PATCH
- **MAJOR:** Breaking API changes
- **MINOR:** New features, backward-compatible
- **PATCH:** Bug fixes

**Initial Releases:**
- **1.0.0:** Core features (current + essential improvements)
- **1.1.0:** Retry logic
- **1.2.0:** Request builder
- **1.3.0:** JSON helpers
- **2.0.0:** Breaking changes if needed (fluent API redesign?)

**Deprecation Policy:**
- Mark deprecated features with `obsolete` note
- Keep deprecated features for one minor version
- Remove in next major version

### Testing Strategy

**Test Categories:**
1. **Unit Tests:** Individual features (timeout, headers, parsing)
2. **Integration Tests:** Real HTTP requests (httpbin.org, JSONPlaceholder)
3. **Mock Tests:** Interceptors, retry logic without network
4. **Error Tests:** Timeout, connection failure, SSL errors
5. **Performance Tests:** Connection reuse, pooling efficiency

**Test Dependencies:**
- simple_testing (Eiffel test framework)
- httpbin.org (HTTP testing service)
- Local mock server (optional)

---

## Step 7: Gap Analysis - Current vs Desired

### Current Implementation (simple_http v1.0.0)

#### What It Has ✅
| Feature | Implementation | Quality |
|---------|---------------|---------|
| HTTP Methods | GET, POST, PUT, DELETE, HEAD, PATCH | ✅ Complete |
| Basic Timeout | `timeout`, `connect_timeout` | ✅ Adequate |
| Response Status | `status`, `body`, `headers` | ✅ Good |
| Status Helpers | `is_success`, `is_redirect`, `is_client_error`, `is_server_error` | ✅ Excellent |
| Header Parsing | Case-insensitive access | ✅ Good |
| Error Detection | `has_error`, `error_message` | ✅ Basic |
| Clean API | Simple method signatures | ✅ Good |
| Eiffel Contracts | require/ensure clauses | ✅ Good |

#### What It Lacks ❌

| Feature | Priority | Effort | Impact |
|---------|----------|--------|--------|
| Request Builder API | HIGH | Medium | High |
| Header Customization | HIGH | Low | High |
| Query Parameter Helper | HIGH | Low | High |
| JSON Body Helpers | HIGH | Medium | High |
| Bearer/Basic Auth | HIGH | Low | High |
| Connection Pooling | HIGH | High | Very High |
| Retry Logic | MEDIUM | Medium | High |
| Redirect Control | MEDIUM | Low | Medium |
| Cookie Management | MEDIUM | Medium | Medium |
| Interceptors | MEDIUM | High | Medium |
| Form Data Helpers | LOW | Medium | Medium |
| Multiple Timeout Types | LOW | Low | Low |
| Response Type Inference | LOW | Medium | Medium |
| Error Type Hierarchy | LOW | Medium | Medium |

### Detailed Feature Gaps

#### 1. Request Customization ❌ CRITICAL
**Current:**
- No way to add custom headers (Authorization, Accept, User-Agent)
- No query parameter helpers (manual URL construction)
- No request body helpers (JSON, form data)

**Needed:**
```eiffel
response := http.request
    .url (url)
    .header ("Authorization", "Bearer " + token)
    .header ("Accept", "application/json")
    .query ("limit", "20")
    .json_body (data)
    .execute
```

**Impact:** Blocks most real-world API usage (auth, pagination, filtering)

#### 2. Connection Pooling ❌ CRITICAL
**Current:**
- Creates new `NET_HTTP_CLIENT_SESSION` per request
- No connection reuse
- TCP/TLS handshake every request (10x+ slower)

**Needed:**
- Reuse sessions for same host
- Automatic pooling
- Configurable pool size, idle timeout

**Impact:** Performance 10x worse than it should be

#### 3. Authentication Helpers ❌ HIGH
**Current:**
- Manual header construction
- No Bearer token helper
- No Basic auth helper

**Needed:**
```eiffel
http.set_bearer_token (token)
http.set_basic_auth (username, password)
```

**Impact:** Boilerplate in every auth scenario

#### 4. JSON Integration ❌ HIGH
**Current:**
- No JSON parsing helpers
- No JSON body serialization
- Manual Content-Type management

**Needed:**
- `response.json` (parsed JSON_OBJECT)
- `.json_body(obj)` (auto serialize)
- Integration with simple_json

**Impact:** Tedious for REST API consumption (most common use case)

#### 5. Retry Logic ❌ MEDIUM
**Current:**
- No retry on transient errors
- Manual retry implementation required

**Needed:**
- Configurable retry policy
- Exponential backoff with jitter
- Smart defaults (retry 503, 429, network errors)

**Impact:** Poor resilience, thundering herd risk

#### 6. Redirect Control ❌ MEDIUM
**Current:**
- Uses ISE defaults (unknown behavior)
- No access to redirect chain
- Can't disable redirects

**Needed:**
- `set_max_redirects(n)`
- `follow_redirects(boolean)`
- `response.redirect_chain`

**Impact:** Limited control, security concerns

#### 7. Cookie Management ❌ MEDIUM
**Current:**
- No automatic cookie handling
- Manual cookie headers required

**Needed:**
- Automatic cookie jar
- `enable_cookies`
- Session management

**Impact:** Session-based auth tedious

#### 8. Query Parameters ❌ HIGH
**Current:**
- Manual URL construction
- Error-prone encoding

**Needed:**
```eiffel
.query("status", "active")
.query("limit", "20")
```

**Impact:** Every API call needs pagination, filtering

#### 9. Error Handling ❌ MEDIUM
**Current:**
- Generic `error_occurred` boolean
- String `error_message`
- No error types

**Needed:**
- Error hierarchy (network, SSL, timeout, HTTP)
- Actionable error messages
- Error context

**Impact:** Debugging difficult, can't handle errors specifically

#### 10. Interceptors ❌ LOW
**Current:**
- No hooks for logging, metrics, modification

**Needed:**
- Request/response interceptors
- Error interceptors
- Chainable

**Impact:** Cross-cutting concerns (logging, metrics) require wrapping

### Migration Path for Current Users

**Backward Compatibility:**
- Keep existing `get(url)`, `post(url, data)` methods
- Add new features without breaking old code
- Deprecate gracefully if needed

**Example: Current Code Still Works**
```eiffel
-- Version 1.0.0 code (current)
create http.make
response := http.get ("https://api.example.com/users")

-- Still works in 2.0.0, plus new features available
response := http.request.url("https://api.example.com/users").header(...).execute
```

### Recommended Implementation Phases

#### Phase 1: Essential (v1.1.0) - 2-3 weeks
**Must Have for Basic Usability:**
1. Header customization (add_header, set_headers)
2. Query parameter builder
3. Bearer token / Basic auth helpers
4. Connection reuse (basic pooling)
5. JSON body helpers (if simple_json available)

**Deliverable:** Usable for basic REST API consumption with auth

#### Phase 2: Resilience (v1.2.0) - 2 weeks
**Must Have for Production:**
1. Retry policy with exponential backoff
2. Timeout improvements (separate connect/read)
3. Redirect control
4. Better error handling

**Deliverable:** Production-ready with resilience

#### Phase 3: Advanced (v1.3.0) - 3 weeks
**Nice to Have:**
1. Request builder API (fluent interface)
2. Cookie jar
3. Form data helpers
4. Interceptors

**Deliverable:** Feature parity with modern HTTP clients

#### Phase 4: Polish (v2.0.0) - 2 weeks
**Breaking Changes (if needed):**
1. Full fluent API
2. Response type inference
3. Error type hierarchy
4. Advanced pooling config

**Deliverable:** Best-in-class HTTP client for Eiffel

### Success Criteria

**Must Achieve:**
- [ ] Authenticate to REST APIs (Bearer, Basic)
- [ ] Send/receive JSON
- [ ] Add custom headers
- [ ] Query parameters
- [ ] Connection pooling (10x+ performance)
- [ ] Retry transient errors
- [ ] Handle timeouts gracefully
- [ ] Follow redirects with control
- [ ] Parse common response types

**Should Achieve:**
- [ ] Cookie-based sessions
- [ ] Form data / file uploads
- [ ] Interceptors (logging, metrics)
- [ ] Multiple timeout types
- [ ] Error type hierarchy

**Nice to Have:**
- [ ] HTTP/2 support (if ISE supports)
- [ ] Streaming responses
- [ ] Request/response compression
- [ ] Circuit breaker pattern

---

## Recommendations

### Immediate Actions (v1.1.0 - Essential)

#### 1. Add Header Customization
**Priority:** CRITICAL
**Effort:** LOW (1 day)

```eiffel
feature -- Headers
    add_header (name, value: STRING)
        -- Add custom header

    set_headers (headers: HASH_TABLE [STRING, STRING])
        -- Set multiple headers

    remove_header (name: STRING)
        -- Remove header
```

**Rationale:** Blocks all auth scenarios, critical for any real usage

#### 2. Add Query Parameter Builder
**Priority:** CRITICAL
**Effort:** LOW (1 day)

```eiffel
feature -- Query Parameters
    add_query (name, value: STRING)
        -- Add query parameter (auto URL-encodes)

    set_queries (params: HASH_TABLE [STRING, STRING])
        -- Set multiple query params
```

**Rationale:** Every API uses pagination, filtering - manual URL construction error-prone

#### 3. Add Authentication Helpers
**Priority:** CRITICAL
**Effort:** LOW (1 day)

```eiffel
feature -- Authentication
    set_bearer_token (token: STRING)
        -- Set Authorization: Bearer {token}

    set_basic_auth (username, password: STRING)
        -- Set Authorization: Basic {base64(username:password)}

    set_auth_header (value: STRING)
        -- Set Authorization header directly
```

**Rationale:** Most common use case, reduces boilerplate significantly

#### 4. Implement Connection Pooling
**Priority:** CRITICAL
**Effort:** MEDIUM (3-4 days)

**Changes:**
- Store sessions in HASH_TABLE keyed by host
- Reuse session for same host
- Configurable max connections, idle timeout
- Close idle connections after timeout

**Rationale:** 10x+ performance improvement, fundamental for production use

#### 5. Add JSON Body Helpers
**Priority:** HIGH
**Effort:** MEDIUM (2-3 days, depends on simple_json integration)

```eiffel
feature -- JSON
    set_json_body (json: JSON_VALUE)
        -- Set body to JSON string, Content-Type to application/json

    -- In SIMPLE_HTTP_RESPONSE
    json: detachable JSON_VALUE
        -- Parse body as JSON (lazy, based on Content-Type)
```

**Rationale:** REST APIs predominantly use JSON, manual serialize/deserialize tedious

### Short-Term Actions (v1.2.0 - Resilience)

#### 6. Implement Retry Logic
**Priority:** HIGH
**Effort:** MEDIUM (3-4 days)

**Features:**
- Retry policy class (max retries, backoff strategy)
- Exponential backoff with jitter
- Retry on 429, 503, network errors
- Honor Retry-After header
- Don't retry 4xx (except 429)

**Rationale:** Production systems need resilience, prevents cascading failures

#### 7. Add Redirect Control
**Priority:** MEDIUM
**Effort:** LOW (1-2 days)

```eiffel
feature -- Redirects
    set_max_redirects (count: INTEGER)
        -- 0 = don't follow, default 10

    -- In SIMPLE_HTTP_RESPONSE
    redirect_chain: LIST [STRING]
        -- URLs in redirect chain

    final_url: STRING
        -- Final URL after redirects
```

**Rationale:** Security (prevent open redirects), debugging (where did it go?)

#### 8. Improve Timeout Configuration
**Priority:** MEDIUM
**Effort:** LOW (1 day)

```eiffel
feature -- Timeouts
    set_connect_timeout (seconds: INTEGER)  -- Already exists
    set_read_timeout (seconds: INTEGER)      -- New
    set_total_timeout (seconds: INTEGER)     -- New (overrides others)
```

**Rationale:** Different needs (long downloads vs quick API), better diagnostics

### Medium-Term Actions (v1.3.0 - Advanced)

#### 9. Request Builder API
**Priority:** MEDIUM
**Effort:** HIGH (5-7 days)

**New Class:**
```eiffel
class SIMPLE_HTTP_REQUEST_BUILDER
feature
    url (a_url: STRING): like Current
    method (a_method: STRING): like Current
    header (name, value: STRING): like Current
    query (name, value: STRING): like Current
    body (content: STRING): like Current
    json_body (json: JSON_VALUE): like Current
    timeout (seconds: INTEGER): like Current
    retry (max: INTEGER): like Current
    execute: SIMPLE_HTTP_RESPONSE
end
```

**Rationale:** Fluent API more discoverable, type-safe, readable

#### 10. Cookie Jar
**Priority:** LOW
**Effort:** MEDIUM (3-4 days)

**Features:**
- Automatic cookie storage
- Domain/path matching
- Expiration handling
- Enable/disable
- Manual access

**Rationale:** Session-based auth (though JWT more common now)

### Long-Term Considerations

#### HTTP/2 Support
**Depends:** ISE http_client capabilities
**Action:** Research if `NET_HTTP_CLIENT` supports HTTP/2, enable if so

#### Async API
**Challenge:** Eiffel not async-native
**Action:** Investigate SCOOP (Simple Concurrent Object-Oriented Programming)
**Priority:** LOW (most HTTP clients fine with blocking)

#### Streaming Responses
**Use Case:** Large file downloads, chunked responses
**Priority:** LOW (niche use case)
**Effort:** MEDIUM-HIGH

---

## Conclusion

The current `simple_http` library provides a clean, well-designed foundation for HTTP client functionality in Eiffel. However, compared to modern HTTP clients in other ecosystems (Go Resty, Rust reqwest, Python httpx, Node.js axios), it lacks critical features that prevent it from being production-ready for contemporary REST API consumption.

### Critical Gaps (Blocking Production Use)
1. **No header customization** - Can't add Authorization, Accept, custom headers
2. **No connection pooling** - 10x+ slower than should be
3. **No authentication helpers** - Tedious boilerplate for every API
4. **No query parameter builder** - Error-prone manual URL construction
5. **No JSON integration** - Manual serialize/deserialize for most common use case

### Important Gaps (Limiting Resilience)
1. **No retry logic** - Vulnerable to transient errors, cascading failures
2. **No redirect control** - Security concerns, debugging difficulty
3. **No cookie management** - Session auth tedious
4. **Limited error handling** - Can't differentiate error types

### Nice-to-Have Gaps (Developer Experience)
1. **No fluent request builder** - Less discoverable, verbose
2. **No interceptors** - Cross-cutting concerns (logging, metrics) require wrapping
3. **No form data helpers** - File uploads complex
4. **No response type inference** - Manual parsing always

### Recommended Path Forward

**Phase 1 (v1.1.0 - Essential):** 2-3 weeks
- Header customization
- Query parameters
- Auth helpers
- Connection pooling
- JSON helpers

**Phase 2 (v1.2.0 - Resilience):** 2 weeks
- Retry logic
- Redirect control
- Better timeouts
- Error handling

**Phase 3 (v1.3.0 - Advanced):** 3 weeks
- Fluent request builder
- Cookie jar
- Form data
- Interceptors

**Total Estimated Effort:** 7-8 weeks to feature parity with modern HTTP clients

### Final Assessment

**Current State:** Functional but basic, suitable for simple GET requests only

**With Phase 1:** Production-ready for typical REST API consumption (auth, JSON, queries)

**With Phase 2:** Enterprise-ready with resilience patterns

**With Phase 3:** Best-in-class HTTP client, competitive with any language's offerings

The good news: The foundation is solid. The API design is clean and Eiffel-idiomatic. The architecture is sound. The gaps are primarily additive features, not fundamental redesigns. With focused effort over 2-3 months, `simple_http` can become the definitive HTTP client library for the Eiffel ecosystem.

---

## Sources

### Specifications
- [RFC 7230: HTTP/1.1 Message Syntax and Routing](https://datatracker.ietf.org/doc/html/rfc7230)
- [RFC 7231: HTTP/1.1 Semantics and Content](https://datatracker.ietf.org/doc/html/rfc7231)
- [RFC 7235: HTTP/1.1 Authentication](https://www.rfc-editor.org/rfc/rfc7235.html)
- [RFC 9110: HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html)
- [RFC 7540: HTTP/2](https://datatracker.ietf.org/doc/html/rfc7540)
- [RFC 9114: HTTP/3](https://datatracker.ietf.org/doc/html/rfc9114)
- [Updated HTTP specifications (Axway)](https://blog.axway.com/learning-center/software-development/updated-http-specifications)
- [HTTP RFCs Evolution (Cloudflare)](https://blog.cloudflare.com/cloudflare-view-http3-usage/)

### REST API Best Practices
- [Azure API Design Best Practices](https://learn.microsoft.com/en-us/azure/architecture/best-practices/api-design)
- [REST API Best Practices (Stack Overflow)](https://stackoverflow.blog/2020/03/02/best-practices-for-rest-api-design/)
- [RESTful API Design Guide (Strapi)](https://strapi.io/blog/restful-api-design-guide-principles-best-practices)
- [REST API Standards 2025 (Hevo)](https://hevodata.com/learn/rest-api-best-practices/)

### Go HTTP Clients
- [Resty GitHub](https://github.com/go-resty/resty)
- [HTTP Requests in Go: Comparison](https://medium.com/@praveendayanithi/http-requests-in-go-a-comparison-of-client-libraries-97ab5a8cc51c)
- [Best Golang HTTP Clients](https://hayageek.com/best-golang-http-clients/)
- [Simplifying HTTP Requests with Resty](https://leapcell.io/blog/simplifying-http-requests-in-go-with-resty)

### Rust HTTP Clients
- [reqwest GitHub](https://github.com/seanmonstar/reqwest)
- [reqwest documentation](https://docs.rs/reqwest/)
- [hyper GitHub](https://github.com/hyperium/hyper)
- [Using connection pooling in Rust](https://app.studyraid.com/en/read/11242/350323/using-connection-pooling)
- [How to choose the right Rust HTTP client](https://blog.logrocket.com/best-rust-http-client/)

### Python HTTP Clients
- [HTTPX vs Requests (Oxylabs)](https://oxylabs.io/blog/httpx-vs-requests-vs-aiohttp)
- [Requests vs HTTPX (ScrapingAnt)](https://scrapingant.com/blog/requests-vs-httpx)
- [Best Python HTTP Clients 2025](https://proxyway.com/guides/the-best-python-http-clients)
- [HTTPX Documentation](https://www.python-httpx.org/)
- [Beyond Requests: httpx](https://towardsdatascience.com/beyond-requests-why-httpx-is-the-modern-http-client-you-need-sometimes/)

### Node.js HTTP Clients
- [Axios vs Fetch 2025](https://blog.logrocket.com/axios-vs-fetch-2025/)
- [Exploring HTTP Clients](https://blog.platformatic.dev/exploring-http-clients-axios-requests-and-node-fetch)
- [axios GitHub](https://github.com/axios/axios)
- [Fetch API vs Axios vs Alova](https://www.freecodecamp.org/news/fetch-api-vs-axios-vs-alova/)

### HTTP Client Patterns
- [Connection Management (Apache HttpClient)](https://hc.apache.org/httpcomponents-client-4.5.x/current/tutorial/html/connmgmt.html)
- [HTTP Interceptor Patterns](https://requestly.com/blog/http-interceptor-patterns-for-secure-and-efficient-api-communication/)
- [HttpClient Connection Pooling .NET](https://www.stevejgordon.co.uk/httpclient-connection-pooling-in-dotnet-core)
- [Resilient HTTP Requests .NET](https://aykutaktas06.medium.com/implementing-resilient-http-requests-in-net-core-f7010981e2e4)

### Async/Await Patterns
- [HTTPX Async Support](https://www.python-httpx.org/async/)
- [Best Practices Async/Await C#](https://blog.stackademic.com/best-practices-for-using-async-await-in-c-with-net-core-b067ea3fa9d3)
- [High-Concurrency C# Async](https://medium.com/@orbens/handling-high-concurrency-scenarios-in-c-with-async-await-and-channels-7b3775ca2f95)
- [Async HTTP Client Java](https://www.baeldung.com/async-http-client)

### Retry and Resilience Patterns
- [Retry with Backoff (AWS)](https://docs.aws.amazon.com/prescriptive-guidance/latest/cloud-design-patterns/retry-backoff.html)
- [HTTP Retries with Polly](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/implement-resilient-applications/implement-http-call-retries-exponential-backoff-polly)
- [Resilience Patterns (codecentric)](https://www.codecentric.de/en/knowledge-hub/blog/resilience-design-patterns-retry-fallback-timeout-circuit-breaker)
- [Build Resilient HTTP Apps .NET](https://learn.microsoft.com/en-us/dotnet/core/resilience/http-resilience)
- [Building Resilient HTTP Clients with Polly](https://blog.ludmal.com/p/building-resilient-http-clients-with)

### Error Handling and Security
- [SSL/TLS Handshake Errors (Sectigo)](https://www.sectigo.com/blog/tls-ssl-handshake-errors-how-to-fix-them)
- [Troubleshoot HTTP Errors](https://squareops.com/blog/comprehensive-guide-to-http-errors-in-devops-causes-scenarios-and-troubleshooting-steps/)
- [AWS SDK Troubleshooting](https://docs.aws.amazon.com/sdk-for-java/latest/developer-guide/troubleshooting.html)
- [SSL Connect Error Fix](https://www.digitalocean.com/community/tutorials/ssl-connect-error)

### Cookie and Redirect Handling
- [HttpClient Cookie Guide](https://hc.apache.org/httpclient-legacy/cookies.html)
- [Cookie Management Apache HttpClient](https://www.tutorialspoint.com/apache_httpclient/apache_httpclient_cookies_management.htm)
- [HttpClient Cookies .NET Issue](https://github.com/dotnet/runtime/issues/16983)
- [Java Stateful Sessions Cookies](https://relentlesscoding.com/posts/java-stateful-sessions-or-how-to-properly-send-cookies-with-each-redirect-request/)

### API Design Principles
- [Web Platform Design Principles](https://w3ctag.github.io/design-principles/)
- [Azure API Design Best Practices](https://learn.microsoft.com/en-us/azure/architecture/best-practices/api-design)
- [Creating HTTP API Wrapper Library](https://medium.com/tompee/creating-your-own-http-api-wrapper-library-design-patterns-10b2e232e92d)
- [IDEAL HTTP Client Guide](https://evilmartians.com/chronicles/its-dangerous-to-go-alone-take-our-guide-to-the-ideal-http-client)
- [API Design Principles (YourBasic)](https://yourbasic.org/algorithms/your-basic-api/)

### Eiffel-Specific
- [Eiffel Networking Solutions](https://www.eiffel.org/doc/solutions/Networking)
- [Eiffel Clients and Servers](https://www.eiffel.org/doc/solutions/Clients_and_servers)
- [Eiffel-Loop HTTP Client](http://www.eiffel-loop.com/library/http-client.html)
- [EiffelStudio 18.01 Releases](https://dev.eiffel.com/EiffelStudio_18.01_Releases)
- [Making HTTP Request in Eiffel](https://gnuu.org/2008/09/19/a-basic-http-req-in-eiffel-and-rant/)

---

**Report Generated:** 2025-12-08
**Author:** Claude Opus 4.5 (Research Assistant)
**For:** simple_http library enhancement planning
