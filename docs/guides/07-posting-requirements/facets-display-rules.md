---
title: Facets - Display Rules
description: How display rules control conditional facet visibility based on other facet values, with operators, examples, and implementation guidance.
category: guides/posting-requirements
endpoints: []
prerequisites: [facets]
concepts: [display_rules, equal, in, notempty, contains, selected_option_show_contains]
related: [facets, validation, autocomplete]
audience: [developer]
difficulty: intermediate
---

# Facets - Display Rules

> Display rules make facets conditionally visible based on the values of other facets. They are the primary mechanism for building dynamic, context-sensitive posting requirement forms.

## Overview

Many job boards require fields that only make sense in certain contexts. A "salary description" field should only appear when the recruiter chooses to display salary information. A "state" dropdown is only relevant after selecting a country that has states. Display rules encode this logic directly in the facet definition, so your integration can build reactive forms without hardcoding channel-specific behavior.

A facet with `"display_rules": null` is **always visible**. A facet with display rules is only visible when its conditions are met.

## Structure

Display rules live in the `display_rules` field of a facet object:

```json
{
  "display_rules": {
    "show": [
      { "facet": "<referenced_facet_name>", "op": "<operator>", "value": "<expected_value>" }
    ]
  }
}
```

The `show` array contains one or more conditions. Each condition references another facet by name and defines an operator to evaluate against that facet's current value.

### Evaluation Logic

**All conditions in the `show` array must be true** (AND logic) for the facet to be visible. If any single condition evaluates to false, the facet is hidden.

<!-- theme: warning -->
> The `show` array uses AND logic, not OR. Every condition must match for the facet to appear.

## Operators

| Operator | Description | `value` field |
|----------|-------------|---------------|
| `equal` | The referenced facet's value exactly matches `value` | string |
| `in` | The referenced facet's value matches any entry in `value` | array of strings |
| `notempty` | The referenced facet has a non-empty value | `null` (not used) |
| `contains` | The referenced facet's selected values include `value` (for `MULTIPLE` facets) | string |
| `selected_option_show_contains` | The selected option's `show` property includes this facet's `name` | `null` (not used) |

## Operators in Detail

### `equal` - Show When a Facet Has a Specific Value

The simplest and most common operator. The facet is visible only when the referenced facet's value exactly equals the specified string.

**Real-world example** - A "Pay information" text field that only appears when the recruiter opts to display salary information:

```json
{
  "name": "remunerationDescription",
  "label": "Pay information (displayed on ad)",
  "type": "TEXT",
  "required": true,
  "rules": [{ "data": "50", "rule": "maxlength" }],
  "display_rules": {
    "show": [
      { "op": "equal", "facet": "displaySalaryInformation", "value": "1" }
    ]
  }
}
```

The `displaySalaryInformation` facet is typically a `SELECT` with options like `"0"` (No) and `"1"` (Yes). When the recruiter selects "Yes", the `remunerationDescription` field appears and becomes required.

**Intuitive example** - An "education specialization" field that only appears when the education level is set to "university":

```json
{
  "name": "educationSpecialization",
  "label": "Field of Study",
  "type": "TEXT",
  "required": true,
  "display_rules": {
    "show": [
      { "op": "equal", "facet": "educationLevel", "value": "university" }
    ]
  }
}
```

If `educationLevel` is "high_school" or "vocational", the specialization field stays hidden.

### `in` - Show When a Facet Has One of Several Values

Like `equal`, but the condition matches if the referenced facet's value is **any one** of the values in the array. Useful when multiple values should trigger the same field.

**Real-world example** - A contact person field that appears when the application status is "In Review" (2), "Approved" (7), or "Published" (8):

```json
{
  "name": "ba_contact",
  "label": "Contact Person",
  "type": "STATISCH",
  "required": false,
  "display_rules": {
    "show": [
      { "facet": "Status", "op": "in", "value": ["2", "7", "8"] },
      { "facet": "SupervisionDesired", "op": "equal", "value": "1" }
    ]
  }
}
```

Note this example has **two conditions** (AND logic). The field is visible only when the status is 2, 7, or 8 **and** supervision is desired. Both conditions must be true.

**Intuitive example** - A "driver's license type" field that only appears when the job category is one that involves driving:

```json
{
  "name": "driversLicenseType",
  "label": "Required Driver's License",
  "type": "SELECT",
  "required": true,
  "options": [
    { "key": "B", "label": "Class B (Car)", "sort": "1" },
    { "key": "C", "label": "Class C (Truck)", "sort": "2" },
    { "key": "D", "label": "Class D (Bus)", "sort": "3" }
  ],
  "display_rules": {
    "show": [
      { "op": "in", "facet": "jobCategory", "value": ["delivery", "logistics", "transport", "field_sales"] }
    ]
  }
}
```

### `notempty` - Show When a Facet Has Any Value

The facet is visible whenever the referenced facet's value is truthy. No `value` field is needed.

**Falsy values** (treated as empty): `null`, `""`, `[]`, `{}`, `0`

Note: the string `"0"` is **not** falsy.

**Real-world example** - A "maximum salary" field that only appears when a minimum salary has been entered:

```json
{
  "name": "salary_max",
  "label": "Salary max",
  "type": "TEXT",
  "required": true,
  "rules": [{ "data": "", "rule": "int" }],
  "message": "\"Salary max\" should be higher than \"Salary min\" and should only contain numbers",
  "display_rules": {
    "show": [
      { "op": "notempty", "facet": "salary_min", "value": null }
    ]
  }
}
```

This creates a natural flow: the recruiter enters a minimum salary, and only then does the maximum salary field appear.

**Real-world example** - A "video position" field that only appears when a video URL has been provided:

```json
{
  "name": "seekAnzPositionCode",
  "label": "Video position",
  "type": "SELECT",
  "required": true,
  "options": [
    { "key": "Above", "label": "Above", "sort": "0" },
    { "key": "Below", "label": "Below", "sort": "0" }
  ],
  "display_rules": {
    "show": [
      { "op": "notempty", "facet": "videoUrl", "value": null }
    ]
  }
}
```

**Intuitive example** - A "city" field that only appears after a country has been selected:

```json
{
  "name": "city",
  "label": "City",
  "type": "AUTOCOMPLETE",
  "required": true,
  "autocomplete": { "required_parameters": ["term"], "parameters_source": null },
  "display_rules": {
    "show": [
      { "op": "notempty", "facet": "country", "value": null }
    ]
  }
}
```

### `contains` - Show When a Multi-Select Includes a Value

Used with `MULTIPLE`-type facets. The condition matches when the referenced facet's selected values **include** the specified string. This is different from `equal`-it checks membership in an array rather than exact equality.

**Real-world example** - An "application email" field that only appears when "EMAIL" is one of the selected application methods:

```json
{
  "name": "apply_email",
  "label": "Email",
  "type": "TEXT",
  "required": true,
  "rules": [{ "data": "", "rule": "email" }],
  "display_rules": {
    "show": [
      { "op": "contains", "facet": "apply_method", "value": "EMAIL" }
    ]
  }
}
```

The `apply_method` facet is a `MULTIPLE` where the recruiter can select several methods (e.g., "EMAIL", "URL", "PHONE"). If "EMAIL" is among the selections, the email address field appears. Other selected methods would trigger their own dependent fields with similar display rules.

**Intuitive example** - A "benefits detail" text field that appears when "benefits" is one of the selected compensation components:

```json
{
  "name": "benefitsDescription",
  "label": "Describe the benefits package",
  "type": "TEXTAREA",
  "required": true,
  "display_rules": {
    "show": [
      { "op": "contains", "facet": "compensationComponents", "value": "benefits" }
    ]
  }
}
```

If `compensationComponents` is a `MULTIPLE` with values like `["base_salary", "benefits", "bonus"]`, the field appears because `"benefits"` is included.

### `selected_option_show_contains` - Show Based on the Selected Option's `show` Property

This is the most advanced operator. Instead of checking the referenced facet's **value**, it checks the **`show` property of the currently selected option** in the referenced facet.

Each option in a `SELECT` facet can have a `show` array listing the facet names that should be visible when that option is selected:

```json
{
  "name": "country",
  "type": "SELECT",
  "options": [
    { "key": "usa", "label": "United States", "show": ["state_id", "zip_code"] },
    { "key": "fr",  "label": "France",        "show": ["department"] },
    { "key": "nl",  "label": "Netherlands",    "show": ["province"] }
  ]
}
```

Dependent facets reference `country` with `selected_option_show_contains`:

```json
{
  "name": "state_id",
  "label": "State",
  "type": "SELECT",
  "display_rules": {
    "show": [
      { "facet": "country", "op": "selected_option_show_contains", "value": null }
    ]
  }
}
```

```json
{
  "name": "department",
  "label": "Department",
  "type": "SELECT",
  "display_rules": {
    "show": [
      { "facet": "country", "op": "selected_option_show_contains", "value": null }
    ]
  }
}
```

The evaluation works like this:
1. Look up the currently selected option in the `country` facet (e.g., the user selected `"usa"`)
2. Read that option's `show` array: `["state_id", "zip_code"]`
3. Check if the current facet's `name` is in that array
4. `"state_id"` is in `["state_id", "zip_code"]` -> visible
5. `"department"` is **not** in `["state_id", "zip_code"]` -> hidden

| User selects | `state_id` | `zip_code` | `department` | `province` |
|-------------|:---:|:---:|:---:|:---:|
| United States | visible | visible | hidden | hidden |
| France | hidden | hidden | visible | hidden |
| Netherlands | hidden | hidden | hidden | visible |

<!-- theme: info -->
> This operator is powerful because it lets the channel define visibility mapping **per option** without needing a separate display rule for each combination. The logic is embedded in the option data, not in the dependent facet.

#### Combining with `notempty`

Display rules using `selected_option_show_contains` often appear alongside a `notempty` condition on the same facet. This is because:
- `notempty` ensures the referenced facet has a value selected at all
- `selected_option_show_contains` checks whether the selected option's `show` list includes this facet

**Real-world example**:

```json
{
  "name": "Course",
  "label": "Course of Study",
  "type": "SELECT",
  "required": false,
  "display_rules": {
    "show": [
      { "facet": "TitleCode", "op": "notempty", "value": null },
      { "facet": "TitleCode", "op": "selected_option_show_contains", "value": null }
    ]
  }
}
```

Both conditions must be true (AND logic): `TitleCode` must have a value **and** the selected option's `show` list must include `"Course"`. In practice, when `TitleCode` has a value, the `selected_option_show_contains` check determines which dependent fields appear for that specific title.

#### Real-World Walkthrough: Ad Tier Unlocks Branding Fields

This pattern is common on job boards where premium ad tiers unlock additional fields. Consider a channel that offers multiple ad products - only certain tiers include branding and selling points.

The **controlling facet** (`seekAnzAdvertisementType`) is a `SELECT` whose options are fetched dynamically. Each option may or may not carry a `show` array:

```json
[
  {
    "key": "basic",
    "label": "Basic",
    "data": { "description": "For non-urgent and easy-to-fill roles..." }
  },
  {
    "key": "branded-basic",
    "label": "Branded Basic",
    "show": ["brandingId", "usp1", "usp2", "usp3"],
    "data": { "description": "Branding and extra selling points included..." }
  },
  {
    "key": "branded-advanced",
    "label": "Branded Advanced",
    "show": ["brandingId", "usp1", "usp2", "usp3"],
    "data": { "description": "Enhanced targeting with branding..." }
  },
  {
    "key": "premium",
    "label": "Premium",
    "show": ["brandingId", "usp1", "usp2", "usp3"],
    "data": { "description": "Top performing job ad with full branding..." }
  }
]
```

Notice that "Basic" has **no `show` property**, while all branded/premium tiers include `["brandingId", "usp1", "usp2", "usp3"]`.

The **dependent facet** (`brandingId`) uses the familiar `notempty` + `selected_option_show_contains` combination:

```json
{
  "name": "brandingId",
  "label": "Branding",
  "type": "SELECT",
  "required": true,
  "options": [],
  "autocomplete": { "required_parameters": null, "parameters_source": null },
  "display_rules": {
    "show": [
      { "facet": "seekAnzAdvertisementType", "op": "notempty", "value": null },
      { "facet": "seekAnzAdvertisementType", "op": "selected_option_show_contains", "value": null }
    ]
  }
}
```

The evaluation:

| User selects | `brandingId` | `usp1`, `usp2`, `usp3` |
|-------------|:---:|:---:|
| Basic | hidden | hidden |
| Branded Basic | visible | visible |
| Branded Advanced | visible | visible |
| Premium | visible | visible |

**Key implementation details for this scenario:**

1. **Options are lazy-loaded.** The controlling facet (`seekAnzAdvertisementType`) has `"options": []` and a non-null `autocomplete` field. You must fetch its options via the [Autocomplete](autocomplete.md) endpoint before you can evaluate `selected_option_show_contains`. The `show` arrays live on the option objects returned by autocomplete - they are not part of the facet definition itself.

2. **`show` may be absent.** Not every option carries a `show` property. When the selected option has no `show` array (like "Basic"), `selected_option_show_contains` evaluates to **false** - the dependent facets stay hidden. Treat a missing `show` as an empty array.

3. **The dependent facet is also lazy-loaded.** `brandingId` itself has `"options": []` with autocomplete - once it becomes visible, you need a second autocomplete call to populate its dropdown. Only fetch these options when the facet is actually visible to avoid unnecessary API calls.

## Multiple Conditions (AND Logic)

When the `show` array contains multiple conditions, **all must be true** for the facet to be visible. This allows complex visibility logic.

**Real-world example** - A phone number field that requires both a specific status AND supervision being requested:

```json
{
  "name": "ba_contact_postal_address_voice_number",
  "label": "Phone Number",
  "type": "TEXT",
  "required": false,
  "display_rules": {
    "show": [
      { "facet": "Status", "op": "in", "value": ["2", "7", "8"] },
      { "facet": "SupervisionDesired", "op": "equal", "value": "1" }
    ]
  }
}
```

This creates a two-gate visibility: the status must be one of the expected values **and** supervision must be desired. If either condition fails, the field stays hidden.

**Intuitive example** - A "relocation budget" field that only appears when the job allows remote work AND the position is senior level:

```json
{
  "name": "relocationBudget",
  "label": "Relocation Budget",
  "type": "SELECT",
  "required": true,
  "options": [
    { "key": "5000", "label": "Up to 5,000", "sort": "1" },
    { "key": "10000", "label": "Up to 10,000", "sort": "2" },
    { "key": "unlimited", "label": "No limit", "sort": "3" }
  ],
  "display_rules": {
    "show": [
      { "op": "equal", "facet": "allowsRemoteWork", "value": "0" },
      { "op": "in", "facet": "seniorityLevel", "value": ["senior", "lead", "director"] }
    ]
  }
}
```

## Implementation Guide

### Rendering Flow

When building a posting requirements form:

1. **Load all facets** for the product or contract
2. **Evaluate display rules** for each facet against the current form state
3. **Render visible facets** and hide facets whose conditions are not met
4. **Re-evaluate on every value change** - when the user changes a facet value, re-check all display rules since dependent facets may need to appear or disappear

```
User fills "salary_min" = 50000
  -> Re-evaluate all display rules
  -> "salary_max" has notempty rule on "salary_min"
  -> "salary_min" is now truthy -> show "salary_max"
```

### Hidden Facets and Submission

<!-- theme: warning -->
> **Hidden facets must be omitted from the request payload.** Even if a facet has `required: true`, if it is currently hidden by display rules, do not submit a value for it. Server-side validation skips hidden facets.

When a facet becomes hidden after previously being visible:
- Clear its value from your form state
- Remove it from the submission payload
- Any facets that depend on the now-hidden facet should also be re-evaluated (cascading hide)

### Cascading Visibility

Display rules can form chains: facet A controls facet B, which controls facet C. When A's value changes and hides B, facet C should also be hidden since its dependency (B) no longer has a value.

**Example chain**:

```
country (always visible)
  -> state (notempty on country)
    -> city (notempty on state)
```

If the user clears `country`, both `state` and `city` should disappear. Your implementation should:
1. Clear `country` -> hide `state`, clear `state`'s value
2. `state` cleared -> hide `city`, clear `city`'s value

### Pseudocode

```javascript
function isFacetVisible(facet, formState, allFacets) {
  if (!facet.display_rules) return true;

  return facet.display_rules.show.every(condition => {
    const refValue = formState[condition.facet];

    switch (condition.op) {
      case 'equal':
        return refValue === condition.value;

      case 'in':
        return condition.value.includes(refValue);

      case 'notempty':
        return refValue != null
          && refValue !== ''
          && refValue !== 0
          && !(Array.isArray(refValue) && refValue.length === 0)
          && !(typeof refValue === 'object' && Object.keys(refValue).length === 0);

      case 'contains':
        return Array.isArray(refValue) && refValue.includes(condition.value);

      case 'selected_option_show_contains':
        const refFacet = allFacets.find(f => f.name === condition.facet);
        const selectedOption = refFacet?.options.find(o => o.key === refValue);
        return selectedOption?.show?.includes(facet.name) ?? false;

      default:
        return false;
    }
  });
}
```

## Related

- [Facets](facets.md) - complete facet object reference, all types, options, and validation rules
- [Facets - Endpoint Reference](facets.endpoints.md) - JSON examples for all facet types and display rules
- [Validation](validation.md) - client-side and server-side validation
- [Autocomplete](autocomplete.md) - dynamic options for SELECT, HIER, and AUTOCOMPLETE facets
