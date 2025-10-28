# DS - Отчёт «DevSecOps-сканы и харднинг»

> Этот файл - **индивидуальный**. Его проверяют по **rubric_DS.md** (5 критериев × {0/1/2} → 0-10).
> Подсказки помечены `TODO:` - удалите после заполнения.
> Все доказательства/скрины кладите в **EVIDENCE/** и ссылайтесь на конкретные файлы/якоря.

---

## 0) Мета

- **Проект (опционально BYO):** TODO: secdev-seed-s09-s12
- **Версия (commit/date):** TODO: abc123 / 2025-10-28
- **Кратко (1-2 предложения):** TODO: security scanning of FastAPI application with container security

---

## 1) SBOM и уязвимости зависимостей (DS1)

**Interface/formatting:** Syft/Grype; CycloneDX
- **How did I launch it:**

  ``bash
syft dir:. -o cyclonedx-json > EVIDENCE/S09/sbom.json
enter the sbom password:/work/EVIDENCE/S09/sbom.json -o json > EVIDENCE/S09/sca_report.json
  ```

- **S09 - SBOM and SCA**: generated SBOM (CycloneDX) filled with SCA. 
- **Artekakty**: EVIDENCE/S09/sbom.json, EVIDENCE/S09/sca_report.json. 
- **Link**: EVIDENCE/S09/sca_summary.md, EVIDENCE/S09/license_summary.md. 
- **Link to resp**: [respository] (https://github.com/alanprawira/secdev-seed-s09-s12)
-  **Link to action**: [workflow] (https://github.com/alanprawira/secdev-seed-s09-s12/actions)
- Up to - 3 medium (sca_summary_old.md ), later - 0 (sca_summary.md )

---

## 2) SAST и Secrets (DS2)

### 2.1 SAST

- **Tool/Profile:** semgrep
- **How did I launch it:**

  ```bash
  semgrep semgrep ci --config p/ci --sarif --output /src/EVIDENCE/S10/semgrep.sarif --metrics=off || true
  ```

- **Artifacts:** `EVIDENCE/S10/semgrep.sarif`
- **Conclusions:** FP reviewed

  ```

### 2.2 Secrets scanning

- **Tool:** gitleaks
- **How did I launch it:**

  ```bash
  gitleaks detect --source=/repo --report-format=json --report-path=/repo/EVIDENCE/S10/gitleaks.json || true
  ```

- **Artifacts:** `EVIDENCE/S10/gitleaks.json`
- **Conclusion:** there are gitleaks threats, but fixed
- **Link to unsuccessful** [fail] (https://github.com/alanprawira/secdev-seed-s09-s12/actions/runs/18881658908)

---

## 3) DAST **или** Policy (Container/IaC) (DS3)

### DAST (lite)

- **Tool/Target:** zap
- **How did I launch it:**

  ```bash
  zap-baseline.py -t http://localhost:8080 -r zap_baseline.html -J zap_baseline.json-d || true
  ```

- **Artifacts:** `EVIDENCE/S11/zap_baseline.html`, `EVIDENCE/S11/zap_baseline.json-d.json`
- **Conclusions:** There are 2 medium risks and 3 low risks in this project.


### Вариант B - Policy / Container / IaC

### Policy / Container / IaC

- **Tool(s):** trivy config / checkov 
- **How did I launch it:**

  ```bash
  trivy image --format json --output /work/EVIDENCE/S12/trivy.json --ignore-unfixed s09s12-app:ci || true
  checkov -d /src/iac -o json > EVIDENCE/S12/checkov.json || true
  ```

- **Artifacts:** `EVIDENCE/S12/trivy.json`, `EVIDENCE/S12/checkov.json`
- Before - 17 warnings (checkov-old.json), after - 15 (checkov.json)
---

## 4) Харднинг (доказуемый) (DS4)

Отметьте **реально применённые** меры, приложите доказательства из `EVIDENCE/`.

- [x] **Контейнер non-root / drop capabilities** → Evidence: `EVIDENCE/S12/non-root.txt`
- [x] **HEALTHCHECK** → Evidence: `EVIDENCE/S12/health.json`
- [x] **Отказ от latest** -> Evidence: `EVIDENCE/S12/checkov.json`
- [x] **K8s non-root** -> Evidence: `EVIDENCE/S12/checkov.json`

> Для «1» достаточно ≥2 уместных мер с доказательствами; для «2» - ≥3 и хотя бы по одной показать эффект «до/после».

---

## 5) Quality-gates и проверка порогов (DS5)

- **Threshold rules (in words):**
SCA: Critical=0; High≤1, "SAST: Critical=0", "Secrets: 0 true findings", "Policy: Violations=0".
- **How are they checked:**
- Automatically: (script/job, fail condition in case of violation)

    ```bash
    SCA: grype sbom:/work/EVIDENCE/S09/sbom.json -o json > EVIDENCE/S09/sca_report.json --fail-on high
    SAST: semgrep scan --config "p/ci" --sarif --output /src/EVIDENCE/S10/semgrep.sarif --metrics=off --error --severity=ERROR 
    Secrets: gitleaks detect --source=/repo --report-format=json --report-path=/repo/EVIDENCE/S10/gitleaks.json --exit-code 1
    Policy/IaC: trivy image --format json --output /work/EVIDENCE/S12/trivy.json --ignore-unfixed s09s12-app:ci --severity HIGH,CRITICAL --exit-code 1
    DAST: zap-baseline.py -m 3 (fail at High)
``

    ```

- **Ссылки на конфиг/скрипт (если есть):**

  ```bash
  GitHub Actions: .github/workflows/security.yml (jobs: sca, sast, secrets, policy, dast)
  или GitLab CI: .gitlab-ci.yml (stages: security; jobs: sca/sast/secrets/policy/dast)
  ```

---

## 6) Триаж-лог (fixed / suppressed / open)

| ID/Binding | Class | Severity | Status | Action | Evidence | Website link/Indication | Comment / owner / expiration date |
|-----------------|-----------|----------|------------|----------|----------------------------------------|-----------------------------------|------------------------------|
| CVE (FULL text )-2024-56201 | SCA | High |fixed| crash | `EVIDENCE/S09/old/sca_report.json` | `EVIDENCE/deps-YYYY-MM-DD.json#CVE` | Sandbox breakout |
| CVE (FULL text )-2024-56326 | SCA | Average |fixed | failure | `EVIDENCE/S09/old/sca_report.json` | `EVIDENCE/dast-YYYY-MM-DD.pdf#123` | - |
| CVE (FULL text )-2025-27516 | SCA | Average/High |fixed| crash | `EVIDENCE/S09/old/sca_report.json` | `EVIDENCE/sast-YYYY-MM-DD.*#77` | - |
| stripe-access token | Gitleaks | High | open | backlog | `EVIDENCE/S10/old/gitleaks.json` | [link] (https://github.com/alanprawira/secdev-seed-s09-s12/actions/runs/18880010884/job/53880167651) | Critical; owner: Alanprawira; expiration date: 2025-10-28 |
---

> Для «2» по DS5 обязательно указывать **owner/expiry/обоснование** для подавлений.

---

##7) Text "before/after" (metrics) (DS4/DS5)

|Control/Measure | Yandex.Metrica | Before | After | Proof (before), (after) |
|---------------|-------------------------|-----:|------:|-------------------------------------------------|
| Politics/MAC | Recognition | 17 | 15 | `EVIDENCE/S12/old/checkov.txt `, `EVIDENCE/S12/checkov.txt ` |
| Network users | The average flow | 3 | 0 | `EVIDENCE/S09/old/sca_summary.md`, `EVIDENCE/S09/sca_summary.md' |

---

## 8) Связь с TM и DV (сквозная нитка)

- **Закрываемые угрозы из TM:** TODO: T-001, T-005, … (ссылки на таблицу трассировки TM)
- **Связь с DV:** TODO: какие сканы/проверки встроены или будут встраиваться в pipeline

---

## 9) Out-of-Scope

- TODO: - **Production Infrastructure**: Only local Docker containers were scanned, not production cloud infrastructure (AWS/Azure/GCP)
- **Third-party APIs**: External service integrations and API endpoints were not included in DAST scanning  
- **Mobile/Client-side**: Security focus was on backend API only, not frontend/mobile applications
- **Performance/Load Testing**: Security scanning focused on vulnerabilities, not performance under load

---

- **DS1. SBOM and SCA:** [ ] 0 [ ] 1 [x] 2  
- **DS2. SAST + Secrets:** [ ] 0 [ ] 1 [x] 2  
- **DS3. DAST or Policy (Container/IaC):** [ ] 0 [ ] 1 [x] 2  
- **DS4. Mining (provable):** [ ] 0 [ ] 1 [x] 2  
- **DS5. Quality-gates, triage and "before/after":** [ ] 0 [ ] 1 [x] 2  

**Total DS (sum):** 10/10
