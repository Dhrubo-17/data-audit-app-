# 🧪 TESTING GUIDE - Data Audit Genius v3

This guide helps you test that everything is working correctly. **No coding knowledge required!**

---

## 📋 QUICK TEST CHECKLIST

Use this to verify your deployment:

- [ ] App starts without errors
- [ ] Can access homepage (http://localhost:5000)
- [ ] Can register a new user
- [ ] Can log in and get a token
- [ ] Can upload a file (authenticated)
- [ ] Can analyze data
- [ ] Can clean data
- [ ] Can download cleaned file
- [ ] Cannot access protected endpoints without token
- [ ] Rate limiting works (blocks after too many requests)
- [ ] Large files are rejected (>50MB)
- [ ] Invalid file types are rejected (.exe, .txt, etc.)

---

## 🚀 AUTOMATED TESTS

### Run All Tests

```bash
# Make sure you're in the project folder
cd Data-Audit-Genius-v3

# Run tests
npm test
```

**Expected Output:**
```
PASS  tests/upload.test.ts
PASS  tests/auth.test.ts
PASS  tests/logic_engine.test.ts

Test Suites: 3 passed, 3 total
Tests:       15 passed, 15 total
```

### What These Tests Check:
- ✅ File upload accepts valid files (.csv, .xlsx, .xls, .json)
- ✅ File upload rejects invalid files (.exe, .pdf, .txt)
- ✅ File size limits work (rejects >50MB)
- ✅ Password hashing works
- ✅ JWT tokens are generated correctly
- ✅ JWT tokens are verified correctly
- ✅ Data parsing works for CSV and Excel
- ✅ Audit logic identifies issues correctly

---

## 🔍 MANUAL TESTS

### Test 1: Start the Application

**What to do:**
```bash
docker-compose up --build
```

**What to look for:**
- ✅ No error messages
- ✅ Message: "Server running on port 5000"
- ✅ Message: "✓ Environment validation passed"
- ✅ Message: "✓ Security middleware initialized"
- ✅ All services show "Up" when you run: `docker-compose ps`

**If it fails:**
- Check Docker Desktop is running
- Check .env file exists and JWT_SECRET is set
- Check ports 5000 and 5432 aren't used by other apps

---

### Test 2: Access the Homepage

**What to do:**
1. Open web browser
2. Go to: http://localhost:5000

**What to look for:**
- ✅ Page loads successfully
- ✅ See "Data Audit Genius" branding
- ✅ See "Register" or "Login" button
- ✅ No error messages

**If it fails:**
- Wait 30 seconds (app might still be starting)
- Check `docker-compose logs app` for errors
- Try http://127.0.0.1:5000 instead

---

### Test 3: Register a New User

**Method A: Using the Web Interface**

1. Click "Register" or "Sign Up"
2. Enter email: test@example.com
3. Enter password: TestPassword123!
4. Click "Create Account"

**What to look for:**
- ✅ Success message appears
- ✅ Redirected to dashboard or homepage
- ✅ Can see your email displayed

**Method B: Using Command Line (curl)**

**For Mac/Linux:**
```bash
curl -X POST http://localhost:5000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"TestPassword123!"}'
```

**For Windows (PowerShell):**
```powershell
Invoke-RestMethod -Uri "http://localhost:5000/api/auth/register" `
  -Method POST `
  -Headers @{"Content-Type"="application/json"} `
  -Body '{"email":"test@example.com","password":"TestPassword123!"}'
```

**Expected Response:**
```json
{
  "message": "User registered successfully",
  "user": {
    "id": 1,
    "email": "test@example.com",
    "role": "user"
  },
  "token": "eyJhbGc..."
}
```

**If it fails:**
- Error "Email already exists": User already registered, try login instead
- Error "Password too short": Use 8+ characters
- Error "Invalid email": Check email format

---

### Test 4: Login

**Method A: Using the Web Interface**

1. Click "Login"
2. Enter email: test@example.com
3. Enter password: TestPassword123!
4. Click "Log In"

**What to look for:**
- ✅ Success message
- ✅ Redirected to dashboard
- ✅ Can see datasets or upload button

**Method B: Using Command Line**

**Mac/Linux:**
```bash
curl -X POST http://localhost:5000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"TestPassword123!"}'
```

**Windows (PowerShell):**
```powershell
Invoke-RestMethod -Uri "http://localhost:5000/api/auth/login" `
  -Method POST `
  -Headers @{"Content-Type"="application/json"} `
  -Body '{"email":"test@example.com","password":"TestPassword123!"}'
```

**Expected Response:**
```json
{
  "message": "Login successful",
  "user": {
    "id": 1,
    "email": "test@example.com",
    "role": "user"
  },
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Save this token!** You'll need it for the next tests.

**If it fails:**
- Error "Invalid credentials": Check email/password
- Error "User not found": Register first

---

### Test 5: Upload a File (With Authentication)

**Prepare a test file:**
- Create a small CSV file called `test.csv`:
```csv
Name,Age,Email
John Doe,30,john@example.com
Jane Smith,25,jane@example.com
Bob Wilson,35,bob@example.com
```

**Method A: Using the Web Interface**

1. Make sure you're logged in
2. Click "Upload Dataset" or similar button
3. Select test.csv
4. Click "Upload"

**What to look for:**
- ✅ File uploads successfully
- ✅ See confirmation message
- ✅ File appears in your datasets list

**Method B: Using Command Line**

**Mac/Linux:**
```bash
# Replace YOUR_TOKEN_HERE with the token from login
curl -X POST http://localhost:5000/api/datasets/upload \
  -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  -F "file=@test.csv"
```

**Windows (PowerShell):**
```powershell
# Create test file first, then:
$token = "YOUR_TOKEN_HERE"
Invoke-RestMethod -Uri "http://localhost:5000/api/datasets/upload" `
  -Method POST `
  -Headers @{"Authorization"="Bearer $token"} `
  -Form @{file=Get-Item -Path "test.csv"}
```

**Expected Response:**
```json
{
  "id": 1,
  "filename": "1706745600000_abc123_test.csv",
  "originalName": "test.csv",
  "mimeType": "text/csv",
  "createdAt": "2025-01-31T12:00:00.000Z"
}
```

**If it fails:**
- Error 401: Token missing or invalid, check Authorization header
- Error 400 "Invalid file type": Check file extension (.csv, .xlsx, .xls, .json only)
- Error 400 "File too large": File must be under 50MB

---

### Test 6: Try Upload WITHOUT Authentication (Should Fail)

**What to do:**
```bash
# Try to upload without token
curl -X POST http://localhost:5000/api/datasets/upload \
  -F "file=@test.csv"
```

**Expected Response:**
```json
{
  "error": "Authentication required",
  "message": "No authorization header provided"
}
```

**What to look for:**
- ✅ Request is rejected
- ✅ Status code: 401 (Unauthorized)
- ✅ Error message mentions authentication

**This proves authentication is working!**

---

### Test 7: Analyze the Dataset

**Method A: Using the Web Interface**

1. Go to your datasets list
2. Click on the dataset you uploaded
3. Click "Analyze" button
4. Wait for analysis to complete (10-30 seconds)

**What to look for:**
- ✅ Analysis completes successfully
- ✅ See data quality report
- ✅ See column types detected
- ✅ See issues found (if any)
- ✅ See cleaning suggestions

**Method B: Using Command Line**

```bash
# Replace 1 with your dataset ID, and YOUR_TOKEN_HERE with your token
curl -X POST http://localhost:5000/api/datasets/1/analyze \
  -H "Authorization: Bearer YOUR_TOKEN_HERE"
```

**Expected Response:**
```json
{
  "analysis": {
    "rowCount": 3,
    "columns": [...],
    "cleaningActions": [...],
    "industryGuess": "General Business"
  }
}
```

---

### Test 8: Clean the Dataset

**Method A: Using the Web Interface**

1. After analysis, review cleaning suggestions
2. Check boxes for fixes you want
3. Click "Clean Dataset"
4. Wait for cleaning to complete

**What to look for:**
- ✅ Cleaning completes successfully
- ✅ See summary of changes made
- ✅ See before/after comparison
- ✅ Download button appears

**Method B: Using Command Line**

```bash
curl -X POST http://localhost:5000/api/datasets/1/clean \
  -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  -H "Content-Type: application/json" \
  -d '{"actionIds":["standardize_date_column1","validate_email_column2"]}'
```

---

### Test 9: Download Cleaned File

**Method A: Using the Web Interface**

1. Click "Download Cleaned File" or "Export"
2. File should download to your Downloads folder

**What to look for:**
- ✅ File downloads successfully
- ✅ File name includes "cleaned"
- ✅ Can open file in Excel or text editor
- ✅ See the cleaned data

**Method B: Using Command Line**

```bash
curl -X GET http://localhost:5000/api/datasets/1/export \
  -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  --output cleaned_file.xlsx
```

---

### Test 10: Rate Limiting

**What to do:**
```bash
# Send many requests quickly
for i in {1..70}; do
  curl http://localhost:5000/api/health
  echo "Request $i"
done
```

**What to look for:**
- ✅ First 60 requests succeed
- ✅ Requests 61+ get error 429
- ✅ Error message: "Too many requests"

**This proves rate limiting is working!**

**Wait 1 minute and try again:**
- ✅ Requests work again after window resets

---

### Test 11: File Type Validation

**Create invalid test files:**

**Test 11a: .exe file (should be rejected)**
```bash
echo "test" > virus.exe

curl -X POST http://localhost:5000/api/datasets/upload \
  -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  -F "file=@virus.exe"
```

**Expected:** Error 400 "Invalid file type"

**Test 11b: .txt file (should be rejected)**
```bash
echo "test data" > data.txt

curl -X POST http://localhost:5000/api/datasets/upload \
  -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  -F "file=@data.txt"
```

**Expected:** Error 400 "Invalid file type"

**Test 11c: .csv file (should succeed)**
```bash
curl -X POST http://localhost:5000/api/datasets/upload \
  -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  -F "file=@test.csv"
```

**Expected:** Success 201

**This proves file type validation is working!**

---

### Test 12: File Size Limit

**What to do:**
```bash
# Create a 60MB file (over the 50MB limit)
dd if=/dev/zero of=large.csv bs=1M count=60

# Try to upload it
curl -X POST http://localhost:5000/api/datasets/upload \
  -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  -F "file=@large.csv"
```

**Expected Response:**
```json
{
  "error": "File too large. Maximum size: 50MB"
}
```

**What to look for:**
- ✅ Upload is rejected
- ✅ Status code: 400
- ✅ Error mentions file size

**This proves file size limiting is working!**

---

## 🗄️ DATABASE TESTS

### Test 13: Check Database Connection

**What to do:**
```bash
docker-compose exec db psql -U postgres -d data_audit_genius -c "SELECT 1;"
```

**Expected Output:**
```
 ?column? 
----------
        1
(1 row)
```

### Test 14: Check Users Table Exists

**What to do:**
```bash
docker-compose exec db psql -U postgres -d data_audit_genius -c "\dt users"
```

**Expected Output:**
```
         List of relations
 Schema | Name  | Type  |  Owner   
--------+-------+-------+----------
 public | users | table | postgres
```

### Test 15: Count Users

**What to do:**
```bash
docker-compose exec db psql -U postgres -d data_audit_genius -c "SELECT COUNT(*) FROM users;"
```

**Expected Output:**
```
 count 
-------
     1
(1 row)
```
(Should be 1 if you registered one user)

---

## 🔐 SECURITY TESTS

### Test 16: SQL Injection Protection

**What to do:**
Try to inject SQL via the email field:
```bash
curl -X POST http://localhost:5000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@example.com; DROP TABLE users;--","password":"test"}'
```

**What to look for:**
- ✅ Request fails safely
- ✅ Database still intact
- ✅ Run: `docker-compose exec db psql -U postgres -d data_audit_genius -c "\dt users"`
- ✅ Users table still exists

**This proves SQL injection protection is working!**

### Test 17: Path Traversal Protection

**What to do:**
```bash
curl -X POST http://localhost:5000/api/datasets/upload \
  -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  -F "file=@test.csv;filename=../../etc/passwd"
```

**What to look for:**
- ✅ File is saved with sanitized name
- ✅ No path traversal occurs
- ✅ Check: `ls uploads/` - file should be in uploads directory with safe name

---

## 🐳 DOCKER TESTS

### Test 18: Container Health

**What to do:**
```bash
docker-compose ps
```

**Expected Output:**
```
NAME                    STATUS              PORTS
data-audit-genius-db-1  Up (healthy)       5432/tcp
data-audit-genius-app-1 Up (healthy)       0.0.0.0:5000->5000/tcp
```

**What to look for:**
- ✅ All services show "Up"
- ✅ Health status is "healthy"
- ✅ Ports are mapped correctly

### Test 19: Container Logs

**What to do:**
```bash
docker-compose logs app --tail=20
```

**What to look for:**
- ✅ No error messages
- ✅ See "Server running on port 5000"
- ✅ See "✓ Environment validation passed"

### Test 20: Container Restart

**What to do:**
```bash
docker-compose restart app
```

**What to look for:**
- ✅ Container restarts successfully
- ✅ App accessible again within 30 seconds
- ✅ Data persists (uploads and database intact)

---

## ⚡ PERFORMANCE TESTS

### Test 21: Response Time

**What to do:**
```bash
time curl http://localhost:5000/health
```

**Expected:**
- ✅ Response time < 100ms
- ✅ Status 200

### Test 22: Concurrent Requests

**What to do:**
```bash
# Run 10 concurrent requests
for i in {1..10}; do
  curl http://localhost:5000/health &
done
wait
```

**What to look for:**
- ✅ All requests succeed
- ✅ No errors
- ✅ Server remains responsive

---

## 🎯 END-TO-END TEST SCENARIO

### Complete User Journey

**Scenario:** New user uploads, analyzes, cleans, and downloads data

1. **Register**
   ```bash
   curl -X POST http://localhost:5000/api/auth/register \
     -H "Content-Type: application/json" \
     -d '{"email":"fulltest@example.com","password":"Test1234!"}' \
     | jq .token -r > token.txt
   ```

2. **Upload File**
   ```bash
   TOKEN=$(cat token.txt)
   curl -X POST http://localhost:5000/api/datasets/upload \
     -H "Authorization: Bearer $TOKEN" \
     -F "file=@test.csv" \
     | jq .id -r > dataset_id.txt
   ```

3. **Analyze**
   ```bash
   DATASET_ID=$(cat dataset_id.txt)
   curl -X POST http://localhost:5000/api/datasets/$DATASET_ID/analyze \
     -H "Authorization: Bearer $TOKEN"
   ```

4. **Clean**
   ```bash
   curl -X POST http://localhost:5000/api/datasets/$DATASET_ID/clean \
     -H "Authorization: Bearer $TOKEN" \
     -H "Content-Type: application/json" \
     -d '{"actionIds":[]}'
   ```

5. **Download**
   ```bash
   curl -X GET http://localhost:5000/api/datasets/$DATASET_ID/export \
     -H "Authorization: Bearer $TOKEN" \
     --output cleaned_file.xlsx
   ```

6. **Verify**
   ```bash
   ls -lh cleaned_file.xlsx
   file cleaned_file.xlsx
   ```

**Expected:**
- ✅ All steps complete successfully
- ✅ No errors at any stage
- ✅ cleaned_file.xlsx exists and is valid

---

## 📊 TEST RESULTS CHECKLIST

After running all tests, verify:

### Core Functionality
- [ ] ✅ App starts successfully
- [ ] ✅ Homepage loads
- [ ] ✅ User registration works
- [ ] ✅ User login works
- [ ] ✅ File upload works (with auth)
- [ ] ✅ Data analysis works
- [ ] ✅ Data cleaning works
- [ ] ✅ File download works

### Security
- [ ] ✅ Authentication required for protected endpoints
- [ ] ✅ Invalid file types rejected
- [ ] ✅ Large files rejected
- [ ] ✅ Rate limiting active
- [ ] ✅ SQL injection prevented
- [ ] ✅ Path traversal prevented

### Database
- [ ] ✅ Database connection works
- [ ] ✅ Users table exists
- [ ] ✅ Data persists across restarts

### Docker
- [ ] ✅ All containers healthy
- [ ] ✅ No errors in logs
- [ ] ✅ Restart works correctly

---

## 🐛 TROUBLESHOOTING FAILED TESTS

### If Registration Fails
- Check database is running: `docker-compose ps db`
- Check logs: `docker-compose logs db`
- Verify DATABASE_URL in .env

### If Upload Fails
- Check uploads directory exists: `ls -la uploads/`
- Check token is valid (not expired)
- Check file size and type

### If Rate Limiting Doesn't Work
- Check logs for rate limit middleware loading
- Verify RATE_LIMIT_MAX in .env
- Wait full window (60 seconds) between tests

### If Database Tests Fail
- Restart database: `docker-compose restart db`
- Check PostgreSQL logs: `docker-compose logs db`
- Verify DATABASE_URL matches docker-compose.yml

---

## ✅ CERTIFICATION

Once all tests pass, your deployment is **production-ready**!

**Sign off:**
```
Tested by: ________________
Date: ________________
All tests passed: YES / NO
Notes: ________________
```

---

**For deployment instructions, see DEPLOY.md**
**For release details, see RELEASE_NOTES.md**
