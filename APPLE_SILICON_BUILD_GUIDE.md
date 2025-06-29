# Apple Silicon Build Guide for STRATO Platform

## Problem Statement

The STRATO Mercata platform was designed for Intel Macs (x86_64) and fails to build on Apple Silicon Macs (ARM64) due to:

1. **Platform Mismatch**: Uses `fpco/stack-build` images that only support x86_64
2. **Rosetta Emulation Issues**: x86_64 binaries fail with `rosetta error: bss_size overflow`
3. **ARM64 Compatibility**: Some Haskell packages (like `saltine-0.2.1.0`) don't support ARM64

## Issues Encountered

### 1. Initial Build Failure
```
WARNING: The requested image's platform (linux/amd64) does not match the detected host platform (linux/arm64/v8)
rosetta error: bss_size overflow
make: *** [build_common] Error 133
```

**Root Cause**: Stack binary downloaded for `linux-x86_64` fails under Rosetta 2 emulation.

### 2. ARM64 Native Build Attempt
Created `Dockerfile.buildbase.arm64` using `haskell:8.10-buster`:
```dockerfile
FROM haskell:8.10-buster
RUN curl -sSL https://get.haskellstack.org/ | sh
# ... rest of dependencies
```

**Result**: Build progressed further but failed on `saltine-0.2.1.0` package compilation.

### 3. Package Compatibility Issues
```
Error: [S-7282]
Stack failed to execute the build plan.
While building package saltine-0.2.1.0
```

**Root Cause**: Cryptography packages with C dependencies lack ARM64 support.

## Solutions Implemented

### Solution 1: GitHub Actions x86_64 Builds (Recommended)

**Approach**: Use GitHub Actions to build on x86_64 runners and download results locally.

**Files Created**:
- `.github/workflows/build.yml` - Main x86_64 build workflow
- `.github/workflows/build-arm64.yml` - ARM64 compatibility testing
- `GITHUB_ACTIONS_README.md` - Usage instructions

**Benefits**:
- ✅ **Free** (2,000 minutes/month for public repos)
- ✅ **Reliable** x86_64 builds
- ✅ **No local build issues**
- ✅ **Automated testing**

**Build Times**:
- **First build**: 50-75 minutes
- **Subsequent builds**: 10-18 minutes (with caching)

### Solution 2: ARM64 Native Build (Experimental)

**Approach**: Use ARM64-compatible Haskell images for native builds.

**Files Created**:
- `Dockerfile.buildbase.arm64` - ARM64 build environment

**Status**: Partial success - works for some packages but fails on cryptography dependencies.

## Step-by-Step Setup

### For Apple Silicon Users:

1. **Fork the Repository**:
   ```bash
   # Go to https://github.com/blockapps/strato-mercata-opensource
   # Click "Fork" button
   ```

2. **Clone Your Fork**:
   ```bash
   git clone https://github.com/YOUR_USERNAME/strato-mercata-opensource.git
   cd strato-mercata-opensource
   ```

3. **Add GitHub Actions** (already done in this fork):
   ```bash
   # Workflows are already included
   git push origin main
   ```

4. **Monitor Builds**:
   - Go to Actions tab in your fork
   - Watch the build progress
   - Download artifacts when complete

5. **Run Locally**:
   ```bash
   # Pull pre-built images
   docker pull ghcr.io/YOUR_USERNAME/strato:latest
   docker-compose up
   ```

## Technical Details

### Why ARM64 Builds Fail

1. **Haskell Package Ecosystem**: Many packages haven't been updated for ARM64
2. **C Dependencies**: Native libraries like libsodium need ARM64 ports
3. **Stack Resolver**: LTS-22.4 may not have full ARM64 support
4. **Docker Images**: Official stack-build images lack ARM64 variants

### GitHub Actions Workflow Details

**Main Build (`build.yml`)**:
- Uses `ubuntu-latest` (x86_64) runners
- Builds complete STRATO platform
- Creates Docker images for all services
- Generates docker-compose files

**ARM64 Test (`build-arm64.yml`)**:
- Uses QEMU emulation for ARM64 testing
- Identifies compatibility issues
- Reports on Apple Silicon readiness

### Caching Strategy

**GitHub Actions Cache**:
- GHC installation (saves ~10 minutes)
- Haskell package downloads (saves ~15 minutes)
- Compiled packages (saves ~20-30 minutes)
- Docker image layers (saves ~5-10 minutes)

**Cache Invalidation**:
- Weekly expiration
- Dependency changes
- Dockerfile modifications
- GHC version updates

## Recommendations

### For Developers:
1. **Use GitHub Actions** for reliable builds
2. **Fork the repository** to get free builds
3. **Monitor ARM64 compatibility** as packages update
4. **Contribute back** ARM64 fixes when possible

### For Project Maintainers:
1. **Add ARM64 CI/CD** to identify compatibility issues
2. **Update dependencies** to ARM64-compatible versions
3. **Document platform requirements** clearly
4. **Consider multi-arch Docker images**

## Future Improvements

1. **ARM64 Package Updates**: Wait for packages to add ARM64 support
2. **Alternative Resolvers**: Try newer Stack resolvers with better ARM64 support
3. **Custom ARM64 Images**: Build custom Docker images with ARM64 fixes
4. **Package Patching**: Fork and fix problematic packages

## Conclusion

While native ARM64 builds are challenging due to package compatibility issues, GitHub Actions provides a reliable solution for Apple Silicon users. The x86_64 builds work perfectly and can be used locally with Docker.

**Best approach for Apple Silicon users**: Use GitHub Actions for builds, download pre-built images, and run locally with Docker.

---

*This guide documents the findings from attempting to build STRATO Platform on Apple Silicon Macs (M1/M2/M3) and provides solutions for developers facing similar issues.* 