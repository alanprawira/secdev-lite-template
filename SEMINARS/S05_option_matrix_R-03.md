# S05 - Матрица вариантов - R-03

## 0) Контекст риска (из S04)

* **Risk-ID:** `R-03`
* **Threat:** `I` (Information Disclosure)
* **DFD element/edge:** `API Controller Logs, Auth Service Error Handling`
* **NFR link (ID):** `NFR-003`, `NFR-004`
* **L×I (1-5):** `L=4, I=4, Score=16`
* **Ограничения/предпосылки:** GDPR compliance requirements; existing logging infrastructure; need for debugging capability.

---

## 2) Таблица сравнения вариантов

| Alternative | Summary | Security impact (↑,1-5) | Blast radius reduction (↑,1-5) | Complexity (↓,1-5) | Time-to-mitigate (↓,1-5) | Dependencies (↓,1-5) | **Benefit** | **Cost** | **Net** | Notes |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| A | Structured Logging with PII Masking + RFC7807 Errors | 4 | 4 | 2 | 2 | 2 | **8** | **6** | **+2** | Comprehensive coverage of logs and errors |
| B | Log Redaction Service (post-processing) | 3 | 3 | 4 | 4 | 4 | **6** | **12** | **-6** | Complex, data already exposed in logs |
| C | Selective Logging (disable sensitive endpoints) | 2 | 2 | 3 | 1 | 1 | **4** | **5** | **-1** | Loses observability for critical flows |

---

## 4) Решение (для переноса в ADR)

* **Chosen alternative:** `A`
* **Почему:** Provides direct PII protection at the source with reasonable complexity. Ensures compliance while maintaining observability.
* **ADR candidate (название):** `PII Protection in Logs & Errors`
* **Связки:** Risk-ID `R-03`, NFR-ID `NFR-003, NFR-004`, DFD `API Controller, Auth Service`
* **Следующие шаги (минимум):**
    1. Implement PII masking middleware for all logs
    2. Configure RFC7807 error format per NFR-003
    3. Establish data retention policy (30 days) per NFR-004
    4. Create audit process for PII compliance