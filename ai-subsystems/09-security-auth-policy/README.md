# 09 Security Auth Policy

Purpose: handle auth, policy limits, managed settings, privacy level, and security review boundaries.

Approximate source size: medium.

## Source Paths

- `src/utils/auth.ts`
- `src/utils/privacyLevel.ts`
- `src/utils/sessionIngressAuth.ts`
- `src/utils/sanitization.ts`
- `src/utils/mtls.ts`
- `src/services/api/client.ts`
- `src/services/api/errors.ts`
- `src/services/api/errorUtils.ts`
- `src/services/api/withRetry.ts`
- `src/services/oauth/*`
- `src/services/policyLimits/*`
- `src/services/remoteManagedSettings/*`
- `src/services/settingsSync/*`
- `src/security-review.ts`
- `src/review.ts`
- `src/commit-push-pr.ts`

## Implement

- Auth state model.
- API client wrapper and retry/error policy.
- Privacy/security policy boundary.
- Managed settings sync adapter.
- OAuth boundary.
- Security review command/service placeholder.

## Required Placeholders

- `TODO_SUBSYSTEM(09-security-auth-policy): secure credential storage`
- `TODO_SUBSYSTEM(09-security-auth-policy): real OAuth provider config`
- `TODO_SUBSYSTEM(09-security-auth-policy): enterprise managed settings source`
- `TODO_SUBSYSTEM(09-security-auth-policy): security scanner backend`
- `TODO_SUBSYSTEM(09-security-auth-policy): git hosting provider adapter`

## Public Contract

Expose:

- `AuthController`
- `ApiClient`
- `PolicyController`
- `getPrivacyLevel()`
- `runSecurityReview(input)`

