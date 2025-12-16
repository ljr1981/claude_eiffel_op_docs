# simple_docker Vision Document

**Container Management for Eiffel Applications**

*Research Date: December 15, 2025*
*Status: VISION (Pre-Development)*

---

## Executive Summary

`simple_docker` is a proposed Eiffel library for programmatic Docker container management. It would enable Eiffel applications to build, run, and manage Docker containers without shelling out to CLI commands, bringing containerization capabilities directly into the Simple Eiffel ecosystem.

**Primary Goal**: Enable Eiffel developers to containerize and deploy applications programmatically with Design by Contract guarantees.

---

## Research Findings

### 1. Existing Docker SDKs

| Language | SDK | Status |
|----------|-----|--------|
| **Go** | docker/docker (official) | Production, most complete |
| **Python** | docker-py | Production, official |
| **Rust** | shiplift, bollard | Community-maintained |
| **C** | [libdocker](https://github.com/danielsuo/libdocker) | Community, wraps REST API |

**Key Insight**: Docker Engine exposes a REST API over Unix socket (`/var/run/docker.sock`). Any language with HTTP/socket capabilities can interact with Docker.

### 2. Docker Engine API Capabilities

The Docker Engine API (v1.44+) provides:

| Category | Operations |
|----------|------------|
| **Containers** | Create, Start, Stop, Kill, Remove, Inspect, Logs, Attach, Wait, Exec |
| **Images** | Pull, Push, Build, Tag, Remove, Inspect, List, Search |
| **Networks** | Create, Connect, Disconnect, Remove, Inspect |
| **Volumes** | Create, Remove, Inspect, List |
| **System** | Info, Version, Events, Ping |

All operations use JSON request/response over HTTP.

### 3. Existing Eiffel Docker Images

EiffelStudio Docker images already exist:

| Image | Source | Description |
|-------|--------|-------------|
| `eiffel/eiffel:latest` | [eiffel-docker/eiffel](https://github.com/eiffel-docker/eiffel) | EiffelStudio 23.09 on Debian |
| `eiffel/eiffel:dev` | Same | Latest dev release |
| `mmonga/docker-eiffel` | [monga/docker-eiffel](https://github.com/monga/docker-eiffel) | GPL version with VNC |
| `jumanjiman/eiffelstudio` | [jumanjiman/docker-eiffelstudio](https://github.com/jumanjiman/docker-eiffelstudio) | Fedora-based |

**What's Missing**: Programmatic control from Eiffel code.

### 4. Kubernetes C Client

An official [Kubernetes C client](https://github.com/kubernetes-client/c) exists, enabling future `simple_kubernetes` development.

---

## Vision: What simple_docker Should Provide

### Core Capabilities

```
simple_docker/
├── DOCKER_CLIENT           -- Main facade
├── DOCKER_CONTAINER        -- Container management
├── DOCKER_IMAGE            -- Image operations
├── DOCKER_NETWORK          -- Network configuration
├── DOCKER_VOLUME           -- Volume management
├── DOCKERFILE_BUILDER      -- Programmatic Dockerfile generation
├── COMPOSE_BUILDER         -- docker-compose.yaml generation
└── CONTAINER_SPEC          -- Container configuration with DBC
```

### Phase 1: Container Basics (MVP)

```eiffel
class DOCKER_CLIENT

feature -- Container Operations

    run_container (a_spec: CONTAINER_SPEC): DOCKER_CONTAINER
            -- Create and start a container from specification.
        require
            spec_valid: a_spec.is_valid
            image_exists: image_exists (a_spec.image)
        ensure
            container_running: Result.is_running
        end

    stop_container (a_container: DOCKER_CONTAINER)
            -- Stop a running container.
        require
            container_running: a_container.is_running
        ensure
            container_stopped: not a_container.is_running
        end

    container_logs (a_container: DOCKER_CONTAINER): STRING
            -- Get container stdout/stderr logs.
        require
            container_exists: a_container.exists
        end

feature -- Image Operations

    pull_image (a_name: STRING)
            -- Pull image from registry.
        require
            name_not_empty: not a_name.is_empty
        ensure
            image_exists: image_exists (a_name)
        end

    build_image (a_dockerfile: DOCKERFILE_BUILDER; a_tag: STRING)
            -- Build image from Dockerfile specification.
        require
            dockerfile_valid: a_dockerfile.is_valid
            tag_not_empty: not a_tag.is_empty
        ensure
            image_exists: image_exists (a_tag)
        end

end
```

### Phase 2: Dockerfile Builder (Fluent API)

```eiffel
class DOCKERFILE_BUILDER

feature -- Builder Pattern

    from_image (a_image: STRING): like Current
            -- Set base image (FROM).
        require
            image_not_empty: not a_image.is_empty
        end

    run (a_command: STRING): like Current
            -- Add RUN instruction.
        end

    copy (a_source, a_dest: STRING): like Current
            -- Add COPY instruction.
        end

    workdir (a_path: STRING): like Current
            -- Set working directory.
        end

    expose (a_port: INTEGER): like Current
            -- Expose port.
        require
            valid_port: a_port > 0 and a_port <= 65535
        end

    env (a_name, a_value: STRING): like Current
            -- Set environment variable.
        end

    cmd (a_command: ARRAY [STRING]): like Current
            -- Set default command.
        end

    entrypoint (a_command: ARRAY [STRING]): like Current
            -- Set entrypoint.
        end

    to_string: STRING
            -- Generate Dockerfile content.
        ensure
            valid_dockerfile: Result.starts_with ("FROM ")
        end

end

-- Usage Example:
dockerfile := (create {DOCKERFILE_BUILDER})
    .from_image ("eiffel/eiffel:latest")
    .workdir ("/app")
    .copy (".", "/app")
    .run ("ec -config my_app.ecf -target my_app -c_compile")
    .expose (8080)
    .cmd (<<"./EIFGENs/my_app/W_code/my_app">>)

docker.build_image (dockerfile, "my-eiffel-app:latest")
```

### Phase 3: Container Specification with DBC

```eiffel
class CONTAINER_SPEC

feature -- Configuration

    image: STRING
            -- Docker image to use

    name: detachable STRING
            -- Optional container name

    ports: HASH_TABLE [INTEGER, INTEGER]
            -- Port mappings (host -> container)

    volumes: HASH_TABLE [STRING, STRING]
            -- Volume mounts (host_path -> container_path)

    environment: HASH_TABLE [STRING, STRING]
            -- Environment variables

    network: detachable STRING
            -- Network to connect to

    memory_limit: INTEGER
            -- Memory limit in bytes (0 = unlimited)

    cpu_limit: REAL
            -- CPU limit (1.0 = 1 core)

feature -- Validation (DBC)

    is_valid: BOOLEAN
            -- Is this specification valid?
        do
            Result := not image.is_empty
        end

invariant
    memory_non_negative: memory_limit >= 0
    cpu_non_negative: cpu_limit >= 0.0

end
```

### Phase 4: Docker Compose Builder

```eiffel
class COMPOSE_BUILDER

feature -- Services

    add_service (a_name: STRING; a_spec: CONTAINER_SPEC): like Current
            -- Add a service definition.
        require
            name_not_empty: not a_name.is_empty
            spec_valid: a_spec.is_valid
        end

    depends_on (a_service, a_dependency: STRING): like Current
            -- Set service dependency.
        require
            service_exists: has_service (a_service)
            dependency_exists: has_service (a_dependency)
        end

feature -- Networks

    add_network (a_name: STRING): like Current
            -- Add network definition.
        end

feature -- Output

    to_yaml: STRING
            -- Generate docker-compose.yaml content.
        end

    save (a_path: STRING)
            -- Save to file.
        end

end
```

### Phase 5: Eiffel-Specific Templates

```eiffel
class EIFFEL_CONTAINER_TEMPLATES

feature -- Pre-built Templates

    eiffel_app_dockerfile (a_ecf: STRING; a_target: STRING): DOCKERFILE_BUILDER
            -- Generate Dockerfile for an Eiffel application.
        do
            create Result
            Result
                .from_image ("eiffel/eiffel:latest")
                .workdir ("/app")
                .copy (".", "/app")
                .run ("ec -config " + a_ecf + " -target " + a_target + " -c_compile")
                .cmd (<<eifgen_path (a_target)>>)
        end

    eiffel_web_dockerfile (a_ecf, a_target: STRING; a_port: INTEGER): DOCKERFILE_BUILDER
            -- Generate Dockerfile for Eiffel web service.
        do
            Result := eiffel_app_dockerfile (a_ecf, a_target)
            Result.expose (a_port)
        end

end
```

---

## Implementation Approach

### Option A: Wrap libdocker (C Library)

```eiffel
feature {NONE} -- External (Inline C)

    c_docker_init: POINTER
        external "C inline use <docker.h>"
        alias "return docker_init();"
        end

    c_docker_get (a_handle: POINTER; a_endpoint: POINTER): POINTER
        external "C inline use <docker.h>"
        alias "return docker_get($a_handle, $a_endpoint);"
        end
```

**Pros**: Reuse existing C library
**Cons**: Additional dependency (libdocker)

### Option B: Direct HTTP to Docker Socket

```eiffel
feature -- Implementation

    docker_get (a_endpoint: STRING): STRING
            -- GET request to Docker API.
        local
            l_socket: UNIX_SOCKET
        do
            create l_socket.make_client ("/var/run/docker.sock")
            l_socket.connect
            l_socket.put_string ("GET " + a_endpoint + " HTTP/1.1%R%NHost: localhost%R%N%R%N")
            Result := l_socket.read_string
            l_socket.close
        end
```

**Pros**: No external dependencies (use simple_http or raw sockets)
**Cons**: More implementation work

### Recommendation: Option B (Direct Socket/Pipe)

The Docker Engine API is simple REST over a local IPC mechanism. We already have `simple_http` for the HTTP protocol layer.

**Platform-Specific Transport:**

| Platform | Transport | Path |
|----------|-----------|------|
| **Linux/macOS** | Unix socket | `/var/run/docker.sock` |
| **Windows** | Named pipe | `\\.\pipe\docker_engine` |

```eiffel
feature -- Connection

    connect
            -- Connect to Docker Engine.
        do
            if {PLATFORM}.is_windows then
                connect_named_pipe ("\\.\pipe\docker_engine")
            else
                connect_unix_socket ("/var/run/docker.sock")
            end
        ensure
            connected: is_connected
        end

feature {NONE} -- Platform-specific

    connect_named_pipe (a_path: STRING)
            -- Connect via Windows named pipe.
        require
            is_windows: {PLATFORM}.is_windows
        do
            -- Use CreateFile Win32 API (same pattern as simple_win32_api)
            pipe_handle := c_create_file (a_path)
        end

    connect_unix_socket (a_path: STRING)
            -- Connect via Unix domain socket.
        require
            is_unix: not {PLATFORM}.is_windows
        do
            -- Standard socket API
            socket_fd := c_socket_connect (a_path)
        end
```

**Windows named pipes** can be accessed via Win32 `CreateFile` - a pattern we already use in `simple_win32_api`. This keeps the library self-contained within the Simple Eiffel ecosystem with no external dependencies.

---

## Development Phases

| Phase | Scope | Effort | Priority |
|-------|-------|--------|----------|
| **1** | Container run/stop/logs, Image pull (Windows named pipe + Unix socket) | LOW | HIGH |
| **2** | Dockerfile builder (fluent API) | LOW | HIGH |
| **3** | Container spec with full options | MEDIUM | MEDIUM |
| **4** | Docker Compose builder | MEDIUM | MEDIUM |
| **5** | Eiffel-specific templates | LOW | MEDIUM |
| **6** | Windows native containers (not Linux-on-Windows) | HIGH | LOW |

**Note on Phase 1**: Windows support via named pipe is included from the start - Docker Desktop for Windows uses the same REST API, just over `\\.\pipe\docker_engine` instead of a Unix socket. Phase 6 refers to *Windows native containers* (Windows Server containers), which are a separate container runtime with different base images.

---

## Use Cases

### 1. CI/CD Integration (simple_ci)

```eiffel
-- In simple_ci pipeline
docker := create {DOCKER_CLIENT}.make
container := docker.run_container (
    create {CONTAINER_SPEC}.make_with_image ("eiffel/eiffel:latest")
        .with_volume (project_path, "/app")
        .with_command ("ec -config my_app.ecf -c_compile")
)
docker.wait_for (container)
if container.exit_code = 0 then
    print ("Build succeeded%N")
else
    print ("Build failed: " + docker.container_logs (container))
end
```

### 2. Microservice Deployment

```eiffel
-- Deploy simple_web application
dockerfile := templates.eiffel_web_dockerfile ("my_web.ecf", "my_web_exe", 8080)
docker.build_image (dockerfile, "my-web-service:1.0")

container := docker.run_container (
    create {CONTAINER_SPEC}.make_with_image ("my-web-service:1.0")
        .with_port (8080, 8080)
        .with_env ("DATABASE_URL", db_url)
)
```

### 3. Test Environment Setup

```eiffel
-- Spin up test database
postgres := docker.run_container (
    create {CONTAINER_SPEC}.make_with_image ("postgres:15")
        .with_port (5432, 5432)
        .with_env ("POSTGRES_PASSWORD", "test")
        .with_name ("test-db")
)

-- Run tests
run_tests

-- Cleanup
docker.stop_container (postgres)
docker.remove_container (postgres)
```

---

## Dependencies

| Dependency | Purpose | Status |
|------------|---------|--------|
| **simple_ipc** | Cross-platform IPC (named pipes + Unix sockets) | **NEW - MUST CREATE FIRST** |
| simple_http | HTTP protocol layer | EXISTS (chunked encoding issues) |
| simple_json | JSON parsing/generation | EXISTS |
| simple_file | File operations | EXISTS |
| simple_archive | Tar archive creation for build context | EXISTS |
| simple_process | Process execution (fallback) | EXISTS |

**Development Order**:
1. **Create `simple_ipc`** first (prerequisite for simple_docker)
2. Then create `simple_docker` using simple_ipc for Docker Engine communication

**simple_ipc Implementation Notes**:
- Use inline C (Eric Bezault pattern) - NO external library dependencies
- Windows: `CreateFile`, `ReadFile`, `WriteFile` for named pipes
- Linux/macOS: `socket()`, `connect()`, `read()`, `write()` for Unix sockets
- Reference WEL_PIPE and UNIX_STREAM_SOCKET for API patterns only (don't depend on them)
- Single unified `IPC_CONNECTION` interface for both platforms

---

## Success Metrics

| Metric | Target |
|--------|--------|
| Container operations (run/stop/logs) | 100% API coverage |
| Image operations (pull/build) | 100% API coverage |
| Dockerfile generation | Full instruction set |
| Compose generation | Service + network + volume |
| Windows (Docker Desktop) | Day-1 support via named pipe |
| Linux/macOS | Phase 1 with Unix socket |

---

## Conclusion

`simple_docker` would bring containerization into the Simple Eiffel ecosystem, enabling:

1. **Programmatic container management** from Eiffel code
2. **Design by Contract** for container specifications
3. **CI/CD integration** with simple_ci
4. **Microservice deployment** for simple_web applications
5. **Eiffel-specific templates** for common patterns

**Recommended Priority**: HIGH (enables modern deployment workflows)
**Estimated Effort**: MEDIUM (6 phases, core in 2-3 phases)

---

## Developer Pain Points (Research)

Understanding common frustrations helps design a better SDK.

### 1. Docker 2024 State of Application Development Survey

| Pain Point | Finding |
|------------|---------|
| **Security complexity** | 34% of developers find container security tasks difficult |
| **Build performance** | Slow builds remain a top frustration |
| **"Works on my machine"** | Environment inconsistency still problematic |
| **Debug visibility** | Difficulty debugging containers in production |

### 2. docker-py GitHub Issues (Common Patterns)

| Issue | Description | simple_docker Mitigation |
|-------|-------------|--------------------------|
| **API hangs** | `containers.run()` hangs indefinitely on missing images | Explicit timeout parameters with DBC contracts |
| **Silent failures** | Operations fail without clear error messages | Detailed error types with contract violations |
| **Version mismatches** | API version incompatibility with Docker Engine | Version negotiation in `DOCKER_CLIENT.make` |
| **Resource leaks** | Containers/networks not cleaned up on errors | RAII pattern with `dispose` + invariants |
| **Streaming complexity** | Log streaming and attach operations hard to use | Simple iterator pattern for logs |

### 3. Common SDK Anti-Patterns to Avoid

| Anti-Pattern | Problem | simple_docker Approach |
|--------------|---------|------------------------|
| **Magic defaults** | Unexpected behavior from hidden defaults | Explicit configuration, no surprises |
| **Stringly-typed API** | Errors only at runtime | Strong typing with `CONTAINER_SPEC`, `DOCKERFILE_BUILDER` |
| **Callback hell** | Complex async handling | Synchronous by default, SCOOP for async |
| **Monolithic client** | Hard to test, mock | Separate classes per domain (container, image, network) |

### 4. Key Design Decisions from Pain Points

1. **Fail fast with contracts**: Preconditions catch errors before Docker API calls
2. **Explicit timeouts**: All operations accept timeout parameters (default 30s)
3. **Detailed errors**: `DOCKER_ERROR` class with error code, message, and suggestion
4. **Resource tracking**: `DOCKER_CLIENT` tracks created resources for cleanup
5. **Version awareness**: Client negotiates API version on connection

---

## Extended Research Findings

### 8. Docker API Authentication

| Scenario | Authentication | Notes |
|----------|----------------|-------|
| **Local socket/pipe** | None required | Relies on filesystem permissions |
| **Remote TCP (insecure)** | None | Port 2375 - NOT recommended |
| **Remote TCP (TLS)** | Mutual TLS | Port 2376 - CA cert, client cert, client key |
| **Private registries** | Bearer token (OAuth 2.0) | See Registry Authentication below |

**Key Finding**: Local connections (our primary use case) require no authentication. The Docker socket/pipe is protected by OS-level access control.

```eiffel
-- For local Docker Desktop on Windows:
create client.make  -- No auth needed, just connect to named pipe

-- For remote Docker host (future):
create client.make_with_tls (host, ca_cert, client_cert, client_key)
```

**Security Statistics (2024)**:
- 90% of exposed Docker TCP endpoints have no authentication ([NCC Group](https://www.howtogeek.com/devops/how-to-secure-dockers-tcp-socket-with-tls/))
- 68% of container breaches involve exposed APIs ([Sysdig 2024](https://docs.docker.com/engine/security/protect-access/))

### 9. Streaming Response Handling

Docker uses streaming for logs, build output, and exec attach. This is a **critical implementation concern**.

| Operation | Response Type | Handling Strategy |
|-----------|---------------|-------------------|
| `GET /containers/{id}/logs` | Chunked stream | Read in chunks, yield to caller |
| `POST /build` | Chunked JSON stream | Parse JSON objects as they arrive |
| `POST /containers/{id}/attach` | Raw stream (stdin/stdout/stderr multiplexed) | Demultiplex using Docker stream protocol |
| `GET /events` | Server-sent events | Long-lived connection with JSON events |

**Known Issues from docker-py**:
- HTTP libraries buffer chunks, breaking interactivity ([GitHub Issue #77](https://github.com/upserve/docker-api/issues/77))
- JSON response may be incomplete with chunked encoding ([moby #16118](https://github.com/moby/moby/issues/16118))

**Mitigation for simple_docker**:
```eiffel
feature -- Streaming

    read_log_stream (a_container: STRING): ITERATION_CURSOR [STRING]
            -- Stream logs line by line.
        do
            -- Use small buffer (1-4KB) for responsiveness
            -- Yield each complete line to caller
            -- Handle incomplete chunks at buffer boundary
        end
```

**We already hit this issue**: simple_http has chunked encoding problems. simple_docker must handle this properly.

### 10. Tar Archive for Build Context

Docker build requires sending context as a tar archive. **simple_archive already supports this!**

| Requirement | simple_archive Support | Status |
|-------------|------------------------|--------|
| Create tar from files | `create_archive (path, files)` | EXISTS |
| Create tar from directory | `create_archive_from_directory (path, dir)` | EXISTS |
| Compression (gzip, bzip2, xz) | Via ISE etar library | VERIFY |

**Docker Build Flow**:
```
1. Client creates tar of build context
2. POST /build with tar as request body
3. Docker extracts tar, runs Dockerfile
4. Returns streaming build output
```

**simple_docker implementation**:
```eiffel
build_image (a_dockerfile: DOCKERFILE_BUILDER; a_context_dir: STRING; a_tag: STRING)
        -- Build image from Dockerfile and context directory.
    local
        l_archive: SIMPLE_ARCHIVE
        l_tar_path: STRING
    do
        -- Create tar from context directory
        l_tar_path := temp_file_path + ".tar"
        create l_archive.make
        l_archive.create_archive_from_directory (l_tar_path, a_context_dir)

        -- POST tar to Docker API
        post_build (l_tar_path, a_tag)
    end
```

### 11. Container State Machine

Containers follow a well-defined lifecycle:

```
                    ┌─────────┐
                    │  IMAGE  │
                    └────┬────┘
                         │ docker create
                         ▼
┌──────────────────────────────────────────────────────────┐
│                      CREATED                              │
└────┬─────────────────────────────────────────────────────┘
     │ docker start
     ▼
┌──────────────────────────────────────────────────────────┐
│                      RUNNING                              │◄─────┐
└────┬──────────────┬──────────────┬───────────────────────┘      │
     │              │              │                               │
     │ pause        │ stop         │ kill                         │
     ▼              ▼              ▼                               │
┌─────────┐   ┌──────────┐   ┌──────────┐                         │
│ PAUSED  │   │ EXITED   │   │ EXITED   │                         │
└────┬────┘   └────┬─────┘   └──────────┘                         │
     │             │                                               │
     │ unpause     │ restart                                       │
     └─────────────┴───────────────────────────────────────────────┘
```

| State | Description | Allowed Transitions |
|-------|-------------|---------------------|
| **created** | Container exists but never started | start, remove |
| **running** | Process executing | pause, stop, kill |
| **paused** | Process suspended (memory preserved, CPU released) | unpause |
| **exited** | Process terminated (exit code available) | restart, remove |
| **dead** | Failed to remove (resources stuck) | force remove |

**DBC Contracts**:
```eiffel
stop_container (a_container: DOCKER_CONTAINER)
    require
        container_running: a_container.is_running
    ensure
        container_stopped: a_container.state = State_exited
    end

start_container (a_container: DOCKER_CONTAINER)
    require
        container_startable: a_container.state = State_created or a_container.state = State_exited
    ensure
        container_running: a_container.is_running
    end
```

### 12. Actual Docker API Request/Response Examples

**Captured from Docker Desktop 4.44.3 (API v1.51)**:

**GET /version**:
```json
{
  "Version": "28.3.2",
  "ApiVersion": "1.51",
  "MinAPIVersion": "1.24",
  "Os": "linux",
  "Arch": "amd64",
  "KernelVersion": "6.6.87.2-microsoft-standard-WSL2"
}
```

**GET /containers/json** (actual output):
```json
{
  "Id": "cc9889be04c1",
  "Names": ["/angry_taussig"],
  "Image": "0eba4c05742a",
  "State": "exited",
  "Status": "Exited (1) 2 months ago",
  "Ports": [],
  "Networks": "bridge"
}
```

**POST /containers/create** (request body):
```json
{
  "Image": "alpine",
  "Cmd": ["echo", "hello world"],
  "Env": ["MY_VAR=value"],
  "HostConfig": {
    "PortBindings": {"8080/tcp": [{"HostPort": "8080"}]},
    "Binds": ["/host/path:/container/path"]
  }
}
```

**Response**: `{"Id": "container_id", "Warnings": []}`

### 13. Existing Eiffel IPC Code & New Library Requirement

**Reference Material** (study, don't depend on):

| Reference | Library | Class | Use As |
|-----------|---------|-------|--------|
| Windows named pipe | WEL | `WEL_PIPE` | **REFERENCE ONLY** - WEL is Windows-only, won't compile on Linux/macOS |
| Unix socket | NET | `UNIX_STREAM_SOCKET` | **REFERENCE ONLY** - for understanding socket API |

**Existing simple_* dependencies**:

| Need | Library | Class | Status |
|------|---------|-------|--------|
| HTTP protocol | simple_http | `SIMPLE_HTTP` | EXISTS (chunked issues to address) |
| JSON parsing | simple_json | `SIMPLE_JSON` | EXISTS |
| Tar archives | simple_archive | `SIMPLE_ARCHIVE` | EXISTS |

**NEW LIBRARY REQUIRED: `simple_ipc`**

simple_docker needs a new `simple_ipc` library that provides cross-platform IPC using **inline C** (Eric Bezault pattern), NOT depending on WEL or other platform-specific libraries.

```eiffel
-- simple_ipc: Cross-platform IPC with inline C

class SIMPLE_IPC

feature -- Factory

    named_pipe (a_name: STRING): IPC_CONNECTION
            -- Connect to Windows named pipe.
        require
            is_windows: {PLATFORM}.is_windows
        do
            create {NAMED_PIPE_CONNECTION} Result.make (a_name)
        end

    unix_socket (a_path: STRING): IPC_CONNECTION
            -- Connect to Unix domain socket.
        require
            is_unix: not {PLATFORM}.is_windows
        do
            create {UNIX_SOCKET_CONNECTION} Result.make (a_path)
        end

    docker_connection: IPC_CONNECTION
            -- Platform-appropriate connection to Docker Engine.
        do
            if {PLATFORM}.is_windows then
                Result := named_pipe ("docker_engine")
            else
                Result := unix_socket ("/var/run/docker.sock")
            end
        end

end
```

**Windows Named Pipe (inline C, no WEL dependency)**:
```eiffel
class NAMED_PIPE_CONNECTION

inherit
    IPC_CONNECTION

feature {NONE} -- Win32 API (inline C)

    c_open_pipe (a_name: POINTER): POINTER
            -- CreateFile to open named pipe.
        external
            "C inline use <windows.h>"
        alias
            "[
                return CreateFileW(
                    (LPCWSTR)$a_name,
                    GENERIC_READ | GENERIC_WRITE,
                    0, NULL, OPEN_EXISTING, 0, NULL
                );
            ]"
        end

    c_read_pipe (a_handle, a_buffer: POINTER; a_size: INTEGER; a_bytes_read: TYPED_POINTER [INTEGER]): BOOLEAN
        external
            "C inline use <windows.h>"
        alias
            "return (EIF_BOOLEAN)ReadFile((HANDLE)$a_handle, $a_buffer, (DWORD)$a_size, (LPDWORD)$a_bytes_read, NULL);"
        end

    c_write_pipe (a_handle, a_buffer: POINTER; a_size: INTEGER; a_bytes_written: TYPED_POINTER [INTEGER]): BOOLEAN
        external
            "C inline use <windows.h>"
        alias
            "return (EIF_BOOLEAN)WriteFile((HANDLE)$a_handle, $a_buffer, (DWORD)$a_size, (LPDWORD)$a_bytes_written, NULL);"
        end

    c_close_handle (a_handle: POINTER): BOOLEAN
        external
            "C inline use <windows.h>"
        alias
            "return (EIF_BOOLEAN)CloseHandle((HANDLE)$a_handle);"
        end

end
```

**Unix Socket (inline C)**:
```eiffel
class UNIX_SOCKET_CONNECTION

inherit
    IPC_CONNECTION

feature {NONE} -- Unix API (inline C)

    c_socket_connect (a_path: POINTER): INTEGER
            -- Create and connect Unix domain socket.
        external
            "C inline use <sys/socket.h>, <sys/un.h>, <unistd.h>, <string.h>"
        alias
            "[
                struct sockaddr_un addr;
                int fd = socket(AF_UNIX, SOCK_STREAM, 0);
                if (fd < 0) return -1;
                memset(&addr, 0, sizeof(addr));
                addr.sun_family = AF_UNIX;
                strncpy(addr.sun_path, (char*)$a_path, sizeof(addr.sun_path)-1);
                if (connect(fd, (struct sockaddr*)&addr, sizeof(addr)) < 0) {
                    close(fd);
                    return -1;
                }
                return fd;
            ]"
        end

    c_socket_read (a_fd: INTEGER; a_buffer: POINTER; a_size: INTEGER): INTEGER
        external
            "C inline use <unistd.h>"
        alias
            "return (EIF_INTEGER)read((int)$a_fd, $a_buffer, (size_t)$a_size);"
        end

    c_socket_write (a_fd: INTEGER; a_buffer: POINTER; a_size: INTEGER): INTEGER
        external
            "C inline use <unistd.h>"
        alias
            "return (EIF_INTEGER)write((int)$a_fd, $a_buffer, (size_t)$a_size);"
        end

    c_socket_close (a_fd: INTEGER): INTEGER
        external
            "C inline use <unistd.h>"
        alias
            "return (EIF_INTEGER)close((int)$a_fd);"
        end

end
```

**Usage in simple_docker**:
```eiffel
class DOCKER_CLIENT

feature {NONE} -- Initialization

    make
            -- Connect to local Docker Engine.
        do
            connection := (create {SIMPLE_IPC}).docker_connection
            connection.connect
        ensure
            connected: connection.is_connected
        end

feature -- API

    get_version: DOCKER_VERSION
            -- Get Docker Engine version.
        do
            connection.send ("GET /v1.51/version HTTP/1.1%R%NHost: localhost%R%N%R%N")
            Result := parse_version (connection.receive (4096))
        end

feature {NONE} -- Implementation

    connection: IPC_CONNECTION

end
```

### 14. Registry Authentication (Private Registries)

Docker registry authentication uses OAuth 2.0 bearer tokens:

**Flow**:
```
1. Client: GET /v2/my-repo/manifests/latest
2. Registry: 401 Unauthorized
   WWW-Authenticate: Bearer realm="https://auth.docker.io/token",service="registry.docker.io"
3. Client: GET https://auth.docker.io/token?service=registry.docker.io&scope=repository:my-repo:pull
   Authorization: Basic base64(username:password)
4. Auth Server: {"token": "eyJ..."}
5. Client: GET /v2/my-repo/manifests/latest
   Authorization: Bearer eyJ...
6. Registry: 200 OK
```

**Cloud Provider Specifics**:

| Provider | Auth Endpoint | Token Validity |
|----------|---------------|----------------|
| Docker Hub | auth.docker.io | Session-based |
| GitHub Container Registry | ghcr.io | PAT or GITHUB_TOKEN |
| AWS ECR | `aws ecr get-login-password` | 12 hours |
| Azure ACR | `az acr login` | 3 hours |

**simple_docker Implementation**:
```eiffel
class DOCKER_REGISTRY_AUTH

feature -- Authentication

    login (a_registry, a_username, a_password: STRING)
            -- Authenticate with registry.
        do
            -- Store credentials for this registry
            credentials.put (create {REGISTRY_CREDENTIAL}.make (a_username, a_password), a_registry)
        end

    get_token (a_registry, a_scope: STRING): STRING
            -- Get bearer token for scope.
        local
            l_cred: REGISTRY_CREDENTIAL
            l_http: SIMPLE_HTTP
        do
            l_cred := credentials.item (a_registry)
            create l_http.make
            l_http.set_basic_auth (l_cred.username, l_cred.password)
            Result := l_http.get (auth_url (a_registry, a_scope))
            -- Parse token from JSON response
        end

feature {NONE} -- Implementation

    credentials: HASH_TABLE [REGISTRY_CREDENTIAL, STRING]

end
```

---

## Research Sources

### SDK Documentation
- [Docker SDK Documentation](https://docs.docker.com/reference/api/engine/sdk/)
- [Docker Engine API v1.44](https://docs.docker.com/reference/api/engine/version/v1.44/)
- [libdocker C SDK](https://github.com/danielsuo/libdocker)
- [Docker SDK for Python](https://docker-py.readthedocs.io/)
- [Kubernetes C Client](https://github.com/kubernetes-client/c)
- [Compose SDK (Go)](https://docs.docker.com/compose/compose-sdk/)

### Existing Eiffel Docker Work
- [eiffel-docker/eiffel](https://github.com/eiffel-docker/eiffel)

### Alternative SDKs Studied
- [shiplift (Rust)](https://github.com/softprops/shiplift)
- [bollard (Rust)](https://github.com/fussybeaver/bollard)
- [dcrx Dockerfile Builder](https://pypi.org/project/dcrx/)

### Developer Pain Point Research
- [Docker 2024 State of Application Development Survey](https://www.docker.com/blog/docker-2024-state-of-application-development-report/)
- [docker-py GitHub Issues](https://github.com/docker/docker-py/issues) - API hang patterns, version compatibility
- [Docker Community Forums](https://forums.docker.com/) - Common developer frustrations

### Extended Research (Authentication, Streaming, IPC)
- [Protect the Docker daemon socket](https://docs.docker.com/engine/security/protect-access/) - TLS configuration
- [Configure remote access for Docker daemon](https://docs.docker.com/engine/daemon/remote-access/) - TCP setup
- [How to Secure Docker's TCP Socket With TLS](https://www.howtogeek.com/devops/how-to-secure-dockers-tcp-socket-with-tls/)
- [Docker Container Lifecycle States](https://www.baeldung.com/ops/docker-container-states) - State machine
- [Docker Container Lifecycle](https://last9.io/blog/docker-container-lifecycle/) - State transitions
- [Registry Token Authentication Specification](https://distribution.github.io/distribution/spec/auth/token/) - OAuth flow
- [Docker Engine API Examples](https://docs.docker.com/reference/api/engine/sdk/examples/) - API usage patterns
- [Build context | Docker Docs](https://docs.docker.com/build/concepts/context/) - Tar archive requirements

### ISE Eiffel Reference (study only, don't depend on)
- `$ISE_LIBRARY/library/wel/support/wel_pipe.e` - Windows named pipe patterns
- `$ISE_LIBRARY/library/net/local/socket/unix_stream_socket.e` - Unix socket patterns
