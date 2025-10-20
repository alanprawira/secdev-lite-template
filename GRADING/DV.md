# DV - Mini-project "DevOps pipeline"

## 0) Meta

- **Project:**: training template in [repository](https://github.com/alanprawira/secdev-seed-s06-s08.git)
- **Version (commit/date):** Add fixes / 2025-10-20
- **Briefly (1-2 sentences):** collect and visualize information about users and various objects, while simultaneously verifying the protection system
---

## 1) Reproducibility of local build and tests (DV1)

- **One command for building/testing:**
```bash
python3 -m venv .venv && source .venv/bin/activate && pip install -r requirements.txt && python scripts/init_db.py && pytest -v --junitxml=EVIDENCE/S06/test-report.xml
```
- **Tool versions (fixed):**
   ```bash
  python 3.11.0
  annotated-types==0.7.0
  anyio==4.11.0
  certifi==2025.10.5
  click==8.3.0
  fastapi==0.115.0
  h11==0.16.0
  httpcore==1.0.9
  httpx==0.27.2
  idna==3.11
  iniconfig==2.3.0
  Jinja2==3.1.4
  MarkupSafe==3.0.3
  packaging==25.0
  pluggy==1.6.0
  pydantic==2.9.1
  pydantic_core==2.23.3
  pytest==8.3.2
  sniffio==1.3.1
  starlette==0.38.6
  typing_extensions==4.15.0
  uvicorn==0.30.6
  ```

**Description of the steps (briefly):** 

1. Install Python versions  9 - 12 
2. Run the one-liner command, install dependencies, and run tests.
3. Check the resulting reports in the created folder EVIDENCE/S06.

##2) Containerization (DV2)

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

  ##3) CI: Basic pipeline and stable run (DV3)

- **CI Platform:** GitHub Actions 
- **CI config file:** 
- **Stages (minimum):** Checkout → Setup Python → Cache pip → Install deps → Init DB → Run tests → Upload artifacts
- **Configuration fragment (key steps):**

    ```yaml
  jobs:
    build-test:
      runs-on: ubuntu-latest
      env:
        DB_PATH: app.db
        EVIDENCE_DIR: EVIDENCE/S08
        PYTHONUNBUFFERED: "1"
      steps:
        - name: Checkout
          uses: actions/checkout@v4

        - name: Setup Python
          uses: actions/setup-python@v5
          with:
            python-version: "3.11"

        - name: Cache pip
          uses: actions/cache@v4
          with:
            path: ~/.cache/pip
            key: ${{ runner.os }}-pip-${{ hashFiles('**/requirements.txt') }}
            restore-keys: |
              ${{ runner.os }}-pip-

        - name: Install deps
          run: pip install -r requirements.txt

        - name: Init DB (optional)
          run: |
            if [ -f scripts/init_db.py ]; then
              python scripts/init_db.py || true
            fi

        - name: Run tests
          run: |
            mkdir -p "$EVIDENCE_DIR"
            pytest -v --junitxml="$EVIDENCE_DIR/test-report.xml"

        - name: Upload artifacts (EVIDENCE/S08)
          if: always()
          uses: actions/upload-artifact@v4
          with:
            name: evidence-s08
            path: EVIDENCE/S08/**
            if-no-files-found: warn
    ```
**Stability:** The last 7 launches are green
_ Put the files in `/EVIDENCE/` and sign their purpose._

| Artifact/log | Path to `EVIDENCE/`            | Comment |
|---------------------------------|-------------------------------|----------------------------------------------|
| Successful build/Test log (CI) | `ci-YYYY-MM-DD-build.txt ` | key steps /time |
| Local build log (optional) | `local-build-YYYY-MM-DD.txt ` | for reconciliation |
| Description of the build result | `package-notes.txt ` | what image/wheel/archive did you get |
| Freeze/tool versions | `pip-freeze.txt ` (or equivalent) | reproducibility of the environment |

##5) Secrets and environmental variables

-```dotenv

LOG_LEVEL=info
DB_PATH=/home/appuser/data/app.db

# app
API_TOKEN=
ADMIN_EMAIL=
ADMIN_PASS=

# ci/registry
REG_USER=
REG_PASS=
REGISTRY_URL=
```

**.gitignore (ключевые строки):**

```gitignore
.env
**/.env
.venv/
__pycache__/
*.pyc
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

**Использование секретов в CI:**

```yaml
- name: Login to registry (masked)
  env:
    REG_USER: ${{ secrets.REG_USER }}
    REG_PASS: ${{ secrets.REG_PASS }}
    REGISTRY_URL: ${{ vars.REGISTRY_URL }}
  run: |
    echo "::add-mask::$REG_PASS"
    echo "$REG_PASS" | docker login -u "$REG_USER" --password-stdin "$REGISTRY_URL"
```

**Быстрый grep-чек на секреты:**

```bash
git grep -nE 'AKIA[0-9A-Z]{16}|secret(_key)?=|api[_-]?key=|token=|password=|passwd=' || true
```

---

## 6) VGM
| Тип | Файл в `EVIDENCE/` | Дата/время | Коммит/версия | Runner/OS |
|---------|------------------------------------|-----------------------|-------------------|---------------|
| CI-Privacy Policy / `CI-2025-10-20-build.txt ` | '2025-10-20 hh: mm' / '<commit-sha> | / 'gha-ubuntu' |
| Лок.local-build-2025-10-20.txt ` | '2025-10-20 hh: mm' / '<commit-sha> ' / 'local' |
| Pac palage / `Pac palage-notes.txt | | ` 2025-10-20 ' / ' <image / tag>|— - |
/ Freeze | ' pip-freeze.txt | / '2025-10-20 | /' <commit-sha>|— - |
/ Grep | ' grep-secrets.txt | / '2025-10-20 | /' <commit-sha>|— - |

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

- **TM (Threat Modeling):** risks: leakage of secrets, supply chain, unsigned artifacts, root‑runtime.

  - Modify: `.env.example` + secrets in CI, fixed versions, non‑root user, healthcheck, artifacts in `EVIDENCE/'.

  - Reflect in `TM.md ` (flow chart + STRIDE table).

- **DS (DevSecOps hooks):** minimal — `grep-secrets.txt `, `pip-freeze.txt `. Optional — `pip-audit', `safety`, `trivy fs' with reports in `EVIDENCE/` and links from `DS.md `.

---
## 8) DV self-assessment (0/1/2)

- **DV1. Reproducibility of local builds and tests:** [x] 2  
  *There is a one‑liner, the versions are fixed, pytest writes JUnit XML; the steps are described. Make sure that there is `EVIDENCE` everywhere, not `EVIDENCE'.*

- **DV2. Containerization (Docker/Compose):** [x] 2  
  *Dockerfile (non‑root, healthcheck, variables), compose with volume for SQLite, port mapping, env_file.*

- **DV3. CI: basic pipeline and stable run:** [x] 2  
  *GitHub Actions: pip cache, init DB, tests, artifacts. The stability of recent launches has been noted.*

- **DV4. Pipeline artifacts and logs:** [x] 1  

- **DV5. Secrets and configuration of the environment (hygiene):** [x] 2  
  *There is an `.env.example`, rules for working with secrets in CI, a quick grep check, and rotation rules.*

**Итог DV (сумма):** **9/10** 
