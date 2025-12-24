# Auth
tags: #spec
status: Draft

## Requirements
- Real sessions (cookie-based preferred)
- /auth/me for frontend bootstrap
- Account linking across identities
- No stubbed user in staging/prod

## Security
- HttpOnly cookies
- SameSite=Lax
- Secure in prod
