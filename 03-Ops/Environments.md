# Environments
tags: #ops #spec
status: Draft

## Environments
- Dev (local)
- Staging (prod-like)
- Prod

## Split per environment
- DATABASE_URL
- OAuth credentials + redirect URIs
- SESSION_SECRET
- TOKEN_ENCRYPTION_KEY
- FRONTEND_URL / API_BASE_URL

## Rule
Never share DB or OAuth credentials between staging and prod.
