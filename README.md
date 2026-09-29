# README - CXone Guide Demo Page Language Selection

## Overview

This demo page includes NICE CXone Guide and Chat integration with support for both English and Spanish experiences.

A language selector has been added to the page that allows users to switch between:

- English (`en-US`)
- Spanish (`es-ES`)

The selected language is used to:

1. Set the CXone Guide locale.
2. Set the CXone Chat locale.
3. Populate the CXone custom field `language_select`.
4. Persist the user's language preference using browser local storage.

> **Important:** The locale used by CXone Guide and Chat for Spanish is `es-ES`, while the custom field value sent to CXone is `es-US`. This is intentional and required for this implementation.

---

## Language Selector

A language dropdown is displayed in the page header.

```html
<select id="languageSelect">
  <option value="en-US">English</option>
  <option value="es-ES">Español</option>
</select>
```

When a selection is made:

1. The value is saved to localStorage.
2. The page reloads.
3. CXone initializes using the selected language.

---

## CXone Configuration

The page reads the saved language preference during initialization.

```javascript
const selectedLanguage =
  localStorage.getItem('demoLanguage') || 'en-US';
```

The default language is:

```javascript
en-US
```

if no language has been selected.

The selected language is then applied to both Guide and Chat.

```javascript
cxone('guide', 'init', {
  locale: selectedLanguage
});

cxone('chat', 'setLocale', selectedLanguage);
```

### Language Mapping

The page uses a separate value when populating the CXone custom field.

```javascript
const languageCustomFieldValue =
  selectedLanguage === 'es-ES'
    ? 'es-US'
    : 'en-US';
```

This allows CXone Guide and Chat to use the proper localization locale (`es-ES`) while still sending the expected custom field value (`es-US`) to Studio, routing logic, or integrations.

---

## Custom CXone Field

The page populates the custom field:

```text
language_select
```

using the mapped language value.

Configuration:

```javascript
cxone('chat', 'setContactCustomField',
  'language_select',
  languageCustomFieldValue
);
```

### Values Sent to CXone

| Selected Language | Custom Field Value |
|------------------|-------------------|
| English | en-US |
| Spanish | es-US |

### CXone Custom Field Definition

The following custom field must exist in the CXone business unit.

**Ident**

```text
language_select
```

**Type**

```text
Dropdown
```

**Values**

```text
en-US | English
es-US | Spanish
```

This field can be referenced by Studio scripts, routing logic, Guide workflows, reporting, or bot integrations.

---

## Browser Local Storage

The selected language is stored locally using:

```javascript
localStorage.setItem(
  'demoLanguage',
  selectedLanguage
);
```

**Storage Key**

```text
demoLanguage
```

This allows the user's language preference to persist across page refreshes.

Stored values:

```text
en-US
es-ES
```

---

## Current Language Logic

| User Selection | Guide Locale | Chat Locale | Custom Field Value |
|---------------|-------------|-------------|-------------------|
| English | en-US | en-US | en-US |
| Spanish | es-ES | es-ES | es-US |

---

## Browser Locale Detection (Optional)

If desired, the default language can be determined using the visitor's browser locale instead of always defaulting to English.

Example:

```javascript
const browserLanguage =
  navigator.language.startsWith('es')
    ? 'es-ES'
    : 'en-US';

const selectedLanguage =
  localStorage.getItem('demoLanguage') ||
  browserLanguage;
```

With this approach:

- Spanish browsers automatically load Spanish.
- All other browsers default to English.
- User selections continue to override the default through localStorage.

---



## CXone Dependencies

This implementation assumes:

- CXone Guide is enabled and configured.
- CXone Chat localization is enabled.
- Spanish translations exist within Guide.
- The custom field `language_select` has been created in CXone.
- Any Studio script, bot workflow, or routing logic consuming the value expects:

```text
en-US
es-US
```

- The CXone Guide locale for Spanish is configured as:

```text
es-ES
```

---

## Files Modified

The following enhancements were made to the original demo page:

- Added language selector dropdown.
- Added Guide locale initialization.
- Added Chat locale initialization.
- Added custom field population (`language_select`).
- Added language-to-custom-field mapping logic.
- Added language persistence using localStorage.
- Added automatic page reload when a language selection changes.
- Added support for optional browser locale detection.

This allows a single HTML page to support both English and Spanish users without maintaining separate page versions.

---

## Testing

Verify the following scenarios during implementation.

### English

1. Load the page.
2. Select **English**.
3. Open CXone Guide.
4. Confirm:
   - Guide displays in English.
   - Chat displays in English.
   - `language_select = en-US`.

### Spanish

1. Load the page.
2. Select **Español**.
3. Open CXone Guide.
4. Confirm:
   - Guide displays in Spanish.
   - Chat displays in Spanish.
   - Guide requests use `locale=es-ES`.
   - `language_select = es-US`.

### Persistence

1. Select a language.
2. Refresh the page.
3. Confirm the selected language remains active.

### Developer Validation

When Spanish is selected, verify browser Developer Tools show Guide requests similar to:

```text
.../configuration?locale=es-ES
```

and not:

```text
.../configuration?locale=es-US
```

---

## Support

If modifications are made to language values, Guide localization, Studio routing, or bot integrations, ensure references to the custom field below remain synchronized:

```text
language_select
```

Expected custom field values:

```text
en-US
es-US
```

Expected CXone localization values:

```text
en-US
es-ES
```
