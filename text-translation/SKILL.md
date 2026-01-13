---
name: text-translation
description: Trigger for repo i18n work when the user asks to add/update translations or translation keys (locale JSON edits), and/or to replace hardcoded UI strings with `t('...')`/fix missing keys. Do NOT use for one-off plain-language “translate this text” requests when no code/locale files need changes.
---

## Quick workflow

1. Find the project translation hook (often `useTranslation`, `useI18n`, or `useIntl`).
2. Replace UI strings with translation keys (e.g., `t('credit.instant_transfer')`).
3. Ensure the key exists in every locale JSON; if it’s missing, add it with the translated value (don’t leave it falling back to the raw key).
4. If the string needs variables, use interpolation (commonly `{{name}}`, `{{amount}}`).

## Where to find the file

Locale dictionaries can be named/located differently per project (e.g., `localeEn.json`, `en.json`, `en-US.json`, `i18n/en.json`). Search for the translation provider/loading code (often `setLocale(...)`) to confirm.

## Examples

```ts
// React Native
const { t } = useTranslation();
<Text>{t('today')}</Text>
<Text>{t('welcome_user', { name: userName })}</Text>

// React Web
const { t } = useTranslation()
<p>{t('tomorrow')}</p>
<span>{t('next_month', { name: userName })}</span>
```
