# Open Inspection Registry

[中文说明 / Chinese README](README_cn.md)

## English

Open Inspection Registry is a public catalog of reusable image-inspection templates,
observations, and skills. It helps users send images and an inspection intent to an Open
Inspection API or MCP server and receive structured, evidence-based results.

The service accepts one to three images. You can send image URLs or Base64 image content. Your
inspection intent can be a registered `skill`, a free-form `user_skill`, or a `prompt`.

### Current templates

- `general_inspection`
- `safety_inspection`
- `quality_inspection`

Pass a template as `template_slug`. If it is omitted or unavailable, the service uses the
general inspection template.

### REST API

Replace `<YOUR_API_BASE_URL>` with the service URL provided by your administrator. Never add
API keys, SAS URLs, private image URLs, or production credentials to this repository.

Analyze one to three images by URL with an observation set:

```bash
curl -X POST '<YOUR_API_BASE_URL>/v1/inspections:analyze' \
  -H 'Content-Type: application/json' \
  -d '{
    "image_urls": ["<IMAGE_URL_1>", "<IMAGE_URL_2>"],
    "observation_set": "manufacturing_quality_basic",
    "prompt": "Check visible labels, cleanliness, and surface damage."
  }'
```

Single-image input is also supported:

```json
{
  "image_url": "<IMAGE_URL_1>",
  "user_skill": "Check whether the warehouse label is present and readable."
}
```

To select a ready template, use `/v1/inspections:analyze-template`:

```json
{
  "image_urls": ["<IMAGE_URL_1>"],
  "template_slug": "quality_inspection",
  "prompt": "Check visible labels, cleanliness, and surface damage."
}
```

For local image files, use `/v1/inspections:analyze-upload` with one to three `files` fields
and an `observation_set`. The request must include one of `skill`, `user_skill`, or `prompt`.

Public REST endpoints:

| Method | Path | Purpose |
| --- | --- | --- |
| `GET` | `/healthz` | Service health check |
| `GET` | `/v1/registry/observation-sets` | List built-in observation sets |
| `GET` | `/v1/templates` | List ready templates |
| `GET` | `/v1/templates/{slug}` | Read one template |
| `POST` | `/v1/inspections:analyze` | Analyze image URLs with an observation set |
| `POST` | `/v1/inspections:analyze-template` | Analyze image URLs with a template |
| `POST` | `/v1/inspections:analyze-upload` | Analyze uploaded image files |

Template administration endpoints are restricted to administrators.

### MCP

The Streamable HTTP MCP endpoint is:

```text
<YOUR_API_BASE_URL>/mcp/
```

The MCP tool is `analyze_inspection`:

```json
{
  "image_urls": ["<IMAGE_URL_1>"],
  "template_slug": "safety_inspection",
  "user_skill": "Check whether required protective equipment is visible."
}
```

Use `images_base64` instead of `image_urls` when sending image bytes. Provide exactly one of
these two input forms. MCP supports `template_slug`, `skill`, `user_skill`, and `prompt`.

### Response example

```json
{
  "inspection_result": {
    "finding": "The image shows a visible label and no major surface damage."
  },
  "image_results": [
    {
      "index": 1,
      "source": "<IMAGE_SOURCE>",
      "observations": {
        "label_status": "present",
        "surface_damage": "none"
      }
    }
  ],
  "observations": {
    "label_status": "present",
    "surface_damage": "none"
  }
}
```

Results are grounded in visible image evidence. An observation may be `unknown` when the
image does not provide enough evidence.

### Contributing

To propose a new inspection field or checkpoint, read [SKILL.md](SKILL.md). Do not submit
customer images, confidential documents, real service URLs, API keys, SAS URLs, or secrets.
