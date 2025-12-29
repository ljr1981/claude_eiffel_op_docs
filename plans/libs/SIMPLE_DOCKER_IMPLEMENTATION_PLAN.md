# simple_docker Implementation Plan

**Created:** 2025-12-15
**Status:** APPROVED
**Oracle Reference:** Query "docker" or "ipc"

---

## Executive Summary

This plan delivers **simple_docker** - a programmatic Docker container management library for Eiffel. It includes creating the prerequisite **simple_ipc** library first, then building simple_docker on top of it.

**Deliverables:**
1. `simple_ipc` - Cross-platform IPC library (named pipes + Unix sockets)
2. `simple_docker` - Docker Engine API client library
3. Full ecosystem integration (README, docs, tests, GitHub repos, env vars)

**Research Completed:** 14 vectors in `D:\prod\reference_docs\analyses\SIMPLE_DOCKER_VISION.md`

---

## Part 1: simple_ipc Library

### Overview

Cross-platform IPC using inline C (inline C pattern). NO dependency on WEL (Windows-only).

**Note:** simple_ipc already exists at `D:\prod\simple_ipc` with Windows implementation. This plan restructures it for cross-platform support.

### Classes

| Class | Purpose |
|-------|---------|
| `IPC_CONNECTION` | Deferred base class with DBC contracts |
| `NAMED_PIPE_CONNECTION` | Windows named pipes (refactor from current) |
| `UNIX_SOCKET_CONNECTION` | Unix domain sockets (new) |
| `SIMPLE_IPC` | Facade/factory for platform detection |

### Files to Create/Modify

```
D:\prod\simple_ipc\
├── src\
│   ├── simple_ipc.e              [REPLACE - becomes facade]
│   ├── ipc_connection.e          [NEW - deferred base]
│   ├── named_pipe_connection.e   [NEW - from current simple_ipc.e]
│   └── unix_socket_connection.e  [NEW - Unix implementation]
├── Clib\
│   ├── simple_ipc.h              [EXISTS]
│   ├── simple_ipc.c              [EXISTS]
│   ├── simple_ipc_unix.h         [NEW]
│   └── simple_ipc_unix.c         [NEW]
├── testing\
│   ├── lib_tests.e               [MODIFY - expand tests]
│   └── test_app.e                [MODIFY - new test cases]
├── docs\
│   └── index.html                [CREATE]
├── simple_ipc.ecf                [MODIFY - platform conditions]
├── README.md                     [UPDATE]
├── CHANGELOG.md                  [UPDATE - v2.0.0]
└── package.json                  [UPDATE]
```

### Implementation Phases

**Phase 1: Core (Windows Focus)**
- Create `IPC_CONNECTION` deferred base class
- Refactor current `SIMPLE_IPC` to `NAMED_PIPE_CONNECTION`
- Create `SIMPLE_IPC` facade with platform detection
- Create stub `UNIX_SOCKET_CONNECTION`
- Update ECF with platform conditions

**Phase 2: Unix Support**
- Implement `simple_ipc_unix.c/.h` with inline C
- Complete `UNIX_SOCKET_CONNECTION`
- Test on Linux/macOS (future)

**Phase 3-6: Performance, Docs, Tests, Hardening**
- Buffer optimization, connection caching
- IUARC documentation
- Comprehensive test suite
- Security hardening

---

## Part 2: simple_docker Library

### Overview

Docker Engine API client communicating via simple_ipc.

**Dependencies:**
- simple_ipc (IPC transport)
- simple_json (JSON parsing)
- simple_archive (tar for build context)
- simple_file (file operations)

### Classes

| Class | Purpose | Priority |
|-------|---------|----------|
| `DOCKER_CLIENT` | Main facade - all Docker operations | P1 |
| `DOCKER_CONTAINER` | Container representation with state | P1 |
| `DOCKER_IMAGE` | Image representation | P1 |
| `CONTAINER_SPEC` | Container config with fluent API + DBC | P1 |
| `CONTAINER_STATE` | State constants | P1 |
| `DOCKER_ERROR` | Error handling | P1 |
| `DOCKERFILE_BUILDER` | Fluent Dockerfile generation | P2 |
| `DOCKER_NETWORK` | Network operations | P2 |
| `DOCKER_VOLUME` | Volume operations | P2 |
| `DOCKER_VERSION` | Version info | P2 |
| `COMPOSE_BUILDER` | docker-compose.yaml generation | P3 |
| `DOCKER_REGISTRY_AUTH` | Private registry auth | P3 |
| `EIFFEL_CONTAINER_TEMPLATES` | Eiffel-specific templates | P3 |

### Files to Create

```
D:\prod\simple_docker\
├── src\
│   ├── docker_client.e
│   ├── docker_container.e
│   ├── docker_image.e
│   ├── docker_network.e
│   ├── docker_volume.e
│   ├── docker_version.e
│   ├── docker_error.e
│   ├── docker_response.e
│   ├── docker_registry_auth.e
│   ├── container_spec.e
│   ├── container_state.e
│   ├── dockerfile_builder.e
│   ├── compose_builder.e
│   └── templates\
│       └── eiffel_container_templates.e
├── testing\
│   ├── lib_tests.e
│   └── test_app.e
├── docs\
│   ├── index.html
│   ├── user-guide.html
│   ├── api-reference.html
│   ├── architecture.html
│   └── cookbook.html
├── simple_docker.ecf
├── README.md
├── CHANGELOG.md
└── package.json
```

### ECF Configuration

```xml
<!-- Dependencies -->
<library name="simple_ipc" location="$SIMPLE_IPC\simple_ipc.ecf"/>
<library name="simple_json" location="$SIMPLE_JSON\simple_json.ecf"/>
<library name="simple_archive" location="$SIMPLE_ARCHIVE\simple_archive.ecf"/>
<library name="simple_file" location="$SIMPLE_FILE\simple_file.ecf"/>
```

### Implementation Phases

**Phase 1: Core Functionality (MVP)**
- DOCKER_CLIENT with named pipe connection
- Container: create, start, stop, remove, logs
- Image: pull, list, exists
- CONTAINER_SPEC basic config
- CONTAINER_STATE, DOCKER_ERROR

**Phase 2: Expanded Features**
- Full container lifecycle (pause, unpause, restart, kill, exec, wait)
- Port mapping, volume mounts, environment vars
- DOCKERFILE_BUILDER fluent API
- Network and volume operations

**Phase 3: Performance Optimization**
- Streaming log support
- Connection reuse
- Efficient tar context creation
- Timeout handling

**Phase 4: API Documentation**
- README.md with quick start
- IUARC 5-document standard
- Code comments

**Phase 5: Test Coverage**
- LIB_TESTS comprehensive suite
- Requires Docker Desktop running

**Phase 6: Production Hardening**
- COMPOSE_BUILDER
- EIFFEL_CONTAINER_TEMPLATES
- DOCKER_REGISTRY_AUTH
- SCOOP verification

---

## Part 3: Ecosystem Integration

### GitHub Repository Creation

```bash
# Create repos in simple-eiffel organization
gh repo create simple-eiffel/simple_ipc --public --description "Cross-platform IPC for Eiffel: named pipes (Windows) + Unix sockets"
gh repo create simple-eiffel/simple_docker --public --description "Docker container management for Eiffel: build, run, manage containers programmatically"
```

### Environment Variables

Add to `D:\prod\env_setup.sh`:
```bash
export SIMPLE_IPC=/d/prod/simple_ipc
export SIMPLE_DOCKER=/d/prod/simple_docker
```

Windows persistent (via setx):
```cmd
setx SIMPLE_IPC D:\prod\simple_ipc
setx SIMPLE_DOCKER D:\prod\simple_docker
```

### Documentation Site Updates

Update `https://simple-eiffel.github.io/`:
- Add simple_ipc to library list
- Add simple_docker to library list
- Link to individual docs pages

### Oracle Registration

```bash
oracle-cli.exe log info simple_ipc "Library created: Cross-platform IPC with inline C"
oracle-cli.exe log info simple_docker "Library created: Docker container management"
```

---

## Part 4: Detailed Task Breakdown

### Week 1: simple_ipc Restructure

| # | Task | Files | Tests |
|---|------|-------|-------|
| 1 | Create IPC_CONNECTION deferred base | src/ipc_connection.e | - |
| 2 | Refactor SIMPLE_IPC to NAMED_PIPE_CONNECTION | src/named_pipe_connection.e | 5 existing |
| 3 | Create SIMPLE_IPC facade | src/simple_ipc.e | 2 new |
| 4 | Create UNIX_SOCKET_CONNECTION stub | src/unix_socket_connection.e | - |
| 5 | Update ECF with platform conditions | simple_ipc.ecf | - |
| 6 | Expand test suite | testing/lib_tests.e | 15 total |
| 7 | Update README, CHANGELOG, package.json | *.md, package.json | - |
| 8 | Create docs/index.html | docs/index.html | - |

### Week 2: simple_docker Core

| # | Task | Files | Tests |
|---|------|-------|-------|
| 1 | Create directory structure | simple_docker/* | - |
| 2 | Generate ECF with UUID | simple_docker.ecf | - |
| 3 | Implement DOCKER_CLIENT connection | src/docker_client.e | 3 |
| 4 | Implement CONTAINER_STATE | src/container_state.e | - |
| 5 | Implement DOCKER_ERROR | src/docker_error.e | 2 |
| 6 | Implement CONTAINER_SPEC | src/container_spec.e | 3 |
| 7 | Implement DOCKER_CONTAINER | src/docker_container.e | 5 |
| 8 | Implement DOCKER_IMAGE | src/docker_image.e | 3 |
| 9 | Container operations: create, start, stop | docker_client.e | 5 |
| 10 | Image operations: pull, list | docker_client.e | 3 |

### Week 3: simple_docker Expanded

| # | Task | Files | Tests |
|---|------|-------|-------|
| 1 | Container: pause, unpause, restart, kill | docker_client.e | 4 |
| 2 | Container: exec, logs, wait | docker_client.e | 3 |
| 3 | DOCKERFILE_BUILDER | src/dockerfile_builder.e | 5 |
| 4 | DOCKER_NETWORK | src/docker_network.e | 2 |
| 5 | DOCKER_VOLUME | src/docker_volume.e | 2 |
| 6 | Network/volume client operations | docker_client.e | 4 |

### Week 4: Polish & Hardening

| # | Task | Files | Tests |
|---|------|-------|-------|
| 1 | COMPOSE_BUILDER | src/compose_builder.e | 3 |
| 2 | DOCKER_REGISTRY_AUTH | src/docker_registry_auth.e | 2 |
| 3 | EIFFEL_CONTAINER_TEMPLATES | src/templates/*.e | 2 |
| 4 | Streaming log support | docker_client.e | 1 |
| 5 | IUARC documentation | docs/*.html | - |
| 6 | README, CHANGELOG, package.json | *.md, package.json | - |
| 7 | Final test pass | testing/*.e | All |
| 8 | GitHub repos + push | - | - |
| 9 | Oracle registration | - | - |

---

## Part 5: Key Technical Decisions

### 1. Named Pipe Connection (Windows)

```eiffel
-- Connect to Docker Engine
create connection.make_client ("docker_engine")
-- Sends HTTP/1.1 over named pipe
connection.write_string ("GET /v1.51/version HTTP/1.1%R%NHost: localhost%R%N%R%N")
response := connection.read_string (65536)
```

### 2. Docker API Version Negotiation

```eiffel
-- On connect, negotiate API version
make
    do
        create connection.make_client ("docker_engine")
        api_version := negotiate_version  -- Start with 1.51, fall back if needed
    end
```

### 3. Error Handling Pattern

```eiffel
-- All operations set last_error on failure
run_container (a_spec: CONTAINER_SPEC): DOCKER_CONTAINER
    do
        Result := do_create_container (a_spec)
        if Result /= Void then
            do_start_container (Result)
        end
    ensure
        result_or_error: Result /= Void xor has_error
    end
```

### 4. Build Context with simple_archive

```eiffel
-- Create tar, POST to /build
build_image (a_context_dir: STRING; a_tag: STRING)
    local
        l_archive: SIMPLE_ARCHIVE
    do
        create l_archive.make
        l_archive.create_archive_from_directory (tar_path, a_context_dir)
        send_build_request (tar_path, a_tag)
    end
```

---

## Part 6: Test Requirements

### Prerequisites
- Docker Desktop running
- Images available: `alpine:latest`, `hello-world`, `nginx:alpine`

### Test Categories

| Category | Count | Coverage |
|----------|-------|----------|
| Connection | 3 | Ping, version, connect |
| Images | 5 | Pull, list, exists, build, remove |
| Containers | 12 | Full lifecycle |
| Networks | 3 | Create, connect, remove |
| Volumes | 2 | Create, remove |
| Dockerfile | 3 | Basic, multi-stage, validation |
| Spec | 3 | Fluent API, JSON, validation |
| **Total** | **31** | |

---

## Part 7: Success Criteria

| Metric | Target |
|--------|--------|
| simple_ipc compiles | Windows + Linux stubs |
| simple_docker compiles | All classes |
| Tests passing | 31+ tests |
| Documentation | IUARC 5-doc standard |
| GitHub repos | Created and pushed |
| Ecosystem integration | Env vars, oracle registered |

---

## Critical Files Reference

| File | Purpose |
|------|---------|
| `D:\prod\reference_docs\analyses\SIMPLE_DOCKER_VISION.md` | Research findings |
| `D:\prod\simple_ipc\src\simple_ipc.e` | Current Windows IPC |
| `D:\prod\simple_ipc\Clib\simple_ipc.c` | Win32 API patterns |
| `D:\prod\simple_json\src\core\simple_json.e` | JSON parsing |
| `D:\prod\simple_archive\src\simple_archive.e` | Tar archive |
| `D:\prod\simple_testing\src\test_set_base.e` | Test base class |

---

## Execution Command

After plan approval:
```bash
# Start with simple_ipc restructure
cd /d/prod/simple_ipc
# Then simple_docker creation
mkdir /d/prod/simple_docker
```
