# CXone Guide Demo Page Language Selection

## Overview

This demo page includes NICE CXone Guide and Chat integration with support for English and Spanish experiences.

A language selector in the page header allows users to switch between:

- English (`en-US`)
- Spanish (`es-ES`)

The selected language is used to:

1. Set the CXone Guide locale.
2. Set the CXone Chat locale.
3. Populate the CXone contact custom field `language_select`.
4. Persist the user's language preference using browser local storage.

> **Important:** CXone Guide and Chat use `es-ES` for the Spanish widget locale. The `language_select` contact custom field uses `es-US` for Spanish. This distinction is intentional and required for the Studio and integration logic used by this implementation.

---

## Language Selector

The language dropdown is displayed in the page header.

```html
<select id="languageSelect" aria-label="Select language">
  <option value="en-US">English</option>
  <option value="es-ES">Español</option>
</select>
```

When the user changes the selected language:

1. The selected locale is saved to browser local storage.
2. The page reloads.
3. CXone Guide and Chat initialize using the selected locale.
4. The corresponding mapped value is assigned to the `language_select` contact custom field.

---

## Supported Languages

The page limits the saved language value to the supported locales:

```javascript
const supportedLanguages = [
  'en-US',
  'es-ES'
];
```

The saved language is read during page initialization:

```javascript
let selectedLanguage =
  localStorage.getItem('demoLanguage') || 'en-US';
```

If the saved value is not supported, the page resets it to English:

```javascript
if (!supportedLanguages.includes(selectedLanguage)) {
  selectedLanguage = 'en-US';
  localStorage.setItem('demoLanguage', selectedLanguage);
}
```

The default language is:

```text
en-US
```

---

## CXone Initialization

The CXone loader is initialized using the tenant-specific GovCloud loader URL:

```javascript
(function (n, u) {
  window.CXoneDfo = n;

  window[n] = window[n] || function () {
    (window[n].q = window[n].q || []).push(arguments);
  };

  window[n].u = u;

  const script = document.createElement('script');
  script.type = 'module';
  script.src =
    u + '?' + Math.round(Date.now() / 1000 / 3600);

  document.head.appendChild(script);
})(
  'cxone',
  'https://web-modules-de-na2.nicecxone-gov.com/loader/1/loader.js'
);
```

CXone is then initialized using tenant or brand ID `1209`:

```javascript
cxone('init', '1209');
```

---

## Guide and Chat Locale

The selected language is applied separately to CXone Guide and Chat:

```javascript
cxone('guide', 'init', {
  locale: selectedLanguage
});

cxone('chat', 'setLocale', selectedLanguage);
```

Expected localization values:

```text
en-US
es-ES
```

When Spanish is selected, both the Guide configuration and Chat interface receive:

```text
es-ES
```

The `es-US` value is not used to initialize Guide or Chat.

---

## HTML Document Language

The page also updates the HTML document language based on the selected locale:

```javascript
document.documentElement.lang =
  selectedLanguage === 'es-ES'
    ? 'es'
    : 'en';
```

This helps keep the document language synchronized with the selected CXone experience.

---

## Language Custom Field Mapping

The value required by Studio differs from the locale used by the CXone widget.

The page maps the selected widget locale to the expected contact custom field value:

```javascript
const languageCustomFieldValue =
  selectedLanguage === 'es-ES'
    ? 'es-US'
    : 'en-US';
```

This produces the following mapping:

| User Selection | Guide Locale | Chat Locale | Contact Custom Field |
|---|---|---|---|
| English | `en-US` | `en-US` | `en-US` |
| Spanish | `es-ES` | `es-ES` | `es-US` |

---

## CXone Contact Custom Field

The page populates the following CXone contact custom field:

```text
language_select
```

The field is populated before the chat begins using:

```javascript
cxone(
  'chat',
  'setContactCustomField',
  'language_select',
  languageCustomFieldValue
);
```

This is an interaction-level contact custom field. Studio, routing logic, reporting, or connected integrations can access the value associated with the newly created chat interaction.

### Expected Values

```text
en-US
es-US
```

### Important Distinction

The following values serve different purposes:

```text
es-ES = CXone Guide and Chat localization
es-US = language_select contact custom field
```

Do not pass `es-US` to `setLocale()` or the Guide initialization.

Do not pass `es-ES` to the `language_select` contact custom field unless the downstream Studio or integration logic is also updated to expect that value.

---

## CXone Custom Field Definition

The contact custom field must exist in the applicable CXone business unit.

### Ident

```text
language_select
```

### Type

```text
Dropdown
```

### Allowed Values

```text
en-US | English
es-US | Spanish
```

The field ident and allowed dropdown values must match the values used by the page exactly.

The implementation expects the field to be a contact custom field associated with the interaction, not a customer-card custom field.

---

## Initialization Order

The custom field is set as part of the initial CXone configuration, before the user opens or starts the chat.

The relevant order is:

```javascript
cxone('init', '1209');

cxone('guide', 'init', {
  locale: selectedLanguage
});

cxone('chat', 'setLocale', selectedLanguage);

cxone(
  'chat',
  'setContactCustomField',
  'language_select',
  languageCustomFieldValue
);
```

Avoid setting the custom field again inside the function that opens the Guide widget. Reassigning the field while Guide is opening the chat can result in inconsistent initialization behavior.

---

## Opening the Guide Widget

The custom page button opens the existing CXone Guide channel button:

```javascript
function openCxoneGuide() {
  const guideButton = document.querySelector(
    '#cxone-guide-container ' +
    '[data-selector="GUIDE_CHANNEL_BUTTON"]'
  );

  if (guideButton) {
    guideButton.click();
  } else {
    console.error(
      'CXone Guide channel button was not found.'
    );
  }
}
```

The page button uses:

```html
<button
  type="button"
  class="btn"
  onclick="openCxoneGuide()"
>
  Chat with Support
</button>
```

The `openCxoneGuide()` function does not modify the locale or custom field. Those values are configured earlier during page initialization.

---

## Browser Local Storage

The selected widget locale is stored in browser local storage.

### Storage Key

```text
demoLanguage
```

### Stored Values

```text
en-US
es-ES
```

The language dropdown saves the selected locale and reloads the page:

```javascript
const languageSelect =
  document.getElementById('languageSelect');

languageSelect.value = selectedLanguage;

languageSelect.addEventListener(
  'change',
  function () {
    if (!supportedLanguages.includes(this.value)) {
      return;
    }

    localStorage.setItem(
      'demoLanguage',
      this.value
    );

    window.location.reload();
  }
);
```

The value stored in local storage is the widget locale, not the mapped custom-field value.

Therefore, Spanish is stored as:

```text
es-ES
```

The page maps it to `es-US` only when setting the contact custom field.

---

## Optional Browser Locale Detection

The current implementation defaults to English when no saved selection exists.

If browser locale detection is required, the default can be updated as follows:

```javascript
const browserLanguage =
  navigator.language.startsWith('es')
    ? 'es-ES'
    : 'en-US';

let selectedLanguage =
  localStorage.getItem('demoLanguage') ||
  browserLanguage;
```

With this optional approach:

- Browsers configured for Spanish default to `es-ES`.
- Other browsers default to `en-US`.
- A saved user selection overrides the browser default.
- The Spanish contact custom field still maps to `es-US`.

---

## Debug Logging

The page includes temporary console logging for language validation:

```javascript
console.log(
  'CXone widget locale:',
  selectedLanguage
);

console.log(
  'CXone language_select value:',
  languageCustomFieldValue
);
```

When English is selected, the browser console should show:

```text
CXone widget locale: en-US
CXone language_select value: en-US
```

When Spanish is selected, the browser console should show:

```text
CXone widget locale: es-ES
CXone language_select value: es-US
```

These logs confirm the values calculated by the page. They can be removed after implementation testing is complete.

---

## CXone Dependencies

This implementation assumes:

- CXone Guide is enabled and configured.
- CXone Chat localization is enabled.
- Spanish translations are configured and published for Guide.
- The Spanish Guide and Chat locale is `es-ES`.
- The contact custom field `language_select` exists in CXone.
- The field ident is exactly `language_select`.
- The field accepts `en-US` and `es-US` as dropdown values.
- Studio or downstream integration logic expects `en-US` or `es-US`.
- The page uses tenant or brand ID `1209`.
- The page uses the appropriate CXone GovCloud loader URL.

---

## Implementation Changes

The following enhancements were made to the original demo page:

- Added an English and Spanish language selector.
- Added supported-language validation.
- Added CXone Guide locale initialization.
- Added CXone Chat locale initialization.
- Added HTML document language synchronization.
- Added `es-ES` to `es-US` language mapping.
- Added contact custom-field population through `setContactCustomField`.
- Added language persistence using local storage.
- Added an automatic page reload when the language changes.
- Removed duplicate locale and custom-field calls.
- Removed custom-field reassignment from `openCxoneGuide()`.
- Moved the language-selection script outside the footer content.
- Added temporary console logging for validation.

This allows one HTML page to support English and Spanish CXone experiences without maintaining separate page versions.

---

## Testing

Testing should use a new chat interaction. An existing interaction may retain values that were assigned when the interaction was originally created.

For the cleanest test, use a new InPrivate or Incognito browser session.

### English Test

1. Open the page in a new browser session.
2. Select **English**.
3. Allow the page to reload.
4. Open CXone Guide.
5. Start a new chat.
6. Send a message.
7. Confirm:
   - Guide displays in English.
   - Chat displays in English.
   - The console shows `en-US` for the widget locale.
   - The console shows `en-US` for `language_select`.
   - Studio receives `language_select = en-US`.

### Spanish Test

1. Open the page in a new browser session.
2. Select **Español**.
3. Allow the page to reload.
4. Open CXone Guide.
5. Start a new chat.
6. Send a message.
7. Confirm:
   - Guide displays in Spanish.
   - Chat displays in Spanish.
   - The console shows `es-ES` for the widget locale.
   - The console shows `es-US` for `language_select`.
   - Studio receives `language_select = es-US`.

### Persistence Test

1. Select a language.
2. Refresh the page.
3. Confirm the selected language remains active.
4. Confirm the dropdown displays the saved selection.
5. Confirm Guide and Chat use the saved locale.

### Developer Tools Validation

When Spanish is selected, Guide requests should use:

```text
locale=es-ES
```

They should not use:

```text
locale=es-US
```

The browser console should separately show:

```text
CXone language_select value: es-US
```

---

## Troubleshooting

### Widget displays Spanish but Studio receives a blank field

Verify:

1. The field ident is exactly:

   ```text
   language_select
   ```

2. The field is configured as a contact custom field.

3. The field allows the underlying value:

   ```text
   es-US
   ```

4. The code uses:

   ```javascript
   cxone(
     'chat',
     'setContactCustomField',
     'language_select',
     languageCustomFieldValue
   );
   ```

5. The browser console shows:

   ```text
   CXone language_select value: es-US
   ```

6. The Studio script is reading the contact custom field associated with the current interaction.

7. The test uses a newly created chat interaction.

### Widget does not display Spanish

Verify:

1. The dropdown value is:

   ```text
   es-ES
   ```

2. Local storage contains:

   ```text
   demoLanguage = es-ES
   ```

3. Guide initializes with:

   ```javascript
   cxone('guide', 'init', {
     locale: selectedLanguage
   });
   ```

4. Chat receives:

   ```javascript
   cxone('chat', 'setLocale', selectedLanguage);
   ```

5. Spanish translations are configured and published in CXone Guide.

### Resetting the saved language

Run the following command in the browser console:

```javascript
localStorage.removeItem('demoLanguage');
```

Then reload the page.

---

## Support and Maintenance

If the language values, Guide localization, Studio routing, or bot integration are changed, update all references consistently.

### Contact Custom Field

```text
language_select
```

### Expected Custom Field Values

```text
en-US
es-US
```

### Expected CXone Localization Values

```text
en-US
es-ES
```

### Spanish Mapping

```text
Widget locale: es-ES
Contact custom field: es-US
```
