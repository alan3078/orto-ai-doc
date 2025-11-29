# FN/FE/ONB/001 - Tasks

## Subtasks

### FN/FE/ONB/001-01: Create Terms of Service page
- [ ] Create /terms page with TOS content
- [ ] Add sections: Usage, Privacy, Data, Liability
- [ ] Style with proper typography
- [ ] Add print-friendly version
- [ ] Add last updated date

### FN/FE/ONB/001-02: Implement agreement acceptance flow
- [ ] Add tosAcceptedAt field to User model
- [ ] Create /onboarding/terms page
- [ ] Display TOS with scrollable content
- [ ] Add acceptance checkbox
- [ ] Save acceptance timestamp
- [ ] Redirect to next step

### FN/FE/ONB/001-03: Add first-login detection
- [ ] Add mustResetPassword field to User model
- [ ] Set true for newly created users
- [ ] Add middleware to check onboarding status
- [ ] Redirect to onboarding if incomplete
- [ ] Skip onboarding for completed users

### FN/FE/ONB/001-04: Build password reset wizard
- [ ] Create /onboarding/password page
- [ ] Build password change form
- [ ] Add current password field (optional for first login)
- [ ] Add new password with strength indicator
- [ ] Add confirm password field
- [ ] Validate password requirements
- [ ] Update password and clear mustResetPassword flag
- [ ] Redirect to dashboard

### FN/FE/ONB/001-05: Integration testing
- [ ] Test new user complete flow
- [ ] Test returning user bypass
- [ ] Test password validation
- [ ] Test redirect protection
- [ ] Test session persistence through flow
