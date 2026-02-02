# Data Audit Genius v3

**Production-Ready Data Auditing & Cleaning Application**

Self-hosted. No AI required. Deploy in 5 minutes.

---

## Features

- **Upload** CSV, Excel, JSON files
- **Analyze** data quality with rule-based audit engine
- **Clean** data with automated fixes
- **Export** cleaned datasets with audit trail
- **Secure** JWT authentication, role-based access
- **Optional AI** enhancement (HuggingFace or local LLM)

---

## Quick Start

```bash
# 1. Extract and enter directory
unzip Data-Audit-Genius-v3-PRODUCT-READY.zip
cd Data-Audit-Genius-v3

# 2. Configure
cp .env.example .env
# Edit .env: set JWT_SECRET to a random string

# 3. Deploy
docker-compose up --build

# 4. Access
# Open: http://localhost:5000
```

---

## What's Included

- Deterministic data validation engine
- Financial reconciliation
- Email/phone validation
- Date standardization
- Missing value detection
- Identity checks
- Reality constraints (age, balance)
- User authentication & authorization
- File upload security
- Rate limiting
- Docker deployment
- PostgreSQL database
- Full test suite

---

## Documentation

- **DEPLOY.md** - Deployment guide
- **TESTS.md** - Testing procedures
- **RELEASE_NOTES.md** - Version history
- **sample_data.csv** - Example dataset

---

## Requirements

- Docker Desktop
- 4GB RAM
- 2GB disk space

---

## Technology

- Backend: Node.js, Express, TypeScript
- Frontend: React, Tailwind CSS
- Database: PostgreSQL (production), SQLite (dev)
- Auth: JWT with bcrypt
- Security: Helmet, rate limiting, CORS
- AI: Optional (HuggingFace Inference API or local LLM)

---

## License

MIT

---

**No AI required. Works offline. Production ready.**
