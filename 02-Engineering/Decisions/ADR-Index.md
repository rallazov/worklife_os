# ADR Index (temp prompt)
tags: #adr

- [[02-Engineering/Decisions/ADR-0001-Identity-vs-Mailbox-Separation|ADR-0001 Identity vs Mailbox Separation]] (create when ready)
- [[02-Engineering/Decisions/ADR-0002-Session-Auth-Cookies|ADR-0002 Session Auth Cookies]] (create when ready)
- [[02-Engineering/Decisions/ADR-0003-Workbench-Filter-Truth-Model|ADR-0003 Workbench Filter Truth Model]] (create when ready)


# Pre-Launch Security & Cleanup Audit for Domayn

You are auditing this codebase before production deployment and investor demo. This is a FastAPI + React application. Perform a thorough audit and provide actionable findings.

## PART 1: SECURITY AUDIT (Critical)

### 1.1 Secrets & Credentials Exposure
Scan the ENTIRE codebase for:
- [ ] Hardcoded API keys, tokens, passwords
- [ ] `.env` files committed to git (check git history too)
- [ ] Credentials in comments or TODOs
- [ ] Private keys, certificates, or PEM files
- [ ] Database connection strings with passwords
- [ ] OAuth client secrets in frontend code
- [ ] JWT secrets hardcoded anywhere

Commands to run:
```bash
# Search for common secret patterns
grep -rn "OPENAI_API_KEY\|SECRET_KEY\|password\|api_key\|token" --include="*.py" --include="*.ts" --include="*.tsx" --include="*.js" --include="*.json"

# Check for .env files in git
git ls-files | grep -E "\.env"

# Search git history for secrets
git log -p | grep -E "(api_key|secret|password|token)" | head -50
```

### 1.2 .gitignore Verification
Ensure these are in `.gitignore` and NOT in the repo:
```
# Must be ignored
.env
.env.*
*.pem
*.key
credentials/
secrets/
*.db
*.sqlite
*.sqlite3
app.db
__pycache__/
node_modules/
.DS_Store
*.log
*.pyc
.idea/
.vscode/
dist/
build/
*.egg-info/
.pytest_cache/
coverage/
.coverage
```

### 1.3 Exposed Endpoints
Check for:
- [ ] Debug endpoints (e.g., `/debug`, `/test`, `/_internal`)
- [ ] Admin routes without authentication
- [ ] API endpoints that expose sensitive user data
- [ ] CORS configured too permissively (`*` in production)
- [ ] GraphQL introspection enabled (if applicable)

### 1.4 Authentication & Authorization
Verify:
- [ ] All protected routes have `Depends(get_current_user)` or equivalent
- [ ] User can only access their own data (check `user_id` filters on ALL queries)
- [ ] Session/token expiration is configured
- [ ] Password hashing uses bcrypt/argon2 (not MD5/SHA1)
- [ ] OAuth tokens are encrypted at rest

## PART 2: CODE CLEANUP

### 2.1 Debug & Development Code
Remove or disable:
- [ ] `print()` statements (use logging instead)
- [ ] `console.log()` in frontend (except error handling)
- [ ] `debugger;` statements
- [ ] `# TODO` and `# FIXME` that expose vulnerabilities
- [ ] Commented-out code blocks (more than 10 lines)
- [ ] Mock/fake data in production code paths
- [ ] `USE_MOCK_GPT` or similar test flags defaulting to true

### 2.2 Test & Debug Files
Identify files that should NOT be in production:
- [ ] `test_*.py` files outside `/tests` directory
- [ ] `debug_*.py` scripts
- [ ] `*_backup.*` files
- [ ] `*.bak`, `*.tmp`, `*.old` files
- [ ] Jupyter notebooks (`.ipynb`) with sensitive data
- [ ] SQL dump files (`.sql`)
- [ ] CSV/JSON data files with real user data

### 2.3 Dependencies Audit
```bash
# Backend - check for vulnerabilities
pip audit  # or: safety check

# Frontend - check for vulnerabilities  
cd frontend && npm audit
```

Flag:
- [ ] Unused dependencies in requirements.txt / package.json
- [ ] Pinned versions vs unpinned (security risk)
- [ ] Known vulnerable packages

## PART 3: DATA & DATABASE

### 3.1 Sensitive Data in Repository
Search for:
- [ ] Real email addresses (not @example.com)
- [ ] Real names in seed/test data
- [ ] Production database dumps
- [ ] CSV/Excel files with PII
- [ ] Log files with user data
```bash
# Find data files
find . -name "*.csv" -o -name "*.json" -o -name "*.sql" -o -name "*.db" | head -20

# Search for email patterns in code
grep -rn "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}" --include="*.py" --include="*.ts" --include="*.tsx" --include="*.json"
```

### 3.2 Database Security
- [ ] SQLite file (app.db) is in .gitignore
- [ ] No raw SQL with string concatenation (SQL injection risk)
- [ ] Migrations don't contain sensitive seed data
- [ ] Encrypted fields (google_token) are properly handled

## PART 4: CONFIGURATION

### 4.1 Environment Variables
Create/verify `.env.example` with ALL required variables (no real values):
```
# Database
DATABASE_URL=postgresql+psycopg2://user:password@host:5432/dbname

# Auth
SECRET_KEY=generate-a-secure-random-key
GOOGLE_CLIENT_ID=your-client-id
GOOGLE_CLIENT_SECRET=your-client-secret
GOOGLE_REDIRECT_URI=http://localhost:8000/auth/callback

# OpenAI
OPENAI_API_KEY=sk-your-key

# Frontend
FRONTEND_URL=http://localhost:3000

# Environment
APP_ENV=development
```

### 4.2 Production vs Development Config
Verify:
- [ ] `DEBUG=False` or equivalent for production
- [ ] Different `SECRET_KEY` for production
- [ ] `CORS_ORIGINS` restricted to actual domains
- [ ] `ALLOWED_HOSTS` configured
- [ ] HTTPS enforced (no HTTP in production)

## PART 5: FILE STRUCTURE CLEANUP

### 5.1 Organize Loose Files
Identify misplaced files:
- [ ] Scripts in root that belong in `/scripts`
- [ ] Test files outside `/tests`
- [ ] Config files that should be in `/config`
- [ ] Data files that should be in `/data` (and gitignored)

### 5.2 Remove Unnecessary Files
Flag for deletion:
- [ ] Empty `__init__.py` files that aren't needed
- [ ] Duplicate files (same content, different names)
- [ ] Old backup files
- [ ] IDE-specific files not in .gitignore
- [ ] Build artifacts

## PART 6: FRONTEND SPECIFIC

### 6.1 React Cleanup
- [ ] Remove `React.StrictMode` console warnings in production build
- [ ] No API keys in frontend code (all should go through backend)
- [ ] Environment variables use `REACT_APP_` prefix properly
- [ ] No hardcoded `localhost` URLs (use env vars)
- [ ] Source maps disabled for production build

### 6.2 Build Verification
```bash
cd frontend
npm run build  # Should complete without errors
```

## PART 7: DOCUMENTATION

### 7.1 README.md Must Have
- [ ] Project description
- [ ] Setup instructions (backend + frontend)
- [ ] Environment variables list
- [ ] How to run locally
- [ ] How to run tests
- [ ] Deployment instructions

### 7.2 Remove/Update
- [ ] References to "AppTracker" (old name) → "Domayn"
- [ ] Placeholder text ("Lorem ipsum", "TODO", "FIXME")
- [ ] Developer names/emails if private repo going public

---

## OUTPUT FORMAT

Provide findings in this structure:

### 🔴 CRITICAL (Fix before going live)
| File | Line | Issue | Fix |
|------|------|-------|-----|

### 🟠 HIGH (Fix within 1 week)
| File | Line | Issue | Fix |
|------|------|-------|-----|

### 🟡 MEDIUM (Technical debt)
| File | Line | Issue | Fix |
|------|------|-------|-----|

### 🟢 RECOMMENDATIONS (Nice to have)
| Area | Suggestion |
|------|------------|

### FILES TO DELETE
```
path/to/file1
path/to/file2
```

### FILES TO ADD TO .gitignore
```
pattern1
pattern2
```

### ENVIRONMENT VARIABLES CHECKLIST
| Variable | Status | Notes |
|----------|--------|-------|
| DATABASE_URL | ✅/❌ | |
| SECRET_KEY | ✅/❌ | |
| ... | | |

---

Begin the audit now. Be thorough and specific. Do not skip any section.
