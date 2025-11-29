# FN/FE/I18N/001 - Tasks

## Subtasks

### FN/FE/I18N/001-01: Setup next-intl and locale routing
- [ ] Install next-intl package
- [ ] Create i18n configuration file
- [ ] Setup middleware for locale detection
- [ ] Configure locale-based routing ([locale] segment)
- [ ] Add locale provider to root layout
- [ ] Test basic routing (/en, /zh-CN, /zh-TW)

### FN/FE/I18N/001-02: Create translation files (en, zh-CN, zh-TW)
- [ ] Create messages folder structure
- [ ] Create en.json with all UI strings
- [ ] Create zh-CN.json (Simplified Chinese)
- [ ] Create zh-TW.json (Traditional Chinese)
- [ ] Organize keys by feature/page
- [ ] Add common keys (buttons, labels, errors)

### FN/FE/I18N/001-03: Build LanguageSwitcher component
- [ ] Create LanguageSwitcher dropdown component
- [ ] Display current locale with flag/name
- [ ] List available locales
- [ ] Handle locale change with router.replace
- [ ] Add to header/navigation
- [ ] Style for mobile responsiveness

### FN/FE/I18N/001-04: Migrate all UI strings to translation keys
- [ ] Audit all hardcoded strings in components
- [ ] Replace with useTranslations hook
- [ ] Update server components with getTranslations
- [ ] Handle dynamic values with interpolation
- [ ] Test all pages in each locale

### FN/FE/I18N/001-05: Add locale-aware date/time formatting
- [ ] Configure date-fns locale imports
- [ ] Create useFormattedDate hook
- [ ] Update all date displays to use formatter
- [ ] Configure number formatting
- [ ] Test calendar/picker components
