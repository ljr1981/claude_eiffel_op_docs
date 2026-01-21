# simple_grpc Research Document

## 7-Step Research Process Complete

**Date:** 2025-12-14
**Library:** simple_grpc
**Purpose:** gRPC protocol for Eiffel - Protocol Buffers, HTTP/2, streaming RPC, service definitions

---

## Step 1: Industry Specifications

### gRPC Protocol over HTTP/2
- **Source:** [grpc/PROTOCOL-HTTP2.md](https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-HTTP2.md)
- gRPC uses HTTP/2 as transport layer
- Request: POST method, path = /Service-Name/method-name
- Required headers: content-type (application/grpc), te (trailers)
- Message framing: 1-byte compressed flag + 4-byte length + payload

### Protocol Buffers Wire Format
- **Source:** [protobuf.dev/encoding](https://protobuf.dev/programming-guides/encoding/)
- Wire types: VARINT(0), I64(1), LEN(2), SGROUP(3), EGROUP(4), I32(5)
- Tag encoding: (field_number << 3) | wire_type
- Varint: 7 bits per byte, MSB continuation flag
- ZigZag for signed: (n << 1) ^ (n >> 31)

### gRPC Status Codes
- **Source:** [grpc.io/status-codes](https://grpc.io/docs/guides/status-codes/)
- 17 status codes (0-16): OK, CANCELLED, UNKNOWN, INVALID_ARGUMENT, etc.

### HTTP/2 Frame Structure
- **Source:** [RFC 7540](https://httpwg.org/specs/rfc7540.html)
- 9-byte header: 3-byte length, 1-byte type, 1-byte flags, 4-byte stream ID
- Frame types: DATA, HEADERS, SETTINGS, WINDOW_UPDATE, etc.

---

## Step 2: Existing Implementations Analyzed

### Go gRPC (google.golang.org/grpc)
- Channel-based architecture
- Interceptors for middleware
- Reflection for service discovery

### Java gRPC (io.grpc)
- ManagedChannel for connection pooling
- ServerBuilder pattern
- Stub generation from .proto files

### Python grpcio
- Synchronous and async APIs
- Channel credentials for TLS
- Interceptors for logging/auth

---

## Step 3: Key Patterns Identified

### 1. Channel Pattern
- Abstraction over HTTP/2 connection
- Connection pooling, load balancing
- Lifecycle: create -> use -> shutdown

### 2. Stub Pattern
- Generated client code from .proto
- Method mapping to RPC calls
- Type-safe request/response

### 3. Service Definition
- Interface-like contract
- Method registration
- Request/response type binding

### 4. Streaming Types
- Unary: single request -> single response
- Server streaming: single request -> stream of responses
- Client streaming: stream of requests -> single response
- Bidirectional: stream <-> stream

---

## Step 4: Eiffel Design Decisions

### Architecture
```
SIMPLE_GRPC (Facade)
  |
  +-- SIMPLE_GRPC_CHANNEL (Connection management)
  |     +-- SIMPLE_HTTP2_CONNECTION (HTTP/2 layer)
  |
  +-- SIMPLE_GRPC_SERVICE (Service definition)
  |     +-- SIMPLE_GRPC_METHOD (Method definitions)
  |
  +-- SIMPLE_PROTOBUF (Wire format)
  |     +-- SIMPLE_PROTOBUF_MESSAGE
  |     +-- SIMPLE_PROTOBUF_FIELD
  |
  +-- SIMPLE_GRPC_CALL (RPC execution)
        +-- SIMPLE_GRPC_STREAM (Streaming support)
```

### Class Responsibilities

1. **SIMPLE_GRPC** - Main facade with factory methods
2. **SIMPLE_GRPC_CHANNEL** - HTTP/2 connection to server
3. **SIMPLE_GRPC_SERVICE** - Service contract definition
4. **SIMPLE_GRPC_METHOD** - Individual RPC method
5. **SIMPLE_GRPC_CALL** - Active RPC call
6. **SIMPLE_GRPC_STATUS** - Status code constants
7. **SIMPLE_GRPC_METADATA** - Headers/trailers
8. **SIMPLE_PROTOBUF** - Wire format encoding/decoding
9. **SIMPLE_PROTOBUF_MESSAGE** - Message container
10. **SIMPLE_PROTOBUF_FIELD** - Field definition
11. **SIMPLE_HTTP2_FRAME** - HTTP/2 frame structure
12. **SIMPLE_GRPC_STREAM** - Streaming support

---

## Step 5: API Design

### Creating a Channel
```eiffel
channel := grpc.new_channel ("localhost", 50051)
channel.set_plaintext  -- or set_tls (credentials)
```

### Defining a Service
```eiffel
service := grpc.new_service ("greeter.Greeter")
service.add_unary_method ("SayHello", request_type, response_type)
```

### Making Calls
```eiffel
-- Unary call
call := channel.new_call (service, "SayHello")
call.set_request (request_message)
call.execute
if call.is_ok then
    response := call.response
end
```

### Streaming
```eiffel
-- Server streaming
stream := channel.new_server_stream (service, "ListFeatures")
stream.send_request (request)
from stream.start until stream.after loop
    process (stream.item)
    stream.forth
end
```

---

## Step 6: Test Strategy

1. **Protocol Buffers Tests**
   - Varint encoding/decoding
   - ZigZag encoding
   - Field tag encoding
   - Message serialization

2. **HTTP/2 Frame Tests**
   - Frame header parsing
   - DATA frame encoding
   - HEADERS frame encoding

3. **gRPC Message Tests**
   - Length-prefixed message framing
   - Metadata encoding
   - Status code handling

4. **Integration Tests**
   - Channel creation
   - Service definition
   - Mock call execution

---

## Step 7: Implementation Notes

### Constraints
- No TLS in Phase 1 (plaintext only)
- No HPACK compression (simplified headers)
- No actual TCP socket (use socket abstraction for testing)
- Focus on protocol correctness over performance

### Dependencies
- simple_codec (for Base64)
- ISE base library
- ISE time library

### Wire Format Priority
1. Varint encoding (fundamental)
2. Field tags
3. Length-delimited strings/bytes
4. Nested messages
5. Repeated fields (later)

### Future Enhancements
- TLS/SSL support
- HPACK header compression
- Connection pooling
- Load balancing
- Interceptor chain
- Proto file parsing

---

## Sources

- [gRPC Protocol over HTTP/2](https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-HTTP2.md)
- [Protocol Buffers Encoding](https://protobuf.dev/programming-guides/encoding/)
- [gRPC Status Codes](https://grpc.io/docs/guides/status-codes/)
- [gRPC Core Concepts](https://grpc.io/docs/what-is-grpc/core-concepts/)
- [RFC 7540 - HTTP/2](https://httpwg.org/specs/rfc7540.html)
- [RFC 7541 - HPACK](https://httpwg.org/specs/rfc7541.html)
