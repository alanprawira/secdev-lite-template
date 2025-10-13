## ADR: **JWT TTL + Refresh + Rotation**

**Status**: Proposed

### **Context**

**Risk**: **R-04** "Слабая безопасность verification-токенов" (L=3, I=4, Score=12)
**DFD**: Database Token Storage → Auth Service
**NFR**: **NFR-004** (Privacy/PII, Data Retention)

**Assumptions**:
- Stateless authentication service architecture
- Tokens transmitted over HTTPS only
- No client-side MTLS implementation
- Existing JWT-based authentication flow

### **Decision**

Для защиты от компрометации verification и authentication токенов, а также предотвращения атак повторного использования, внедряем комплексную стратегию управления токенами:

* **Access Token TTL**: Установить на **15 минут** для минимизации окна атаки
* **Refresh Token TTL**: Установить на **7 дней** с secure storage (HttpOnly, Secure flags)
* **Key Rotation**: Автоматическая ротация JWT signing keys каждые **90 дней** через JWKS endpoint
* **Token Validation**: Строгая проверка claims (`iss`, `aud`, `exp`, `iat`) с clock skew ±60 секунд
* **Revocation Mechanism**: Поддержка blacklist для refresh tokens при logout/security events
* **Verification Token Security**: Генерация high-entropy tokens (32+ bytes) с TTL 24 часа

**Область применения**:
* **Эндпойнты**: Все protected endpoints, `/api/auth/refresh`, email verification flows
* **Уровень**: API Gateway validation + Auth Service token management
* **Токены**: Access tokens, Refresh tokens, Email verification tokens

### **Alternatives**

* **Alt B**: Long-lived tokens with database revocation checks
  * **Почему не выбрали**: Вводит statefulness, увеличивает нагрузку на БД, снижает производительность
* **Alt C**: Session-based authentication with server-side storage
  * **Почему не выбрали**: Противоречит stateless архитектуре, сложнее масштабировать

Выбранное решение обеспечивает баланс между безопасностью и производительностью, соответствуя принципам stateless архитектуры.

### **Consequences**

**Положительные эффекты**:
+ Значительное снижение риска компрометации учетных записей
+ Соответствие best practices для JWT безопасности
+ Минимизация ущерба при компрометации токенов

**Отрицательные эффекты**:
- Усложнение клиентской логики для обработки refresh flow
- Дополнительная нагрузка на мониторинг и rotation процессов
- Potential compatibility issues with legacy clients

### **DoD / Acceptance**

**Given** истёкший access token с `exp` в прошлом,
**When** выполняется запрос к защищенному эндпойнту (`POST /api/profile`),
**Then** ответ **401 Unauthorized** с RFC7807 форматом и `correlation_id`

**Given** валидный refresh token,
**When** вызывается `/api/auth/refresh` до истечения TTL,
**Then** возвращается новый access token с обновленным `exp`

**Checks**:
* **Test**: Integration tests verify 401 responses for expired tokens and successful refresh flow
* **Log**: Audit logs contain token validation events with `correlation_id`
* **Scan**: Security scan confirms no hardcoded JWT secrets in code
* **Metric**: Token validation error rate < 1% of total authentication requests

### **Rollback / Fallback**

* **Rollback**: Увеличить TTL access tokens до предыдущих значений через конфигурацию
* **Fallback**: Временное отключение strict claim validation при критических инцидентах
* **Monitoring**: Отслеживать увеличение 401 ошибок и failed refresh attempts

### **Trace**

* **DFD**: Storage → Auth Service (token flows)
* **STRIDE**: R-04 из S04_risk_scoring.md
* **Risk scoring**: R-04 (Top-4, Score=12)
* **NFR**: NFR-004 из S03 реестра
* **Issues**: #SEC-004 (Token Security Implementation)

### **Ownership & Dates**

* **Owner**: Backend Team
* **Reviewers**: Security Lead, DevSecOps
* **Date created**: 2024-06-15
* **Last updated**: 2024-06-15

### **Open Questions**

- Как обрабатывать активные сессии при rotation signing keys?
- Каков процесс emergency key rotation при компрометации?

### **Appendix**

```json
{
  "access_token_ttl": "15m",
  "refresh_token_ttl": "7d",
  "key_rotation_interval": "90d",
  "verification_token_ttl": "24h"
}
