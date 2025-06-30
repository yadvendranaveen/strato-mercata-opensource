# Modular Build Guide for Apple Silicon

## Strategy: Build & Test Individual Modules

You can build and test individual modules locally, then submit targeted changes. This approach lets you:

- ✅ **Test specific modules** on your Apple Silicon Mac
- ✅ **Avoid problematic packages** (like `saltine`)
- ✅ **Submit focused changes** to the original repository
- ✅ **Contribute incrementally** without full platform builds

## Buildable Modules (ARM64 Compatible)

### Core Modules (Likely to Work)
```bash
# Basic configuration and models
core/strato-conf
core/strato-model
core/strato-networks
core/strato-init

# API components
api/core
api/strato-api

# Utility libraries
libs/common-log
libs/format
libs/labeled-error
libs/type-lits
libs/nibblestring
```

### Tools (Good Candidates)
```bash
# Development tools
tools/blockapps-tools
tools/debugger-tools
tools/process-monitor
tools/x509-tools
```

### Libraries (Mostly Safe)
```bash
# Monad libraries
libs/composable-monads/composable-monads-base
libs/composable-monads/redis-monad
libs/composable-monads/sql-monad

# Other utilities
libs/clockwork
libs/ethereum-rlp
libs/json-rpc-server
libs/json-rpc-client
```

## How to Build Individual Modules

### Step 1: Build ARM64 Environment
```bash
# Build the ARM64 buildbase
docker build -f Dockerfile.buildbase.arm64 -t strato-buildbase:arm64 .
```

### Step 2: Build Specific Module
```bash
# Create working directory
mkdir -p .docker-work/usr/local/bin

# Build a specific module
docker run --rm -v "$PWD":/strato-platform -w /strato-platform/strato \
  strato-buildbase:arm64 \
  stack build --no-docker --fast core/strato-conf
```

### Step 3: Test the Module
```bash
# Run tests for the module
docker run --rm -v "$PWD":/strato-platform -w /strato-platform/strato \
  strato-buildbase:arm64 \
  stack test core/strato-conf
```

## Development Workflow

### 1. Choose Your Module
```bash
# Start with simple modules
MODULE="core/strato-conf"

# Or try API modules
MODULE="api/core"
```

### 2. Build and Test
```bash
# Build the module
docker run --rm -v "$PWD":/strato-platform -w /strato-platform/strato \
  strato-buildbase:arm64 \
  stack build --no-docker --fast "$MODULE"

# Run tests
docker run --rm -v "$PWD":/strato-platform -w /strato-platform/strato \
  strato-buildbase:arm64 \
  stack test "$MODULE"
```

### 3. Make Changes
```bash
# Edit files in the module
vim strato/core/strato-conf/src/...

# Rebuild to test changes
docker run --rm -v "$PWD":/strato-platform -w /strato-platform/strato \
  strato-buildbase:arm64 \
  stack build --no-docker --fast "$MODULE"
```

### 4. Submit Changes
```bash
# Commit your changes
git add strato/core/strato-conf/
git commit -m "Improve strato-conf module for Apple Silicon compatibility"

# Push to your fork
git push fork yadvendra:main

# Create pull request to original repo
```

## Modules to Avoid (ARM64 Issues)

### Problematic Modules
```bash
# These likely have ARM64 compatibility issues:
- Any module depending on saltine (cryptography)
- VM/EVM modules (may have C dependencies)
- Some blockchain-specific modules
```

### How to Check Dependencies
```bash
# Check what a module depends on
docker run --rm -v "$PWD":/strato-platform -w /strato-platform/strato \
  strato-buildbase:arm64 \
  stack list-dependencies core/strato-conf
```

## Example: Contributing a Module Fix

### Scenario: Fix strato-conf for Apple Silicon

1. **Identify the issue**:
   ```bash
   # Build the module
   docker run --rm -v "$PWD":/strato-platform -w /strato-platform/strato \
     strato-buildbase:arm64 \
     stack build --no-docker --fast core/strato-conf
   ```

2. **Fix the code**:
   ```bash
   # Edit the problematic file
   vim strato/core/strato-conf/src/SomeFile.hs
   ```

3. **Test your fix**:
   ```bash
   # Rebuild and test
   docker run --rm -v "$PWD":/strato-platform -w /strato-platform/strato \
     strato-buildbase:arm64 \
     stack test core/strato-conf
   ```

4. **Submit the change**:
   ```bash
   git add strato/core/strato-conf/
   git commit -m "Fix ARM64 compatibility in strato-conf module"
   git push fork yadvendra:main
   ```

5. **Create pull request** to the original repository

## Benefits of This Approach

### For You:
- ✅ **Immediate feedback** on your changes
- ✅ **No waiting** for full platform builds
- ✅ **Targeted contributions** to specific modules
- ✅ **Learning opportunity** about the codebase

### For the Project:
- ✅ **Incremental ARM64 support** module by module
- ✅ **Focused improvements** rather than big-bang changes
- ✅ **Community contributions** from Apple Silicon users
- ✅ **Better documentation** of ARM64 compatibility

## Quick Start Commands

```bash
# Build ARM64 environment
docker build -f Dockerfile.buildbase.arm64 -t strato-buildbase:arm64 .

# Try building a simple module
docker run --rm -v "$PWD":/strato-platform -w /strato-platform/strato \
  strato-buildbase:arm64 \
  stack build --no-docker --fast core/strato-conf

# If successful, try another
docker run --rm -v "$PWD":/strato-platform -w /strato-platform/strato \
  strato-buildbase:arm64 \
  stack build --no-docker --fast libs/common-log
```

## Next Steps

1. **Start with simple modules** like `core/strato-conf`
2. **Build and test** each module individually
3. **Identify issues** and fix them
4. **Submit pull requests** for working modules
5. **Document your findings** for other Apple Silicon users

This approach lets you contribute meaningfully while working within the constraints of your Apple Silicon Mac! 🚀 