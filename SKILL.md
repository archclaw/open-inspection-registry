# Submit a New Inspection Field or Checkpoint

## English

An **Observation** defines what can be checked in an image. A **Skill** defines why the user
wants the check. Keep these two concepts separate.

## Before submitting

Check existing templates and observations first. A new field is appropriate only when the
current fields cannot express the requested checkpoint. The field must be observable from one
to three images and work through the existing REST and MCP `analyze_inspection` inputs.

Never submit secrets, customer images, private documents, real API URLs, SAS URLs, or
credentials. Use placeholders or public, non-sensitive example references.

## Required field format

Submit one YAML definition with this shape:

```yaml
id: label_status
name: Label Status
description: |
  Check whether the material label is visible and readable.
type: classifier
values:
  - present
  - missing
  - damaged
  - unknown
examples:
  positive:
    - examples/label-present-01.jpg
    - examples/label-missing-01.jpg
    - examples/label-damaged-01.jpg
  negative:
    - examples/not-a-label-01.jpg
    - examples/irrelevant-scene-01.jpg
    - examples/unclear-image-01.jpg
```

Every field must include:

- `id`: lowercase `snake_case`, stable, and specific
- `name`: a short human-readable name
- `description`: the visual evidence being measured
- `type`: `boolean`, `classifier`, `count`, or `extraction`
- `values`: allowed classifier values, including `unknown` when evidence may be insufficient
- `examples`: at least three positive and three negative references

Use `unknown` when the image is too dark, blurry, incomplete, or otherwise insufficient. Do
not force a positive or negative value when the evidence is not visible.

## Optional Skill proposal

If the field supports a repeatable user goal, include a Skill proposal separately:

```yaml
id: warehouse_label_check
objective: |
  Verify whether warehouse materials have visible and readable labels.
expected_output:
  - Missing label
  - Damaged label
  - Label readable
observations:
  - label_status
```

The Skill describes the user's inspection intent. It must not invent facts unsupported by the
selected observations.

## Issue or pull request checklist

Explain:

1. What inspection problem the field solves.
2. Why existing fields or templates are insufficient.
3. The proposed field ID, type, and allowed values.
4. Positive and negative visual evidence.
5. At least three positive and three negative example references.
6. A REST or MCP request using `template_slug`, `skill`, `user_skill`, or `prompt`.
7. A sample response showing the expected observation value.

Maintainers may revise names, values, examples, or template membership to keep the registry
consistent and backward compatible.

## 中文

本文件用于提交新的 Observation field 或 inspection checkpoint。

**Observation** 定义“图片中要检查什么”；**Skill** 定义“用户为什么要进行这个检查”。
请不要把检查字段和用户意图混写在一起。

### 提交前

请先检查现有模板和观察项。只有当现有字段无法表达新的检查点时，才建议新增字段。新字段
必须能够从一到三张图片中观察，并且能够通过现有 REST API 或 MCP 的
`analyze_inspection` 使用。

绝对不要提交密钥、客户图片、私有文件、真实 API 地址、SAS URL 或凭据。示例请使用占位符
或公开且无敏感信息的引用。

### 必填字段格式

提交一个 YAML 定义，格式如下：

```yaml
id: label_status
name: Label Status
description: |
  检查物料标签是否可见并且清晰可读。
type: classifier
values:
  - present
  - missing
  - damaged
  - unknown
examples:
  positive:
    - examples/label-present-01.jpg
    - examples/label-missing-01.jpg
    - examples/label-damaged-01.jpg
  negative:
    - examples/not-a-label-01.jpg
    - examples/irrelevant-scene-01.jpg
    - examples/unclear-image-01.jpg
```

每个字段必须包含：

- `id`：使用稳定、明确的 lowercase `snake_case`
- `name`：简短的人类可读名称
- `description`：说明要测量的可见图片证据
- `type`：`boolean`、`classifier`、`count` 或 `extraction`
- `values`：classifier 的允许值；证据可能不足时必须包含 `unknown`
- `examples`：至少三个正面和三个反面示例引用

当图片太暗、模糊、不完整或无法判断时使用 `unknown`。不要在图片证据不足时强行返回
正面或反面结果。

### 可选的 Skill 提案

如果这个字段支持一个可重复使用的用户目标，可以另外提交 Skill：

```yaml
id: warehouse_label_check
objective: |
  检查仓库物料是否具有清晰可读的标签。
expected_output:
  - 缺少标签
  - 标签损坏
  - 标签清晰可读
observations:
  - label_status
```

Skill 应描述用户的巡检意图，不能创造所选观察项无法从图片证实的事实。

### Issue 或 Pull Request 清单

请说明：

1. 新字段解决什么巡检问题。
2. 为什么现有字段或模板不够用。
3. 建议的字段 ID、类型和允许值。
4. 什么图片证据属于正面或反面结果。
5. 至少三个正面和三个反面示例引用。
6. 一个使用 `template_slug`、`skill`、`user_skill` 或 `prompt` 的 REST/MCP 请求示例。
7. 一个展示预期 observation 值的响应示例。

维护者可以为了保持公开 registry 的一致性和向后兼容性，调整字段名称、允许值、示例或
模板归属。
