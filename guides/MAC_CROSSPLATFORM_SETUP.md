# Mac Cross-Platform Setup Guide

Reference for setting up the Mac Mini for Simple Eiffel cross-platform development.

## Network Setup (PC ↔ Mac)

### Ethernet Hub/Switch Connection

1. Connect both computers to the same ethernet switch/hub
2. Both will get IPs on the same subnet (likely 192.168.1.x)
3. Mac IP observed: 192.168.1.112
4. No special configuration needed - plug and play

### Quick Test (from Windows)

```bash
ping 192.168.1.112
```

### File Sharing Options (fastest to slowest)

| Method | Speed | Setup |
|--------|-------|-------|
| SMB/CIFS | ~100MB/s on gigabit | Mac: System Preferences → Sharing → File Sharing. Windows: `\\192.168.1.112` in Explorer |
| SCP/SFTP | ~80MB/s | Mac already has SSH. Use WinSCP or `scp` from Git Bash |
| rsync | ~80MB/s + delta sync | Best for syncing repos: `rsync -avz /d/prod/ user@192.168.1.112:~/simple_eiffel/` |

### Recommendation

Clone from GitHub on Mac rather than copying from Windows:
- Clean git history
- No Windows line-ending issues (CRLF → LF)
- Proper Unix permissions

## Mac Environment Setup

### Prerequisites

1. **Xcode Command Line Tools** (required for gcc):
   ```bash
   xcode-select --install
   ```

2. **EiffelStudio 25.02**:
   ```bash
   # Download from https://www.eiffel.com/eiffelstudio/download/
   # Extract to ~/Eiffel_25.02
   tar -xzf EiffelStudio-25.02-linux-x86-64.tar.gz -C ~/
   ```

3. **Environment Variables** (add to `~/.zshrc` or `~/.bashrc`):
   ```bash
   # EiffelStudio configuration (REQUIRED)
   export ISE_EIFFEL=$HOME/Eiffel_25.02
   export ISE_PLATFORM=macosx-x86-64   # or linux-x86-64 for Linux
   export ISE_LIBRARY=$ISE_EIFFEL
   export PATH=$ISE_EIFFEL/studio/spec/$ISE_PLATFORM/bin:$PATH

   # Simple Eiffel ecosystem
   export SIMPLE_EIFFEL=$HOME/simple_eiffel
   ```

   **CRITICAL**: `ISE_LIBRARY` must be set or compilation fails with:
   ```
   eif_langinfo.h: No such file or directory
   ```

## Clone Simple Eiffel Libraries

### Required for simple_notebook

```bash
mkdir -p ~/simple_eiffel && cd ~/simple_eiffel

git clone https://github.com/simple-eiffel/simple_notebook
git clone https://github.com/simple-eiffel/simple_process
git clone https://github.com/simple-eiffel/simple_json
git clone https://github.com/simple-eiffel/simple_file
git clone https://github.com/simple-eiffel/simple_datetime
git clone https://github.com/simple-eiffel/simple_testing
```

### Clone Script (all libraries)

A comprehensive clone script exists on the Mac:
```bash
# Already run - clones all simple_* repos from GitHub
~/clone_simple_eiffel.sh
```

## Build simple_notebook on Mac

### Step 1: Compile C Library for simple_process

```bash
cd ~/simple_eiffel/simple_process/Clib
gcc -c -fPIC -I. simple_process.c -o simple_process.o
```

### Step 2: Build Eiffel Notebook

```bash
cd ~/simple_eiffel/simple_notebook
ec -batch -config simple_notebook.ecf -target notebook_cli -c_compile
```

### Step 3: Run

```bash
./EIFGENs/notebook_cli/W_code/simple_notebook
```

## Troubleshooting

### "ec: command not found"

Environment not set. Source your profile:
```bash
source ~/.zshrc   # or ~/.bashrc
```

### "eif_langinfo.h: No such file or directory"

`ISE_LIBRARY` not set. Add to environment:
```bash
export ISE_LIBRARY=$ISE_EIFFEL
```

### Permission denied on executable

```bash
chmod +x ./EIFGENs/notebook_cli/W_code/simple_notebook
```

### C compilation errors

Ensure Xcode Command Line Tools are installed:
```bash
xcode-select --install
gcc --version  # Should show Apple clang
```

## Status

- [x] Mac Mini identified (192.168.1.112)
- [x] Clone script created and run
- [x] Xcode Command Line Tools installing
- [ ] Ethernet hub connection (pending hardware)
- [ ] EiffelStudio installed on Mac
- [ ] simple_notebook compiled on Mac
- [ ] Cross-platform tests verified

---

*Last updated: 2025-12-21*
