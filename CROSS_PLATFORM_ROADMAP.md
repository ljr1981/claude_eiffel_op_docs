# Simple Eiffel Cross-Platform Roadmap

**Date**: December 2025
**Purpose**: Strategy for Windows/Linux/macOS support

---

## Executive Summary

Simple Eiffel can achieve cross-platform support through:
1. **Immediate**: WSL2 for Linux testing (with caveats)
2. **Short-term**: Docker image with EiffelStudio 25.02
3. **Medium-term**: Platform abstraction layer using Gobo
4. **Long-term**: Native macOS testing (Mac Mini arriving)

---

## Current State Analysis

### Platform-Specific Libraries (11 libraries need work)

| Library | Windows APIs | Cross-Platform Effort |
|---------|--------------|----------------------|
| **simple_win32_api** | Facade for Win32 | Full rewrite as simple_platform |
| **simple_registry** | RegOpenKeyExA, etc. | No Unix equivalent - use config files |
| **simple_clipboard** | OpenClipboard, etc. | X11/Wayland + macOS NSPasteboard |
| **simple_console** | SetConsoleTextAttribute | ANSI escape codes for Unix |
| **simple_env** | GetEnvironmentVariableA | POSIX getenv/setenv (easy) |
| **simple_system** | GetComputerNameA, etc. | POSIX gethostname, sysconf |
| **simple_process** | CreateProcess | fork/exec, popen |
| **simple_mmap** | CreateFileMapping | POSIX mmap |
| **simple_watcher** | ReadDirectoryChangesW | inotify/FSEvents/kqueue |
| **simple_ipc** | CreateNamedPipe | Unix sockets (stub exists) |
| **simple_platform_api** | Win32 facade | Conditional dispatch |

### Platform-Agnostic Libraries (60+ libraries)

These work everywhere without changes:
- All data format libraries (json, xml, yaml, toml, csv, markdown)
- All crypto libraries (hash, encryption, jwt, base64)
- All math libraries (decimal, fraction, math, uuid)
- Database (sql via SQLite), HTTP, Web, WebSocket
- Most utility libraries

---

## Strategy 1: WSL2 (Immediate)

### Pros
- No additional hardware
- Full Linux kernel in Windows
- GUI support via WSLg
- EiffelStudio available via Ubuntu PPA

### Cons
- **10x slower** accessing Windows files (`/mnt/d/prod`)
- Best performance requires copying to WSL filesystem

### Setup
```bash
wsl --install -d Ubuntu
wsl --update
sudo add-apt-repository ppa:eiffelstudio-team/ppa
sudo apt-get install eiffelstudio libgtk-3.0-dev libxtst-dev
```

### Recommended Workflow
```bash
# Copy library to WSL filesystem for testing
cp -r /mnt/d/prod/simple_json ~/eiffel/
cd ~/eiffel/simple_json
ec -batch -config simple_json.ecf -target simple_json_tests -c_compile
./EIFGENs/simple_json_tests/W_code/simple_json
```

**Verdict**: Good for occasional Linux verification, not primary development.

---

## Strategy 2: Docker (Short-term)

### Current Landscape

| Image | Version | Status |
|-------|---------|--------|
| eiffel/eiffel:latest | 23.09 | Best option |
| eiffel/eiffel:dev | Nightly | Bleeding edge |
| mmonga/docker-eiffel | 18.11 | GUI via VNC |

**Gap**: No EiffelStudio 25.02 Docker image exists.

### Action: Create Simple Eiffel Docker Image

```dockerfile
FROM debian:bookworm

# Install dependencies
RUN apt-get update && apt-get install -y \
    libgtk-3-dev libxtst-dev libssl-dev \
    build-essential gcc wget git

# Install EiffelStudio 25.02
RUN wget https://ftp.eiffel.com/pub/download/25.02/... -O /tmp/eiffel.tar.bz2
RUN cd /opt && tar xvfj /tmp/eiffel.tar.bz2

ENV ISE_EIFFEL=/opt/Eiffel_25.02
ENV ISE_PLATFORM=linux-x86-64
ENV PATH=$PATH:$ISE_EIFFEL/studio/spec/$ISE_PLATFORM/bin

# Install Simple Eiffel
RUN git clone https://github.com/simple-eiffel/simple_json.git /opt/simple_eiffel/simple_json
# ... repeat for all libraries

ENV SIMPLE_EIFFEL=/opt/simple_eiffel
```

### CI/CD Integration
```yaml
# .github/workflows/test.yml
jobs:
  linux-tests:
    runs-on: ubuntu-latest
    container: simple-eiffel/eiffelstudio:25.02
    steps:
      - uses: actions/checkout@v4
      - run: ec -batch -config lib.ecf -c_compile
      - run: ./EIFGENs/*/W_code/*.exe
```

---

## Strategy 3: Platform Abstraction (Medium-term)

### Leverage Gobo Eiffel

Gobo provides cross-platform abstractions:

| Gobo Class | Purpose | Simple Eiffel Use |
|------------|---------|-------------------|
| KL_OPERATING_SYSTEM | Platform detection | Replace {PLATFORM}.is_windows |
| KL_FILE_SYSTEM | File operations | Cross-platform file handling |
| KL_SHELL_COMMAND | Process execution | Replace CreateProcess |
| KL_EXECUTION_ENVIRONMENT | Env vars | Replace Win32 env APIs |
| KL_DIRECTORY | Directory ops | Recursive create/delete |

### Architecture Pattern (from simple_ipc)

```eiffel
deferred class PLATFORM_SERVICE

feature
    execute_command (cmd: STRING): INTEGER
        deferred
        end
end

class WINDOWS_PLATFORM inherit PLATFORM_SERVICE
feature
    execute_command (cmd: STRING): INTEGER
        -- Use CreateProcess
    end
end

class POSIX_PLATFORM inherit PLATFORM_SERVICE
feature
    execute_command (cmd: STRING): INTEGER
        -- Use fork/exec via Gobo KL_SHELL_COMMAND
    end
end

class SIMPLE_PLATFORM
feature
    service: PLATFORM_SERVICE
        do
            if {KL_OPERATING_SYSTEM}.is_windows then
                create {WINDOWS_PLATFORM} Result
            else
                create {POSIX_PLATFORM} Result
            end
        end
end
```

### Priority Order

1. **simple_env** (1-2 days) - Easiest, uses POSIX getenv/setenv
2. **simple_console** (2-3 days) - ANSI escape codes
3. **simple_process** (3-5 days) - fork/exec via Gobo
4. **simple_ipc** (3-5 days) - Unix socket stub ready
5. **simple_mmap** (2-3 days) - POSIX mmap
6. **simple_watcher** (5-7 days) - inotify/FSEvents
7. **simple_clipboard** (3-5 days) - X11/macOS
8. **simple_system** (3-5 days) - POSIX equivalents

---

## Strategy 4: macOS (Long-term)

### EiffelStudio macOS Support
- Intel (x86-64): Full support
- Apple Silicon (M1/M2/M3): Supported since 22.12
- Set `ISE_PLATFORM=macosx-armv6`

### Mac Mini Setup (Friday)
1. Install EiffelStudio from download or MacPorts
2. Clone simple_* repositories
3. Set `SIMPLE_EIFFEL` environment variable
4. Test compilation across all libraries

### macOS-Specific Considerations
- Clipboard: NSPasteboard API
- File watching: FSEvents API
- Process: fork/exec (POSIX)
- Paths: Forward slashes (same as Linux)

---

## Gotchas & Known Issues

### WSL2
- `/mnt/` filesystem is 10x slower - don't use for primary dev
- GTK applications may lack window decorations (cosmetic)

### Docker
- No official EiffelStudio 25.02 image yet
- GUI requires VNC setup (mmonga image)

### EiffelStudio
- GPL version is 12 months behind commercial
- .NET/SCOOP not supported together
- Some ARM issues with Clang optimizer (addressed in 25.02)

### Simple Eiffel
- Hardcoded `D:\prod` paths in tests/docs (fixed now with SIMPLE_EIFFEL)
- Win32 inline C in 11 libraries
- simple_pdf has hardcoded Windows tool paths

---

## Recommended Roadmap

### Phase 1: Testing Infrastructure (Week 1-2)
- [ ] Set up WSL2 with EiffelStudio
- [ ] Create Docker image with EiffelStudio 25.02
- [ ] Add CI/CD workflow for Linux testing

### Phase 2: Quick Wins (Week 3-4)
- [ ] Cross-platform simple_env
- [ ] Cross-platform simple_console
- [ ] Update simple_pdf with Linux/macOS tool paths

### Phase 3: Core Platform Layer (Week 5-8)
- [ ] Add Gobo as ecosystem dependency
- [ ] Create simple_platform abstraction
- [ ] Cross-platform simple_process
- [ ] Cross-platform simple_ipc

### Phase 4: Advanced Features (Week 9-12)
- [ ] Cross-platform simple_mmap
- [ ] Cross-platform simple_watcher
- [ ] Cross-platform simple_clipboard
- [ ] Cross-platform simple_system

### Phase 5: macOS Validation (Ongoing)
- [ ] Test all libraries on Mac Mini
- [ ] Fix any macOS-specific issues
- [ ] Document macOS setup

---

## Success Metrics

- [ ] 60+ platform-agnostic libraries compile on Linux
- [ ] 60+ platform-agnostic libraries compile on macOS
- [ ] CI/CD runs tests on both Windows and Linux
- [ ] Docker image published to Docker Hub
- [ ] All 11 platform-specific libraries have cross-platform support

---

## Resources

- [EiffelStudio Downloads](https://www.eiffel.com/eiffelstudio/)
- [Gobo Eiffel Project](https://github.com/gobo-eiffel/gobo)
- [eiffel-docker/eiffel](https://github.com/eiffel-docker/eiffel)
- [Ubuntu PPA](https://launchpad.net/~eiffelstudio-team/+archive/ubuntu/ppa)
- [WSLg](https://github.com/microsoft/wslg)
