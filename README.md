# README - CXone Guide Demo Page Language Selection

## Overview

This demo page includes NICE CXone Guide and Chat integration with support for both English and Spanish experiences.

A language selector has been added to the page that allows users to switch between:

- English (`en-US`)
- Spanish (`es-US`)

The selected language is used to:

1. Set the CXone Guide locale.
2. Set the CXone Chat locale.
3. Populate the custom CXone field `language_select`.
4. Persist the user's language preference using browser local storage.

---

## Language Selector

A language dropdown is displayed in the page header.

```html
<select id="languageSelect">
  <option value="en-US">English</option>
  <option value="es-US">Español</option>
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

The value is then applied to both Guide and Chat.

```javascript
cxone('guide', 'init', {
  locale: selectedLanguage
});

cxone('chat', 'setLocale', selectedLanguage);
```

---

## Custom CXone Field

The page populates the custom field:

```text
language_select
```

using the currently selected language.

Example values:

```text
en-US
es-US
```

Configuration:

```javascript
cxone('chat', 'setCustomField',
  'language_select',
  selectedLanguage
);
```

### CXone Custom Field Definition

The following custom field must exist in the CXone business unit:

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

---

## Current Language Logic

| User Selection | Guide Locale | Chat Locale | Custom Field Value |
|---------------|-------------|-------------|-------------------|
| English | en-US | en-US | en-US |
| Spanish | es-US | es-US | es-US |

---

## Browser Locale Detection (Optional)

If desired, the default language can be determined using the visitor's browser locale instead of always defaulting to English.

Example:

```javascript
const browserLanguage =
  navigator.language.startsWith('es')
    ? 'es-US'
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

## Future Enhancements

If additional languages are required:

1. Add a new option to the language dropdown.
2. Add the language as a valid value in the CXone custom field.
3. Ensure Guide and Chat translations exist for the selected locale.
4. Update any Studio scripts, Guide workflows, bots, or routing logic that consume `language_select`.

Example:

```html
<option value="fr-FR">Français</option>
```

---

## CXone Dependencies

This implementation assumes:

- CXone Guide is enabled and configured.
- CXone Chat localization is enabled.
- Spanish translations exist within Guide.
- The custom field `language_select` has been created in CXone.
- Any Studio script, bot workflow, or routing logic consuming the value expects locale values in the format:

```text
en-US
es-US
```

---

## Files Modified

The following enhancements were made to the original demo page:

- Added language selector dropdown.
- Added Guide locale initialization.
- Added Chat locale initialization.
- Added custom field population (`language_select`).
- Added language persistence using localStorage.
- Added automatic page reload when a language selection changes.
- Added support for optional browser locale detection.

This allows a single HTML page to support both English and Spanish users without maintaining separate page versions.

---

## Testing

Verify the following scenarios during implementation:

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
   - `language_select = es-US`.

### Persistence

1. Select a language.
2. Refresh the page.
3. Confirm the selected language remains active.

---

## Support

If modifications are made to language values, Guide localization, Studio routing, or bot integrations, ensure any references to the custom field below remain synchronized:

```text
language_select
```

Expected values:

```text
en-US
es-US
```
