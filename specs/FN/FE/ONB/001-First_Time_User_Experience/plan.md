# FN/FE/ONB/001 - First-Time User Experience

> **V1.0 Requirement #12**: 1st time user experience - user agreement + user password reset

## Overview

Implement onboarding flow for new users including terms acceptance and mandatory password reset on first login.

## Goals

1. Create Terms of Service page
2. Implement agreement acceptance flow
3. Add first-login detection
4. Build password reset wizard
5. Integration testing

## Scope

### In Scope
- Terms of Service page content
- Agreement checkbox and timestamp tracking
- First-login flag on User model
- Forced password change on first login
- Password strength requirements
- Redirect flow until onboarding complete

### Out of Scope
- Privacy policy separate page
- Cookie consent banner
- Multi-step onboarding wizard
- Product tour

## Success Criteria

- [ ] New user sees Terms of Service on first login
- [ ] User must accept terms to proceed
- [ ] User must change password on first login
- [ ] Password meets strength requirements
- [ ] Onboarding status tracked per user
- [ ] Cannot access app until onboarding complete

## Dependencies

- FN/ADM/AUTH/001 (User Account Types) - **REQUIRED**

## User Flow

```
Login → First Login Check → Terms Acceptance → Password Reset → Dashboard
                ↓
        Returning User → Dashboard
```
