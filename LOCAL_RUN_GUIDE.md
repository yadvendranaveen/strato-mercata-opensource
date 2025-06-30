# Running STRATO Locally on Apple Silicon

## Prerequisites

- ✅ Docker Desktop for Mac (with Apple Silicon support)
- ✅ Git
- ✅ Your GitHub Actions builds completed successfully

## Method 1: Using GitHub Actions Builds (Recommended)

### Step 1: Wait for Builds to Complete
1. Go to your fork: `https://github.com/yadvendranaveen/strato-mercata-opensource`
2. Click the "Actions" tab
3. Wait for the build to complete (50-75 minutes for first build)

### Step 2: Generate Docker Compose Files
Once the build completes, the workflow will generate docker-compose files. You can also generate them locally:

```bash
# Generate docker-compose files
make docker-compose
```

This creates:
- `docker-compose.yml` - Main platform
- `docker-compose.vault.yml` - Vault services
- `docker-compose.identity.yml` - Identity services
- `docker-compose.highway.yml` - Highway services

### Step 3: Set Up Environment Variables
Create a `.env` file with required configuration:

```bash
# Copy the example environment file
cp .env.example .env
```

Or create your own `.env` file with minimal settings:

```env
# Basic STRATO Configuration
NODE_HOST=localhost
NODE_NAME=local-strato
STRATO_HOSTNAME=localhost
STRATO_PORT_API=3000
STRATO_PORT_VAULT_PROXY=8013

# Database
postgres_password=your_password_here
postgres_user=postgres

# OAuth (if using)
OAUTH_CLIENT_ID=your_client_id
OAUTH_CLIENT_SECRET=your_client_secret

# Optional: SSL
ssl=false
```

### Step 4: Run STRATO

#### Option A: Full Platform
```bash
# Start all services
docker-compose up -d

# View logs
docker-compose logs -f

# Stop services
docker-compose down
```

#### Option B: Individual Services
```bash
# Run just STRATO core
docker-compose up strato postgres redis kafka -d

# Run with Vault
docker-compose -f docker-compose.vault.yml up -d

# Run with Identity Provider
docker-compose -f docker-compose.identity.yml up -d
```

## Method 2: Using Pre-built Images (Future)

Once we set up image publishing in GitHub Actions:

```bash
# Pull pre-built images
docker pull ghcr.io/yadvendranaveen/strato:latest
docker pull ghcr.io/yadvendranaveen/highway:latest
docker pull ghcr.io/yadvendranaveen/vault-wrapper:latest
docker pull ghcr.io/yadvendranaveen/identity-provider:latest

# Use with docker-compose
docker-compose up
```

## Method 3: Minimal Local Setup

For development/testing, you can run just the core components:

```bash
# Start minimal services
docker-compose up strato postgres redis kafka -d

# Check if STRATO is running
curl http://localhost:3000/strato/v2.3/_ping
```

## Troubleshooting

### Common Issues:

1. **Port Conflicts**:
   ```bash
   # Check what's using the ports
   lsof -i :3000
   lsof -i :5432
   lsof -i :6379
   ```

2. **Docker Resources**:
   - Increase Docker memory limit (8GB+ recommended)
   - Increase CPU allocation (4+ cores recommended)

3. **Database Issues**:
   ```bash
   # Reset database
   docker-compose down -v
   docker-compose up postgres -d
   ```

4. **Permission Issues**:
   ```bash
   # Fix volume permissions
   sudo chown -R $USER:$USER .docker-work/
   ```

### Health Checks:

```bash
# Check STRATO API
curl http://localhost:3000/strato/v2.3/_ping

# Check PostgreSQL
docker-compose exec postgres pg_isready

# Check Redis
docker-compose exec redis redis-cli ping

# Check Kafka
docker-compose exec kafka kafka-topics --list --bootstrap-server localhost:9092
```

## Development Workflow

1. **Make changes** to your local code
2. **Push to your fork** to trigger GitHub Actions builds
3. **Wait for builds** to complete
4. **Pull new images** and restart services
5. **Test locally** with docker-compose

## Useful Commands

```bash
# View all running containers
docker-compose ps

# View logs for specific service
docker-compose logs strato

# Restart specific service
docker-compose restart strato

# Clean up everything
docker-compose down -v --remove-orphans

# Build specific service locally (if needed)
docker-compose build strato
```

## Next Steps

1. **Monitor the GitHub Actions builds** in your fork
2. **Set up environment variables** for your use case
3. **Start with minimal services** and add more as needed
4. **Contribute back** any improvements to the original repository

---

*This guide assumes you have successfully set up GitHub Actions builds as described in the Apple Silicon Build Guide.* 