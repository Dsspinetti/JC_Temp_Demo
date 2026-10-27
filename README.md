# CXone Guide and Chat Language Integration

## Purpose

This implementation allows a single web page to support English and Spanish CXone Guide, CXone Chat, Studio, and Omilia routing behavior.

The key requirement is that the CXone widget locale and the downstream language value are intentionally different for Spanish:

- **CXone Guide and Chat locale:** `es-ES`
- **CXone contact custom field / Studio / Omilia value:** `es-US`

English uses `en-US` for both.

> **Important:** Do not use `es-US` to initialize CXone Guide or Chat. Do not send `es-ES` to the `language_select` contact custom field unless the downstream Studio and Omilia logic is updated to expect it.

---

## Language Mapping

| User selection | Guide locale | Chat locale | `language_select` value |
|---|---|---|---|
| English | `en-US` | `en-US` | `en-US` |
| Spanish | `es-ES` | `es-ES` | `es-US` |

The `language_select` contact custom field is the value used by the downstream Studio and Omilia logic to select the appropriate experience.

---

## Prerequisites

Before implementing the code, confirm that:

- CXone Guide is configured and published.
- English and Spanish Guide translations are configured and published.
- The CXone loader URL and tenant or brand ID are correct for the target environment.
- A CXone **contact custom field** exists with the ident `language_select`.
- The field accepts the values `en-US` and `es-US`.
- Studio and Omilia expect the same values.

### Contact custom field definition

- **Ident:** `language_select`
- **Type:** Dropdown
- **Expected values:**
  - `en-US` for English
  - `es-US` for Spanish

The ident and values must match the web implementation exactly.

---

## 1. Add the Language Selector

Add the language selector wherever it should appear on the page:

```html
<label for="languageSelect" class="visually-hidden">Select language</label>
<select id="languageSelect" aria-label="Select language">
  <option value="en-US">English</option>
  <option value="es-ES">Español</option>
</select>
```

The dropdown values represent the **CXone widget locales**, not the mapped value sent to Studio and Omilia.

---

## 2. Load CXone

Add the tenant-specific CXone loader in the page `<head>`:

```javascript
(function (n, u) {
  window.CXoneDfo = n;
  window[n] = window[n] || function () {
    (window[n].q = window[n].q || []).push(arguments);
  };
  window[n].u = u;

  const script = document.createElement('script');
  script.type = 'module';
  script.src = u + '?' + Math.round(Date.now() / 1000 / 3600);
  document.head.appendChild(script);
})(
  'cxone',
  'https://web-modules-de-na2.nicecxone-gov.com/loader/1/loader.js'
);
```

> Replace the loader URL if a different CXone environment or region is used.

---

## 3. Resolve and Validate the Selected Language

The implementation stores the user's widget locale in browser local storage. English is used when no supported selection has been saved.

```javascript
const supportedLanguages = ['en-US', 'es-ES'];
let selectedLanguage = localStorage.getItem('demoLanguage') || 'en-US';

if (!supportedLanguages.includes(selectedLanguage)) {
  selectedLanguage = 'en-US';
  localStorage.setItem('demoLanguage', selectedLanguage);
}
```

Only these widget locale values should be stored:

```text
en-US
es-ES
```

---

## 4. Map the Widget Locale to the Downstream Value

Map the selected widget locale before initializing CXone:

```javascript
const languageCustomFieldValue =
  selectedLanguage === 'es-ES' ? 'es-US' : 'en-US';
```

This is the required Spanish conversion:

```text
Widget locale: es-ES
Studio/Omilia value: es-US
```

---

## 5. Initialize CXone and Set the Language Values

Initialize CXone in the following order:

```javascript
document.documentElement.lang =
  selectedLanguage === 'es-ES' ? 'es' : 'en';

cxone('init', '1209');
cxone('guide', 'init', { locale: selectedLanguage });
cxone('chat', 'setLocale', selectedLanguage);

cxone(
  'chat',
  'setContactCustomField',
  'language_select',
  languageCustomFieldValue
);
```

> Replace `1209` if the target environment uses a different tenant or brand ID.

The custom field should be set during initial page configuration, before the user starts the chat. Do not set it again when opening the Guide widget.

---

## 6. Save Language Changes and Reload

After the page content has loaded, connect the dropdown to local storage:

```javascript
const languageSelect = document.getElementById('languageSelect');
languageSelect.value = selectedLanguage;

languageSelect.addEventListener('change', function () {
  if (!supportedLanguages.includes(this.value)) return;

  localStorage.setItem('demoLanguage', this.value);
  window.location.reload();
});
```

Reloading ensures that Guide, Chat, and the contact custom field are initialized together using the newly selected language.

The value stored in local storage remains the widget locale:

- English: `en-US`
- Spanish: `es-ES`

The Spanish value is mapped to `es-US` only when `language_select` is populated.

---

## 7. Open Guide from a Custom Page Button

If the page uses its own support button, it can open the existing CXone Guide channel button:

```javascript
function openCxoneGuide() {
  const guideButton = document.querySelector(
    '#cxone-guide-container [data-selector="GUIDE_CHANNEL_BUTTON"]'
  );

  if (guideButton) {
    guideButton.click();
  } else {
    console.error('CXone Guide channel button was not found.');
  }
}
```

Example page button:

```html
<button type="button" onclick="openCxoneGuide()">
  Chat with Support
</button>
```

This function only opens Guide. It should not change the locale or reassign the custom field.

---

## Optional Browser-Language Default

The current implementation defaults to English when no saved selection exists. To default Spanish-language browsers to Spanish, replace the initial language assignment with:

```javascript
const browserLanguage =
  navigator.language.toLowerCase().startsWith('es')
    ? 'es-ES'
    : 'en-US';

let selectedLanguage =
  localStorage.getItem('demoLanguage') || browserLanguage;
```

A saved user selection will continue to override the browser language.

---

## Validation and Testing

Use a new chat interaction for each test. Existing interactions may retain values assigned when the interaction was created. An InPrivate or Incognito session is recommended for clean testing.

### English test

1. Select **English**.
2. Allow the page to reload.
3. Open Guide and start a new chat.
4. Confirm Guide and Chat display in English.
5. Confirm Studio receives `language_select = en-US`.
6. Confirm Omilia uses the English experience.
7. End the chat through the ellipses in the header

### Spanish test

1. Select **Español**.
2. Allow the page to reload.
3. Open Guide and start a new chat.
4. Confirm Guide and Chat display in Spanish.
5. Confirm Studio receives `language_select = es-US`.
6. Confirm Omilia uses the Spanish experience.
7. End the chat through the ellipses in the header

### Persistence test

1. Select a language.
2. Refresh the page.
3. Confirm the dropdown retains the selection.
4. Confirm Guide and Chat use the saved widget locale.
5. Confirm the mapped custom-field value is correct for the new interaction.

### Temporary debug logging

The following logging can be used during implementation testing:

```javascript
console.log('CXone widget locale:', selectedLanguage);
console.log('CXone language_select value:', languageCustomFieldValue);
```

Expected Spanish output:

```text
CXone widget locale: es-ES
CXone language_select value: es-US
```

Remove the logs after validation if they are not needed in production.

---

## Troubleshooting

### Guide and Chat display Spanish, but Studio receives a blank value

Verify that:

- The contact custom field ident is exactly `language_select`.
- It is configured as a contact custom field, not a customer-card field.
- The field permits the value `es-US`.
- `setContactCustomField` runs before the chat begins.
- Studio reads the field from the current interaction.
- Testing is performed with a newly created chat interaction.

### Guide or Chat does not display Spanish

Verify that:

- The dropdown value is `es-ES`.
- Local storage contains `demoLanguage = es-ES`.
- Guide is initialized with `locale: selectedLanguage`.
- Chat receives `setLocale` with `selectedLanguage`.
- Spanish Guide translations are configured and published.
- The network requests use `locale=es-ES`, not `locale=es-US`.

### Omilia uses the wrong language

Verify that:

- Spanish is mapped from `es-ES` to `es-US` before setting the contact custom field.
- Studio receives `language_select = es-US`.
- The Studio or integration logic passes the expected value to Omilia.
- The Omilia language-routing logic expects `en-US` and `es-US`.

### Reset the saved selection

Run the following command in the browser console, then reload the page:

```javascript
localStorage.removeItem('demoLanguage');
```

---

## Implementation Notes

- Keep the loader URL and tenant or brand ID environment-specific.
- Keep the locale mapping in one location to avoid inconsistent values.
- Initialize locale and custom-field values before the chat starts.
- Do not duplicate locale or custom-field calls in the custom Guide-opening function.
- If supported languages change, update the dropdown, validation list, mapping logic, CXone translations, custom-field values, Studio logic, and Omilia configuration together.
