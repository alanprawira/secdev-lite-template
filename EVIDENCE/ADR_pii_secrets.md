# ADR — "PII Masking + Redaction Filter"

**Status:** Proposed  
**Owner:** DevSecOps Lead  
**Date created:** 2025-10-13  

---

## Context

Risk: R-03 (L=4, I=4, Score=16)
DFD: API Controller Logs, Auth Service Error Handling
NFR: NFR-003/NFR-004 (Privacy/PII, API-Contract/Errors)
Assumptions:

Все сервисы логируют JSON через единый middleware

Содержимое логов иногда включает поля DTO и stack traces

Требования GDPR: отсутствие PII/секретов в логах, хранение ≤30 дней

Необходимо сохранить возможность отладки через correlation_id


---

## Decision

Реализовать централизованную **маскировку PII в логах и ошибках**:
- **Param/Policy:** применить **redaction filter** на уровне лог-middleware (gateway, service)
- **Param/Policy:** шаблон denylist ключей → `["email","ip","token","password","verification_token"]`
- **Param/Policy:** значения заменяются на `"[REDACTED]"` до сериализации в JSON
- **Param/Policy:** ошибки возвращаются в формате RFC7807 без stack traces (error-mapper)
- **Param/Policy:** ротация логов и незавершённых регистраций ≤30 дней
- **Layer:** gateway, service; **scope:** все endpoints с DTO-логированием
- **Storage:** только обезличенные события с correlation_id для трассировки

---

## Alternatives

- **B – Log Redaction Service (post-processing):** отклонено — данные уже попадают в логи, создавая риск утечки
- **C – Selective Logging:** отклонено — теряется observability для критичных auth-потоков

---

## Consequences

**Положительные:**
- Соответствие GDPR и политике приватности (NFR-004)
- Снижение риска утечки PII через логи и ошибки (R-03)
- Единый контракт ошибок в формате RFC7807 (NFR-003)

**Негативные/издержки:**
- Усложнение отладки без raw PII данных
- Требуется поддерживать denylist в актуальном состоянии
- Дополнительные усилия на разработку и тестирование

---

## DoD / Acceptance

```gherkin
Given DTO содержит поля email, ip, token
When сервис записывает лог уровня INFO/ERROR
Then значения этих полей заменяются на "[REDACTED]", отсутствуют в теле лога

Given внутренняя ошибка при регистрации
When клиент получает ответ
Then Content-Type=application/problem+json, нет стэктрейсов, есть correlation_id

Given неподтверждённая учётная запись
When проходит 30 дней
Then запись и verification-токен удаляются согласно политике ретенции
```
Checks:

* test: unit-тесты проверяют маскировку всех sensitive-полей

* log: поиск по "email"/"ip" в прод-логах → 0 совпадений

* scan/policy: статический анализ подтверждает отсутствие PII в error-ответах

* metric/SLO: доля логов, прошедших redaction check ≥ 99%

* retention: job-логи показывают успешную очистку через 30 дней


Rollback / Fallback

* Флаг PII_MASKING_ENABLED=false в конфиге (временное отключение)

* Мониторинг через дашборд pii_redaction_ratio

* При сбоях возвращаемся к detailed logs в non-prod средах

* Отслеживание увеличения времени отладки или support-запросов

Trace
* DFD: S04.md → API Controller Logs, Auth Service Error Handling

* STRIDE: строка R-03 (Information Disclosure)

* Risk scoring: S04_risk_scoring.md → R-03, Top-3 (Score=16)

* NFR: NFR-003 (API-Contract/Errors), NFR-004 (Privacy/PII)

* Issue: #PRIV-001, #API-001

Open Questions
* Какие дополнительные поля считать PII помимо email, ip, token?

* Как обрабатывать сценарии отладки, требующие доступа к PII?

* Разная политика ретенции для audit-логов vs application-логов?
