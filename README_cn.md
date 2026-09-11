# Open Inspection Registry 中文说明

Open Inspection Registry 是一个公开的图像巡检模板、观察项和 Skill 目录。它帮助用户向
Open Inspection API 或 MCP 服务发送图片和巡检意图，并获得基于图片证据的结构化结果。

## 当前模板

- `general_inspection`：通用图像巡检
- `safety_inspection`：安全巡检
- `quality_inspection`：质量巡检

通过 `template_slug` 指定模板。如果不填写，或者模板不可用，服务会使用通用巡检模板。

## REST API

将 `<YOUR_API_BASE_URL>` 替换为管理员提供的服务地址。不要在公开仓库中加入 API key、
SAS URL、私人图片 URL 或生产环境凭据。

服务一次接受一到三张图片。图片可以通过 URL 或 Base64 内容发送。巡检意图可以使用已经
注册的 `skill`、自由文本 `user_skill`，或者 `prompt`。

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

也支持单图片输入：

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

## 示例：5S 现场巡检

假设你有一张本地图片：

```text
./images/work-area.jpg
```

第一步，向服务管理员获取 API 基地址，并替换下面的 `<YOUR_API_BASE_URL>`。公开仓库不
提供真实服务地址或 API key。

第二步，使用上传接口发送图片。这个接口适合本地图片文件：

```bash
curl -X POST '<YOUR_API_BASE_URL>/v1/inspections:analyze-upload' \
  -F 'files=@./images/work-area.jpg' \
  -F 'observation_set=manufacturing_quality_basic' \
  -F 'user_skill=检查现场5S：整理、整顿、清扫、清洁和素养，指出图片中可见的问题。'
```

服务会返回每张图片的 observations 和一句最终 finding。例如：

```json
{
  "inspection_result": {
    "finding": "图片显示工作区域存在物品摆放不整齐和清洁不足的问题。"
  },
  "image_results": [
    {
      "index": 1,
      "observations": {
        "cleanliness": "dirty",
        "surface_damage": "unknown"
      }
    }
  ]
}
```

当前模板列表中还没有专门的 `5s_inspection` 模板。上面的方式可以先使用自由文本获得
5S 反馈，但返回的结构化字段取决于所选 `observation_set`。如果需要固定的 5S 字段，
请按照 [SKILL.md](SKILL.md) 提交新的 field/checkpoint Issue，由维护者创建或发布对应模板。

如果图片已经有可访问的 URL，可以使用模板接口：

```bash
curl -X POST '<YOUR_API_BASE_URL>/v1/inspections:analyze-template' \
  -H 'Content-Type: application/json' \
  -d '{
    "image_urls": ["<IMAGE_URL_1>"],
    "template_slug": "general_inspection",
    "user_skill": "检查图片中的5S现场管理问题。"
  }'
```

简单选择：

- 本地图片文件：使用 `/v1/inspections:analyze-upload`
- 图片 URL + 模板：使用 `/v1/inspections:analyze-template`
- 图片 URL + observation set：使用 `/v1/inspections:analyze`
- MCP 客户端：调用 `analyze_inspection`

## MCP

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

## 输出示例

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

## 提交新的检查点或字段

如果要提交新的 inspection field 或 checkpoint，请按照 [SKILL.md](SKILL.md) 的格式填写
GitHub Issue。Issue 至少应说明：

1. 要解决的巡检问题。
2. 现有字段或模板为什么不够用。
3. 新字段的 `id`、`type` 和允许值。
4. 正面和反面的图片证据。
5. 至少三个正面和三个反面示例引用。
6. 一个 REST 或 MCP 请求示例。
7. 一个预期响应示例。

不要提交客户图片、机密文件、真实服务地址、API key、SAS URL 或其他秘密信息。

[English README](README.md)
