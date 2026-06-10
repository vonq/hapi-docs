---
title: Facets
description: Complete reference for the facet object-all 12 types, options, display rules, and validation rules.
category: guides/posting-requirements
endpoints: []
prerequisites: [posting-requirements-introduction]
concepts: [facet, TEXT, TEXTAREA, HTMLAREA, TEXTEXPAND, SELECT, MULTIPLE, HIER, AUTOCOMPLETE, DATE, STATISCH, AREACOUNT, QUESTIONNAIRE, display_rules, validation_rules, options]
related: [autocomplete, validation, product-posting-requirements, contract-posting-requirements]
audience: [developer]
difficulty: advanced
---

# Facets

> The complete reference for the facet object-field definitions, all 12 types, options, display rules, and validation rules.

## Overview

A **facet** is a single posting requirement field. Every channel defines its own set of facets-the fields a job board needs to publish a vacancy. This page is the definitive reference for the facet object structure. For how to retrieve facets, see [Product Posting Requirements](../05-products/04-posting-requirements.md) and [Contract Posting Requirements](../06-contracts/posting-requirements.md).

See [Facets - Endpoint Reference](./facets.endpoints.md) for full request/response details and JSON examples for each type.

## Facet Object Reference

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Machine identifier-the key to use in `postingRequirements` when ordering. **Case-sensitive.** |
| `label` | string | Human-readable display label |
| `sort` | string | Display order (lower values first). Returned as string-parse as integer for sorting. |
| `required` | boolean | Whether the facet must be filled. Values `0`/`1` are equivalent to `false`/`true`. |
| `type` | string | Facet type-determines the UI control and value format (see below) |
| `options` | array | Predefined options for selection-based facets. Empty array if none or lazy-loaded. |
| `rules` | array | Validation rules (see [Validation Rules](#validation-rules) below) |
| `message` | string | Help text or format instructions to display alongside the input |
| `autocomplete` | object or null | Autocomplete configuration. Non-null means options must be fetched dynamically. See [Autocomplete](autocomplete.md). |
| `display_rules` | object or null | Conditional visibility rules (see [Display Rules](#display-rules) below). Null means always visible. |
| `primary_taxonomy` | string | Classification label grouping related facets (e.g. `job_product`, `employer_location`). Reserved for future use. |

## Facet Types

### TEXT

Single-line text input. Value is a plain string.

**Typical rules**: `maxlength`, `minlength`, `regex`

**Submit as**: `"company_name": "Acme Corp"`

### TEXTAREA

Multi-line text input. Value is a plain string with newlines.

**Typical rules**: `maxlength`, `minlength`

**Submit as**: `"requirements": "3+ years experience\nStrong communication skills"`

### HTMLAREA

Rich text editor. Value is an HTML string with a limited set of allowed tags.

**Allowed tags**: `p[style]`, `br`, `ul`, `ol`, `li`, `h1`, `h2`, `h3`, `em`, `strong`, `b`, `i`, `u`, `sup`, `sub`, `a[href|target|title]`, `img[src|border|alt|title|width|height|align]`, `hr[width|size|noshade]`, `span[class|style]`

Tags outside this set are stripped during server-side validation.

**Typical rules**: `maxlength`, `minlength`

**Submit as**: `"tasks": "<ul><li>Lead the engineering team</li><li>Design system architecture</li></ul>"`

### TEXTEXPAND

Text input where the value is a JSON-encoded array of strings. Used when a channel needs multiple values in a single field.

**Submit as**: `"cities": "[\"Amsterdam\", \"Rotterdam\", \"Utrecht\"]"`

<!-- theme: info -->
> The value must be a **JSON-encoded string**, not an actual array. The server parses it during validation.

### SELECT

Single-select dropdown with predefined or lazy-loaded options.

**Submit as**: `"employment_type": "fulltime"`

If a `SELECT` facet has a non-null `autocomplete` field with an empty `options` array, options must be loaded via the autocomplete endpoint before the user can interact with it.

### MULTIPLE

Multi-select dropdown. Value is an array of selected option keys.

**Typical rules**: `maxitems`, `minitems`

**Submit as**: `"skills": ["java", "python"]`

### HIER

Hierarchical dropdown-options form a tree using `parent` references. Only **leaf nodes** (options whose `key` is never used as another option's `parent`) are selectable.

**Submit as**: `"location": "3"` (only leaf nodes are selectable)

**Building the tree**: Match each option's `parent` value to another option's `key`. Options with `parent: null` are root nodes. Common UI patterns:
- Nested dropdowns with `<optgroup>` elements
- Flattened labels like "Italy > Lazio > Rome" in a single dropdown
- Cascading selects (country → region → city)

Like `SELECT`, a `HIER` facet can be lazy-loaded-check the `autocomplete` field.

### AUTOCOMPLETE

Search-as-you-type field. Options are fetched dynamically from the autocomplete endpoint as the user types.

**Submit as**: `"location": "sydney-cbd"` (the `key` from the selected autocomplete result)

See [Autocomplete](autocomplete.md) for endpoint details, parameter sources, and multi-term patterns.

### DATE

Date picker. Value is a string in the format specified by the `date` validation rule.

**Submit as**: `"start_date": "2025-04-01"`

Common formats: `Y-m-d` (2025-04-01) and `d.m.Y` (01.04.2025). Always check the `date` rule for the expected format.

### STATISCH

Display-only field-informational text shown to the user. **Never submit a value for this type.**

**Submit as**: Omit entirely from the `postingRequirements` object.

### AREACOUNT

Numeric area/region counter. Value is an integer representing a count or quantity associated with a geographic area.

**Submit as**: `"numberOfLocations": "3"`

### QUESTIONNAIRE

Questionnaire builder for configuring candidate screening questions. Used by channels that support [Direct Apply](../10-direct-apply/01-introduction.md).

The value is a structured object defining questions, answer types, and validation. See [Direct Apply Posting Requirements](../10-direct-apply/posting-requirements.md) for the full format.

## Options Object

Each entry in the `options` array contains:

| Field | Type | Description |
|-------|------|-------------|
| `key` | string | Value to submit when selected. Case-sensitive. |
| `label` | string | Display text |
| `sort` | string | Sort order (string-parse as integer). Lower values first. |
| `default` | string | If present, pre-select this option |
| `parent` | string or null | Parent option key (`HIER` facets only). Null for root nodes. |
| `show` | array of strings | Facet names to reveal when this option is selected (used with `selected_option_show_contains` display rule) |
| `requires` | array or null | Vacancy fields that become required when this option is selected (see [Vacancy Field Requirements](#vacancy-field-requirements-on-options) below) |
| `data` | array | Additional metadata-can include images with `src`, `title` for enhanced UI display |

See [Facets - Endpoint Reference](./facets.endpoints.md) for a JSON example with metadata.

### Vacancy Field Requirements on Options

Some options carry a `requires` array that lists vacancy fields the backend will enforce when that option is selected. This creates a cross-field dependency between a posting requirement facet and the campaign's vacancy fields.

**Structure**: Each entry in `requires` points to a vacancy field by dotted path:

```json
{
  "key": "URL",
  "label": "URL",
  "requires": [{ "field": { "name": "applicationUrl" } }]
}
```

When the user selects this option, the referenced vacancy field (`applicationUrl`) becomes required - the backend rejects the order if it's missing.

**Single requirement** - A `SELECT` facet for application method:

```json
{
  "name": "apply_method",
  "type": "SELECT",
  "options": [
    {
      "key": "EMAIL",
      "label": "E-mail",
      "requires": [{ "field": { "name": "contactInfo.emailAddress" } }]
    },
    {
      "key": "URL",
      "label": "URL",
      "requires": [{ "field": { "name": "applicationUrl" } }]
    }
  ]
}
```

Selecting `"EMAIL"` means `contactInfo.emailAddress` must be present in the campaign payload. Selecting `"URL"` means `applicationUrl` must be present.

**Multiple requirements** - A `MULTIPLE` facet where each option requires different vacancy fields, and some options require more than one:

```json
{
  "name": "IGB_ApplicationMethod",
  "type": "MULTIPLE",
  "options": [
    {
      "key": "InternetWebAddress",
      "label": "Website",
      "requires": [{ "field": { "name": "applicationUrl" } }]
    },
    {
      "key": "InternetEmailAddress",
      "label": "E-mail",
      "requires": [{ "field": { "name": "contactInfo.emailAddress" } }]
    },
    {
      "key": "Telephone",
      "label": "Telefoon",
      "requires": [{ "field": { "name": "contactInfo.phoneNumber" } }]
    },
    {
      "key": "PostalAddress",
      "label": "Brief",
      "requires": [
        { "field": { "name": "workingLocation.addressLine1" } },
        { "field": { "name": "workingLocation.city" } },
        { "field": { "name": "workingLocation.zipCode" } }
      ]
    }
  ]
}
```

With `MULTIPLE`, the user can select several options at once. The vacancy field requirements are **cumulative** - if the user selects both `"InternetWebAddress"` and `"Telephone"`, both `applicationUrl` and `contactInfo.phoneNumber` become required.

<!-- theme: warning -->
> ### Validate vacancy fields against selected options
> The `requires` field creates a runtime dependency: which vacancy fields are required depends on which posting requirement options are selected. Your integration should check `requires` when rendering the form and ensure the corresponding vacancy fields are filled before submitting.

## Display Rules

Display rules control whether a facet is visible based on the value of another facet. A facet with `display_rules: null` is always visible.

**Structure**:

```json
{
  "display_rules": {
    "show": [
      { "facet": "countryId", "value": "usa", "op": "equal" }
    ]
  }
}
```

**All conditions** in the `show` array must be true (AND logic) for the facet to be visible.

<!-- theme: warning -->
> **Hidden facets must be omitted from the request payload.** Even if a facet has `required: true`, if it's hidden by display rules, do not submit it. Server-side validation skips hidden facets.

### Operations

| Operation | Description | `value` field |
|-----------|-------------|---------------|
| `equal` | Referenced facet's value equals `value` | string |
| `in` | Referenced facet's value is one of the values in the array | array of strings |
| `notempty` | Referenced facet's value is not empty/falsy | not used |
| `contains` | Referenced facet's selected values (for `MULTIPLE`) include `value` | string |
| `selected_option_show_contains` | The selected option's `show` property includes this facet's `name` | not used |

For detailed explanations, real-world examples, implementation pseudocode, and guidance on cascading visibility, see the dedicated [Facets - Display Rules](facets-display-rules.md) page.

## Validation Rules

Each rule is an object with `rule` (type) and `data` (constraint value as string).

| Rule | Data | Description |
|------|------|-------------|
| `maxlength` | number as string | Maximum string length |
| `minlength` | number as string | Minimum string length |
| `maxitems` | number as string | Maximum selected options (`MULTIPLE`) |
| `minitems` | number as string | Minimum selected options |
| `int` | empty string | Must be an integer |
| `float` | empty string | Decimal number (period `.` as decimal, comma `,` as thousands separator) |
| `regex` | pattern string | Must match regex (includes delimiters and flags, e.g., `/^[a-z]{2}$/i`) |
| `date` | format string | Date format (`Y-m-d` or `d.m.Y`) |
| `email` | empty string | Must be a valid email |
| `url` | empty string | Must be a valid URL |

### Parsing Regex Rules

Regex patterns include delimiters and flags as a single string. See [Facets - Endpoint Reference](./facets.endpoints.md) for a JavaScript parsing example.

<!-- theme: info -->
> Client-side validation is optional-it improves UX by catching errors early. Server-side validation is always enforced on campaign ordering and validation endpoints. See [Validation](validation.md).

## Edge Cases & Gotchas

<!-- theme: warning -->
> **Facet names are case-sensitive.** When submitting `postingRequirements`, keys must exactly match the facet `name`. `"JobCategory"` is not the same as `"jobcategory"`.

<!-- theme: warning -->
> **`sort` is a string.** Despite representing numeric order, the `sort` field on both facets and options is returned as a string. Parse as integer: `parseInt(facet.sort)`.

<!-- theme: warning -->
> **Lazy-loaded SELECT and HIER.** Some `SELECT` and `HIER` facets have a non-null `autocomplete` field but an empty `options` array. Options must be fetched via the autocomplete endpoint before the user can interact with the facet.

<!-- theme: info -->
> **STATISCH is never submitted.** Render as display-only text and omit entirely from the `postingRequirements` object.

<!-- theme: info -->
> **TEXTEXPAND requires JSON encoding.** The value must be a JSON-encoded string (e.g., `"[\"a\", \"b\"]"`), not an actual array.

<!-- theme: info -->
> **HIER-only leaf nodes are selectable.** Parent options are grouping labels. A leaf node is any option whose `key` is never referenced as another option's `parent`.

## Related

- [Posting Requirements Introduction](01-introduction.md)-high-level overview
- [Autocomplete](autocomplete.md)-dynamic options, parameter sources, multi-term
- [Validation](validation.md)-client and server-side validation, validation endpoints
- [Product Posting Requirements](../05-products/04-posting-requirements.md)-retrieving facets from products
- [Contract Posting Requirements](../06-contracts/posting-requirements.md)-retrieving facets from contracts
