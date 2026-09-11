# Open Inspection Registry

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

The legacy single-image form is also supported:

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

## 中文

Open Inspection Registry 是一个公开的图像巡检模板、观察项和 Skill 目录。它帮助用户
向 Open Inspection API 或 MCP 服务发送图片和巡检意图，并获得基于图片证据的结构化结果。

服务一次接受一到三张图片。图片可以通过 URL 或 Base64 内容发送。巡检意图可以使用已经
注册的 `skill`、自由文本 `user_skill`，或者 `prompt`。

### 当前模板

- `general_inspection`：通用图像巡检
- `safety_inspection`：安全巡检
- `quality_inspection`：质量巡检

通过 `template_slug` 指定模板。如果不填写，或者模板不可用，服务会使用通用巡检模板。

### REST API

将 `<YOUR_API_BASE_URL>` 替换为管理员提供的服务地址。不要在公开仓库中加入 API key、
SAS URL、私人图片 URL 或生产环境凭据。

通过图片 URL 和 observation set 分析一到三张图片：

```bash
curl -X POST '<YOUR_API_BASE_URL>/v1/inspections:analyze' \
  -H 'Content-Type: application/json' \
  -d '{
    "image_urls": ["<IMAGE_URL_1>", "<IMAGE_URL_2>"],
    "observation_set": "manufacturing_quality_basic",
    "prompt": "检查可见的标签、清洁度和表面损坏。"
  }'
```

也支持旧的单图片输入：

```json
{
  "image_url": "<IMAGE_URL_1>",
  "user_skill": "检查仓库物料标签是否存在并且清晰可读。"
}
```

如果要选择已经准备好的模板，请使用 `/v1/inspections:analyze-template`：

```json
{
  "image_urls": ["<IMAGE_URL_1>"],
  "template_slug": "quality_inspection",
  "prompt": "检查可见的标签、清洁度和表面损坏。"
}
```

本地图片文件可以使用 `/v1/inspections:analyze-upload`，提交一到三个 `files` 字段和
`observation_set`。请求必须提供 `skill`、`user_skill` 或 `prompt` 其中之一。

公开 REST API 包括：

| 方法 | 路径 | 用途 |
| --- | --- | --- |
| `GET` | `/healthz` | 健康检查 |
| `GET` | `/v1/registry/observation-sets` | 读取内置 observation set |
| `GET` | `/v1/templates` | 读取可用模板列表 |
| `GET` | `/v1/templates/{slug}` | 读取单个模板 |
| `POST` | `/v1/inspections:analyze` | 使用 observation set 分析图片 URL |
| `POST` | `/v1/inspections:analyze-template` | 使用模板分析图片 URL |
| `POST` | `/v1/inspections:analyze-upload` | 分析上传的图片文件 |

模板管理接口只允许管理员使用。

### MCP

Streamable HTTP MCP 地址为：

```text
<YOUR_API_BASE_URL>/mcp/
```

MCP tool 名称为 `analyze_inspection`：

```json
{
  "image_urls": ["<IMAGE_URL_1>"],
  "template_slug": "safety_inspection",
  "user_skill": "检查图片中是否能看到必要的个人防护装备。"
}
```

如果直接发送图片内容，可以使用 `images_base64` 代替 `image_urls`。两者必须二选一。
MCP 支持 `template_slug`、`skill`、`user_skill` 和 `prompt`。

### 输出示例

返回结果包括每张图片的 observations，以及一句简洁的最终 finding：

```json
{
  "inspection_result": {
    "finding": "图片显示标签清晰可见，未发现明显的表面损坏。"
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
  ]
}
```

所有结果都应基于图片中可见的证据。如果图片无法提供足够证据，观察项可以返回
`unknown`，而不是猜测。

### 贡献新检查项

如果要提交新的 inspection field 或 checkpoint，请先阅读 [SKILL.md](SKILL.md)。不要提交
客户图片、机密文件、真实服务地址、API key、SAS URL 或其他秘密信息。
