# Working with HAPI documentation

Start with [`llms.txt`](https://github.com/vonq/hapi-docs/blob/master/llms.txt) for a compact route to the guides, or query
[`docs-manifest.jsonl`](https://github.com/vonq/hapi-docs/blob/master/docs-manifest.jsonl) by document metadata and OpenAPI
operation ID. Open only the sources needed for the task.

Use `rg` to find guide text:

```bash
rg -n -i 'campaign status|webhook' docs --glob '*.md'
```

Use `jq` to select one OpenAPI operation:

```bash
jq --arg id 'hapi-OrderCampaign' '
  .paths | to_entries[] as $path
  | $path.value | to_entries[]
  | select(.value.operationId? == $id)
  | {method: .key, path: $path.key, operation: .value}
' schema/build/public.json
```

Use `jq` to select one component schema:

```bash
jq '.components.schemas.HAPICampaignCreateRequest' schema/build/public.json
```

Do not preload the documentation tree or `schema/build/public.json`. Search the
manifest or guides first, then select the relevant files, operation, or schema.

Apply this authority order:

1. OpenAPI defines exact HTTP paths, methods, parameters, payloads, responses,
   and authentication.
2. Guides and scenarios explain meaning and provide examples.
3. `docs/extra/api-map.yaml` defines sequencing and decision points.

Report conflicts between these sources. Do not guess which value was intended.
