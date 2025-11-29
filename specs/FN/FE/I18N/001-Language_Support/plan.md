# FN/FE/I18N/001 - Language Support

> **V1.0 Requirement #11**: Language Support: CN / TC / EN

## Overview

Implement multi-language support for the application with English, Simplified Chinese, and Traditional Chinese.

## Goals

1. Setup next-intl and locale routing
2. Create translation files
3. Build language switcher component
4. Migrate UI strings to translation keys
5. Add locale-aware date/time formatting

## Scope

### In Scope
- Locales: en (English), zh-CN (Simplified Chinese), zh-TW (Traditional Chinese)
- Static UI text translations
- Date/time format localization
- Number format localization
- Language preference persistence
- Server-side locale detection

### Out of Scope
- Dynamic content translation
- Right-to-left (RTL) support
- Translation management UI
- Machine translation integration

## Success Criteria

- [ ] User can switch between EN/CN/TC
- [ ] All static UI text translated
- [ ] Dates displayed in locale format
- [ ] Language preference persisted
- [ ] SEO-friendly locale URLs (/en, /zh-CN, /zh-TW)

## Technical Stack

- **Library**: next-intl (recommended for Next.js App Router)
- **Routing**: Middleware-based locale detection
- **Persistence**: Cookie + localStorage
- **Format**: JSON translation files

## File Structure

```
src/
├── i18n/
│   ├── config.ts
│   ├── request.ts
│   └── messages/
│       ├── en.json
│       ├── zh-CN.json
│       └── zh-TW.json
├── middleware.ts (locale routing)
└── app/
    └── [locale]/
        └── layout.tsx
```
