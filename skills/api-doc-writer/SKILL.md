---
name: api-doc-writer
description: API integration document writer for generating standardized API documentation in Chinese technical style. Use when the user needs to write, create, or fill out API interface documentation, including request/response parameter tables, example JSON payloads, and structured API descriptions. Triggers on requests like "写接口文档", "帮我写API文档", "生成对接文档", "更新对接文档", or any request to document an API endpoint with its parameters and responses. 接口对接文档助手
---

# API 对接文档助手

Generate structured API integration documentation following the standard format below.

## Document Structure

### When documenting multiple APIs under the same route group

If multiple APIs share a common route prefix (e.g. `/api/v1/user/`), prepend the document with a **Route Summary Table** before the individual API sections:

include these fields:

- alias | 别名
- method | 方法
- url | 接口地址
- status | 开发状态
- description | 描述

**Development status values** (use consistently):

- done
- in progress
- not started
- deprecated

If the user does not specify a status, default to `not started`.

After the summary table, document each API individually in the order they appear in the table.

### For a single API (no route group)

Skip the summary table and go directly to the API detail section.

---

### Each API detail section — produce in this order

1. **Alias Name** — camelCase alias (e.g. `getUserList`)
2. **Description** — one-sentence description of what the API does
3. **Request URL** — full URL or path
4. **Request Method** — GET / POST / PUT / DELETE / PATCH
5. **Request Parameters Table** — all request fields
6. **Response Result Table** — all response fields
7. **Request Example** — `XXXXRequest` JSON block
8. **Response Example(s)** — `XXXXResponse` JSON block(s)

---

## Table Formats 表字段说明

### Request Parameters Table

| 参数名 | 类型 | 必填 | 约束 | 说明 |
|--------|------|------|------|------|
| param  | String | 是  | 1–255 字符 | 描述 |

Include: parameter name, type, required (是/否), description.  
Add a **Header** section row if the API requires special headers.

### Response Result Table

| 参数名 | 类型 | 说明 |
|--------|------|------|
| code   | Integer | 状态码 |
| msg    | String  | 提示信息 |
| data   | Object  | 返回数据 |

For nested objects, document child fields as `data.fieldName` rows beneath the parent, or use a sub-table with a note.

---

## Example JSON Blocks

- Name request example: `GetUserListRequest`, `CreateOrderRequest`, etc.
- Name response example: `GetUserListResponse`, `CreateOrderResponse`, etc.
- If the response has **multiple result shapes** (e.g. success vs. error, or different `data` structures by type), show **two examples** with clear labels (e.g. `// 成功示例`, `// 失败示例` or `// 类型A`, `// 类型B`). Do not show more than two; cover remaining variations only in the table.
- Format all JSON with 2-space indentation.

---

## Example

[Example](references/example.md)

---

## Rules

- Use Chinese for all table headers, labels (描述、请求地址、请求方式、请求参数、返回结果、请求示例、返回示例), and field descriptions.
- Keep alias names and JSON keys in English camelCase.
- When the user provides incomplete info (e.g. missing field types), make reasonable inferences and note them with `// 推断` in the JSON or a footnote.
- If documenting multiple APIs in one session, repeat the full structure for each, separated by `---`.
- Read `references/example.md` for a complete worked example when uncertain about format.
