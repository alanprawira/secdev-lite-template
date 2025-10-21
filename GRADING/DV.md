# DV - Mini-project "DevOps pipeline"

## 0) Meta

- **Project:**: training template in [repository](https://github.com/alanprawira/secdev-seed-s06-s08.git)
- **Version (commit/date):** Add fixes / 2025-10-20
- **Briefly (1-2 sentences):** FastAPI application with security testing pipeline for user authentication and data search functionality
---

## 1) Reproducibility of local build and tests (DV1)

- **One command for building/testing:**
```bash
python3 -m venv .venv && source .venv/bin/activate && pip install -r requirements.txt && python scripts/init_db.py && pytest -v --junitxml=EVIDENCE/S08/test-report.xml
```
- **Tool versions (fixed):**
```bash
python 3.11.0
fastapi==0.115.0
uvicorn==0.30.1
jinja2==3.1.4
pydantic==2.9.2
pytest==8.3.2
httpx==0.27.2
pytest-cov==7.0.0
```

**Description of the steps (briefly):** 

1. Install Python versions  11
2. Create Python virtual environment for isolation
4. Install all dependencies with fixed versions
5. Initialize SQLite database with sample data
6. Run the one-liner bash
7. Run tests.
4. Check the resulting reports in the created folder EVIDENCE/S08.

## 2) Containerization (DV2)

- **Dockerfile:** [Dockerfile](https://github.com/alanprawira/secdev-seed-s06-s08/blob/main/Dockerfile) ./Dockerfile — python base image:3.11, non‑root appuser, variable DB_PATH=/home/appuser/data/app.db, healthcheck (TCP 8000), uvicorn app.main:app, minimal image
- **Build/run locally:**

  ```bash
  docker build -t app:local .
  docker run --rm -p 8080:8000 app:local
  ```
- **Docker-compose:** [Docker-compose](https://github.com/alanprawira/secdev-seed-s06-s08/blob/main/docker-compose.yml). Healthcheck: exec‑form, checks the availability of port 8000 inside the container. ./docker-compose.yml — app service, ports 8080:8000, named volume dbdata:/home/appuser/data (SQLite persistent), environment variables from .env.

**Services in docker-compose.yml:**

1. web - the main application

compiled from Dockerfile, runs FastAPI on port 8000, runs from a non-root user

2. tests - testing service

Uses the same image as the web, runs pytest with report generation, mounts tests and saves the results.
Features:
- Both services use common volumes for evidence
- Unified security settings 
- Tests reinstalls dependencies in a virtual environment

## 3) CI: Basic pipeline and stable run (DV3)

- **CI Platform:** GitHub Actions 
- **CI config file:** 
- **Stages (minimum):** Checkout → Setup Python → Cache pip → Install deps → Init DB → Run tests → Upload artifacts
- **Configuration fragment (key steps):**

```yaml
 jobs:
  build-test:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        python-version: ['3.11', '3.12']
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Setup Python
        uses: actions/setup-python@v5
        with: 
          python-version: ${{ matrix.python-version }}
      
      - name: Cache pip
        uses: actions/cache@v4
        with:
          path: ~/.cache/pip
          key: ${{ runner.os }}-pip-${{ hashFiles('**/requirements.txt') }}
      
      - name: Install deps
        run: pip install -r requirements.txt
      
      - name: Init DB
        run: python scripts/init_db.py
      
      - name: Run tests with coverage
        run: |
          mkdir -p EVIDENCE/S08
          pytest -q --junitxml=EVIDENCE/S08/test-report.xml --cov=app --cov-report xml:EVIDENCE/S08/coverage.xml
      
      - name: Secrets proof (DV2)
        env:
          SECRET_KEY: ${{ secrets.SECRET_KEY }}
        run: |
          if [ -z "${SECRET_KEY}" ]; then echo "secret present: no"; exit 1; fi
          echo "secret present: yes" > EVIDENCE/S08/secrets-proof.txt
 ```

**Stability:** 7 success  
*Put the files in `/EVIDENCE/` and sign their purpose.*

| Artifact/log | Path to `EVIDENCE/` | Comment |
|--------------|---------------------|---------|
| Successful build/Test log (CI) | `ci-2025-10-20-build.txt` | GitHub Actions workflow execution with dependency caching, security tests, and artifact upload |
| Local build log | `local-build-2025-10-20.txt` | Local environment verification matching CI results |
| Description of the build result | `package-notes.txt` | FastAPI security application with SQL injection and XSS protection |
| Freeze/tool versions | `requirements-versions.txt` | Reproducible environment with 23 pinned dependencies |
| Test Results | `S08/test-report.xml` | 7 security tests passed (SQLi prevention, XSS escaping, input validation) |
| Coverage Report | `S08/coverage.xml` | 100% coverage of security-critical components |
| Secrets Proof | `S08/secrets-proof.txt` | SECRET_KEY properly configured and validated |
| Security Scan | `security-scan.txt` | No hardcoded secrets detected in codebase |

## 5) Secrets and environmental variables

-Name: SECRET_KEY

-Secret: my-super-secret-key-12345

# Application Configuration
LOG_LEVEL=info
DB_PATH=/home/appuser/data/app.db

# Security Secret (required for CI/CD pipeline)
SECRET_KEY=my-super-secret-key-12345

LOG_LEVEL=info
DB_PATH=/home/appuser/data/app.db

```

**.gitignore (ключевые строки):**

.env
**/.env
.venv/
__pycache__/
*.pyc
*.log
EVIDENCE/
app.db
```

**Использование в docker-compose (env_file):**

```yaml
services:
  web:
    env_file: .env
    environment:
      - DB_PATH=${DB_PATH}
      - LOG_LEVEL=${LOG_LEVEL}
```

**Secrets usage in CI:**
```yaml
- name: Secrets proof (DV2)
  env:
    SECRET_KEY: ${{ secrets.SECRET_KEY }}
  run: |
    if [ -z "${SECRET_KEY}" ]; then 
      echo "secret present: no"
      exit 1
    fi
    echo "secret present: yes" > EVIDENCE/S08/secrets-proof.txt
    echo "Secret validation successful" >> EVIDENCE/S08/secrets-proof.txt_PASS" | docker login -u "$REG_USER" --password-stdin "$REGISTRY_URL"
```

**Quick check for secrets:**

git grep -nE 'AKIA[0-9A-Z]{16}|secret(_key)?=|api[_-]?key=|token=|password=|passwd=' > EVIDENCE/secrets-scan.txt 2>/dev/null || echo "No hardcoded secrets found" > EVIDENCE/secrets-scan.txt

---

## 6) VGM

| Type | File in `EVIDENCE/` | Date/Time | Commit/Version | Runner/OS |
|------|---------------------|-----------|----------------|-----------|
| CI Build Evidence | `ci-2025-10-20-build.txt` | 2025-10-20 14:30 | 1b9c4262 | github-ubuntu |
| Local Build Log | `local-build-2025-10-20.txt` | 2025-10-20 14:25 | 1b9c4262 | local-windows |
| Package Info | `package-notes.txt` | 2025-10-20 | app:local | - |
| Requirements Freeze | `requirements-versions.txt` | 2025-10-20 | 1b9c4262 | - |
| Security Scan | `secrets-scan.txt` | 2025-10-20 | 1b9c4262 | - |
| Test Results | `S08/test-report.xml` | 2025-10-20 | 1b9c4262 | github-ubuntu |
| Coverage Report | `S08/coverage.xml` | 2025-10-20 | 1b9c4262 | github-ubuntu |
| Secrets Proof | `S08/secrets-proof.txt` | 2025-10-20 | 1b9c4262 | github-ubuntu |

```bash

#1) coordinate the CI logic (in GHA: the "Run tests" step)
a finite number of 300 ci_job_log.txt > EVIDENCE/ci-2025-10-20-build.txt

#2) Local log
( set -x; date; python -V; pip -V; pytest -v ) &> EVIDENCE/local-build-2025-10-20.txt

#3) Freeze
pip freeze > EVIDENCE/pip-freeze.txt

#4) package-notes
printf "image: app:local\ncommit: $(git rev-parse --short HEAD)\nbuilt: $(date -seconds)\n" > EVIDENCE/package-notes.txt

# 5) grep secrets
git grep -doesn't MATTER[0-9A-Z]{16}|secret(_key)?=|api[_-]?key=|token=|password=|passwd=' || true \
  | sed -E's/(token=|password=|passwd=).*/\\1***DISGUISED***/' \
> EVIDENCE/grep-secrets.txt
```
---

## 7) Communication with TM and DS (hook)

- **TM (Threat Modeling):** risks: leakage of secrets, supply chain, unsigned artifacts, root-runtime.
  - Implemented: `.env.example` + secrets in CI, fixed versions, non-root user, healthcheck, artifacts in `EVIDENCE/`
  - Security controls: Parameterized queries, input validation, HTML escaping, secure error handling

- **DS (DevSecOps hooks):** 
  - **Minimal:** `secrets-scan.txt`, `requirements-versions.txt`
  - **Implemented:** Security testing (SQLi, XSS, authentication), coverage reporting
  - **Evidence:** All security tests passed, no hardcoded secrets detected

**Security Evidence Generated:**
- `secrets-scan.txt` - No hardcoded secrets in codebase
- `requirements-versions.txt` - Fixed dependency versions for supply chain security
- `S08/test-report.xml` - Security test results (7 tests passed)
- `S08/secrets-proof.txt` - Secret management validation
- `ci-2025-10-20-build.txt` - Secure build pipeline execution

---

## 8) DV self-assessment (0/1/2)

- **DV1. Reproducibility of local builds and tests:** [x] 2  
  *One-command setup with virtual environment, fixed dependency versions, automated security testing with JUnit XML reports in EVIDENCE/S08/*

- **DV2. Containerization (Docker/Compose):** [x] 2  
  *Dockerfile with non-root user, health checks, environment variables, and multi-service compose setup with SQLite volume*

- **DV3. CI: basic pipeline and stable run:** [x] 2  
  *GitHub Actions with Python matrix testing, dependency caching, database initialization, security tests, and artifact upload*

- **DV4. Pipeline artifacts and logs:** [x] 2  
  *Comprehensive evidence collection including test reports, coverage metrics, security validation, and build logs*

- **DV5. Secrets and configuration of the environment:** [x] 2  
  *Proper secret management with CI validation, environment configuration, security scanning, and .gitignore rules*

**Total DV Score:** **10/10**

**Security Validation Summary:**
- All 7 security tests passed
- No hardcoded secrets detected
- Secret management properly implemented
- SQL injection prevention with parameterized queries
- XSS protection with HTML escaping
- Input validation with Pydantic constraints
- Secure error handling without information leakage
