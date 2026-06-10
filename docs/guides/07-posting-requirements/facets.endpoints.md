---
title: Facets - Endpoint Reference
description: JSON examples for all facet types, options, display rules, and validation rules.
category: guides/posting-requirements
---

> For conceptual overview, see [Facets](./facets.md).

# Facets - Endpoint Reference

This page contains the JSON examples for each facet type, options object fields, display rules, and validation rules. For how to retrieve facets via API, see [Product Posting Requirements](../05-products/04-posting-requirements.md) and [Contract Posting Requirements](../06-contracts/posting-requirements.md).

## Facet Type Examples

### TEXT

Single-line text input. Value is a plain string.

**Typical rules**: `maxlength`, `minlength`, `regex`

```json
{
  "name": "company_name",
  "label": "Company Name",
  "type": "TEXT",
  "required": true,
  "options": [],
  "rules": [{ "rule": "maxlength", "data": "100" }]
}
```

**Submit as**: `"company_name": "Acme Corp"`

### TEXTAREA

Multi-line text input. Value is a plain string with newlines.

**Typical rules**: `maxlength`, `minlength`

```json
{
  "name": "requirements",
  "label": "Job Requirements",
  "type": "TEXTAREA",
  "required": true,
  "options": [],
  "rules": [{ "rule": "maxlength", "data": "5000" }]
}
```

**Submit as**: `"requirements": "3+ years experience\nStrong communication skills"`

### HTMLAREA

Rich text editor. Value is an HTML string with a limited set of allowed tags.

**Allowed tags**: `p[style]`, `br`, `ul`, `ol`, `li`, `h1`, `h2`, `h3`, `em`, `strong`, `b`, `i`, `u`, `sup`, `sub`, `a[href|target|title]`, `img[src|border|alt|title|width|height|align]`, `hr[width|size|noshade]`, `span[class|style]`

Tags outside this set are stripped during server-side validation.

**Typical rules**: `maxlength`, `minlength`

```json
{
  "name": "tasks",
  "label": "Tasks & Responsibilities",
  "type": "HTMLAREA",
  "required": true,
  "options": [],
  "rules": [{ "rule": "maxlength", "data": "10000" }]
}
```

**Submit as**: `"tasks": "<ul><li>Lead the engineering team</li><li>Design system architecture</li></ul>"`

### TEXTEXPAND

Text input where the value is a JSON-encoded array of strings. Used when a channel needs multiple values in a single field.

```json
{
  "name": "cities",
  "label": "Target Cities",
  "type": "TEXTEXPAND",
  "required": false,
  "options": []
}
```

**Submit as**: `"cities": "[\"Amsterdam\", \"Rotterdam\", \"Utrecht\"]"`

<!-- theme: info -->
> The value must be a **JSON-encoded string**, not an actual array. The server parses it during validation.

### SELECT

Single-select dropdown with predefined or lazy-loaded options.

```json
{
  "name": "employment_type",
  "label": "Employment Type",
  "type": "SELECT",
  "required": true,
  "options": [
    { "key": "fulltime", "label": "Full Time", "sort": "1" },
    { "key": "parttime", "label": "Part Time", "sort": "2" },
    { "key": "contract", "label": "Contract", "sort": "3" }
  ]
}
```

**Submit as**: `"employment_type": "fulltime"`

### MULTIPLE

Multi-select dropdown. Value is an array of selected option keys.

**Typical rules**: `maxitems`, `minitems`

```json
{
  "name": "skills",
  "label": "Required Skills",
  "type": "MULTIPLE",
  "required": true,
  "options": [
    { "key": "java", "label": "Java" },
    { "key": "python", "label": "Python" },
    { "key": "go", "label": "Go" }
  ],
  "rules": [
    { "rule": "maxitems", "data": "5" },
    { "rule": "minitems", "data": "1" }
  ]
}
```

**Submit as**: `"skills": ["java", "python"]`

### HIER

Hierarchical dropdown-options form a tree using `parent` references. Only **leaf nodes** (options whose `key` is never used as another option's `parent`) are selectable.

```json
{
  "name": "location",
  "label": "Location",
  "type": "HIER",
  "required": true,
  "options": [
    { "key": "1", "label": "Italy", "parent": null },
    { "key": "2", "label": "Lazio", "parent": "1" },
    { "key": "3", "label": "Rome", "parent": "2" },
    { "key": "4", "label": "Lombardy", "parent": "1" },
    { "key": "5", "label": "Milan", "parent": "4" }
  ]
}
```

**Submit as**: `"location": "3"` (only "Rome" or "Milan" are selectable-they are leaf nodes)

### AUTOCOMPLETE

Search-as-you-type field. Options are fetched dynamically from the autocomplete endpoint as the user types.

```json
{
  "name": "location",
  "label": "Location",
  "type": "AUTOCOMPLETE",
  "required": true,
  "options": [],
  "autocomplete": {
    "required_parameters": ["term"],
    "parameters_source": null
  }
}
```

**Submit as**: `"location": "sydney-cbd"` (the `key` from the selected autocomplete result)

### DATE

Date picker. Value is a string in the format specified by the `date` validation rule.

```json
{
  "name": "start_date",
  "label": "Start Date",
  "type": "DATE",
  "required": false,
  "options": [],
  "rules": [{ "rule": "date", "data": "Y-m-d" }]
}
```

**Submit as**: `"start_date": "2025-04-01"`

Common formats: `Y-m-d` (2025-04-01) and `d.m.Y` (01.04.2025). Always check the `date` rule for the expected format.

### STATISCH

Display-only field-informational text shown to the user. **Never submit a value for this type.**

```json
{
  "name": "branding_notice",
  "label": "Your posting will include standard branding",
  "type": "STATISCH",
  "required": false,
  "options": []
}
```

**Submit as**: Omit entirely from the `postingRequirements` object.

### AREACOUNT

Numeric area/region counter. Value is an integer representing a count or quantity associated with a geographic area.

```json
{
  "name": "numberOfLocations",
  "label": "Number of Locations",
  "type": "AREACOUNT",
  "required": false,
  "options": [],
  "rules": [{ "rule": "int", "data": "" }]
}
```

**Submit as**: `"numberOfLocations": "3"`

## Options Object Example

<details>
<summary>Option with metadata example</summary>

```json
{
  "key": "usa",
  "label": "United States",
  "sort": "1",
  "show": ["stateId", "zipCode"],
  "data": [
    { "image": { "title": "Flag", "src": "https://cdn.example.com/flags/usa.png" } }
  ]
}
```

</details>

## Display Rules Examples

> For a comprehensive guide with all operators, real-world examples, and implementation pseudocode, see [Facets - Display Rules](facets-display-rules.md).

### Basic equality rule

```json
{
  "display_rules": {
    "show": [
      { "facet": "countryId", "value": "usa", "op": "equal" }
    ]
  }
}
```

### `selected_option_show_contains` example

```json
[
  {
    "name": "country",
    "type": "SELECT",
    "options": [
      { "key": "france", "label": "France", "show": ["state_name"] },
      { "key": "usa", "label": "USA", "show": ["state_id", "zip_code"] }
    ]
  },
  {
    "name": "state_id",
    "type": "SELECT",
    "display_rules": {
      "show": [{ "facet": "country", "op": "selected_option_show_contains" }]
    }
  },
  {
    "name": "state_name",
    "type": "TEXT",
    "display_rules": {
      "show": [{ "facet": "country", "op": "selected_option_show_contains" }]
    }
  }
]
```

- User selects **USA** → `state_id` and `zip_code` are visible, `state_name` is hidden
- User selects **France** → `state_name` is visible, `state_id` and `zip_code` are hidden

## Validation Rule Examples

### Regex rule parsing

```json
{ "rule": "regex", "data": "/^[a-zA-Z0-9 ]+$/" }
```

To use in JavaScript:
```javascript
const [, pattern, flags] = data.match(/\/(.*)\/([a-z]*)/);
const regex = new RegExp(pattern, flags);
```
