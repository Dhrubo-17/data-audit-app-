# Data Audit Genius v3 - Deployment Guide

## Quick Start (Docker)

### Step 1: Configure Environment

```bash
cp .env.example .env
```

Edit `.env` and set:

```env
# REQUIRED: Set to a random 32+ character string
JWT_SECRET=your_random_secret_here_change_this

# Database (pre-configured for Docker)
DATABASE_URL=postgres://postgres:postgres@db:5432/data_audit_genius
DB_CLIENT=postgres

# AI (optional - app works without AI)
AI_PROVIDER=none
```

Generate JWT secret:
```bash
openssl rand -hex 32
```

### Step 2: Start Application

```bash
docker-compose up --build
```

Wait for: "Server running on port 5000"

### Step 3: Access Application

Open: http://localhost:5000

---

## Verification

### Register First User

```bash
curl -X POST http://localhost:5000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@example.com","password":"Admin123!"}'
```

Save the `token` from response.

### Upload Sample Data

```bash
TOKEN="paste_your_token_here"

curl -X POST http://localhost:5000/api/datasets/upload \
  -H "Authorization: Bearer $TOKEN" \
  -F "file=@data/sample_small.csv"
```

Save the dataset `id` from response.

### Analyze Data

```bash
DATASET_ID=1

curl -X POST http://localhost:5000/api/datasets/$DATASET_ID/analyze \
  -H "Authorization: Bearer $TOKEN"
```

### Clean Data

```bash
curl -X POST http://localhost:5000/api/datasets/$DATASET_ID/clean \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"actionIds":[]}'
```

### Download Cleaned File

```bash
curl -X GET http://localhost:5000/api/datasets/$DATASET_ID/export \
  -H "Authorization: Bearer $TOKEN" \
  --output cleaned.xlsx
```

---

## Production Deployment

### Environment Variables

Required:
- `JWT_SECRET` - Random 32+ char string
- `DATABASE_URL` - PostgreSQL connection string

Optional:
- `PORT` - Default: 5000
- `AI_PROVIDER` - Options: none, hf, local (Default: none)
- `MAX_UPLOAD_BYTES` - Default: 50MB
- `RATE_LIMIT_MAX` - Default: 60 requests/minute

### Docker Compose (Recommended)

```bash
# Start
docker-compose up -d

# View logs
docker-compose logs -f app

# Stop
docker-compose down

# Restart
docker-compose restart app
```

### Health Checks

```bash
# Basic
curl http://localhost:5000/health

# Full (includes database)
curl http://localhost:5000/healthz
```

Expected:
```json
{"status":"healthy","database":"connected","timestamp":"..."}
```

---

## Troubleshooting

### App won't start
- Check `.env` file exists and JWT_SECRET is set
- Check Docker is running
- Check ports 5000 and 5432 are available

### Database connection failed
```bash
docker-compose logs db
docker-compose restart db
```

### Cannot upload files
```bash
# Check uploads directory
ls -la uploads/

# Fix permissions
sudo chown -R 1001:1001 uploads/
```

---

## Backup & Restore

### Backup
```bash
docker-compose exec -T db pg_dump -U postgres data_audit_genius > backup.sql
tar -czf uploads-backup.tar.gz uploads/
```

### Restore
```bash
cat backup.sql | docker-compose exec -T db psql -U postgres data_audit_genius
tar -xzf uploads-backup.tar.gz
```

---

## Disabling AI

Set in `.env`:
```env
AI_PROVIDER=none
```

The app works fully without AI. All data validation is rule-based.

---

## Common Commands

```bash
# Start services
docker-compose up -d

# Stop services
docker-compose down

# View logs
docker-compose logs -f app

# Shell access
docker-compose exec app sh

# Database access
docker-compose exec db psql -U postgres data_audit_genius

# Check status
docker-compose ps

# Rebuild
docker-compose up --build
```

---

**All features work with AI disabled. The app is production-ready out of the box.**
