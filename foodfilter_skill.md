---
name: foodfilter-skill
description: FoodFilter project skill for extracting restaurant names from links, screenshots, OCR text, social posts, or user-provided lists, then converting them into FoodFilter-compatible import JSON with BLACK or WHITE status. Use this whenever the user wants to analyze restaurant content, batch import shops into FoodFilter, normalize shop names for blacklist/whitelist workflows, prepare export/import payloads, or discuss FoodFilter-specific shop/group/user data structures.
---

# FoodFilter Skill

Use this skill when the task is about FoodFilter data, especially:

- extracting restaurant names from web pages, screenshots, OCR, or social content
- converting extracted results into FoodFilter import JSON
- normalizing shop names for FoodFilter matching
- reasoning about FoodFilter `Shop`, `Group`, or `User` data
- preparing structured data for FoodFilter black/white list workflows

Do not use this skill for general app coding tasks unless the task is specifically about FoodFilter data format or content ingestion.

## Core product context

FoodFilter is a mobile app for managing restaurant blacklists and whitelists.

- `BLACK`: the shop should trigger a warning
- `WHITE`: the shop should trigger a positive match

Users can add shops manually or from OCR/screenshot flows. The app later detects matching shop names on screen and shows reminders or visual indicators.

## Data model

### Shop

| Field | Type | Notes |
|---|---|---|
| `name` | `string` | Shop name |
| `status` | `"BLACK" \| "WHITE"` | Required |
| `notes` | `string` | Optional |
| `groupIds` | `string[]` | Optional |
| `createdAt` | `number` | Unix ms timestamp |

### Group

| Field | Type | Notes |
|---|---|---|
| `name` | `string` | Group name |
| `memberIds` | `string[]` | Member shop IDs |
| `ownerId` | `string` | Group owner |

### User

| Field | Type | Notes |
|---|---|---|
| `account` | `string` | User account |
| `vipStatus` | `enum` | VIP status |
| `prefs` | `object` | User preferences |

## Import format

When generating FoodFilter import data, use this structure:

```json
{
  "version": "1.0",
  "userId": "<current-user-id>",
  "shops": [
    {
      "name": "餐厅名称",
      "status": "BLACK",
      "notes": ""
    }
  ]
}
```

### Rules

- `status` must be exactly `BLACK` or `WHITE`
- every shop must include `name` and `status`
- `notes` may be empty
- `groupIds` is optional
- `createdAt` is optional
- do not invent nested custom fields

## Export format

When the task is about exporting FoodFilter data to another system, use this structure:

```json
{
  "version": "1.0",
  "userId": "<user-id>",
  "exportedAt": 1748000000000,
  "shops": [
    {
      "name": "餐厅名称",
      "status": "BLACK",
      "notes": "服务差",
      "groupIds": ["group_1"],
      "createdAt": 1747000000000
    }
  ],
  "groups": [
    {
      "name": "分组名称",
      "memberIds": ["shop_1"],
      "ownerId": "user_1"
    }
  ]
}
```

## Workflow

When asked to analyze content for FoodFilter, use this sequence:

1. Get the content.
   - link: read the page text
   - image or screenshot: OCR or visual extraction
   - social post: parse post text and visible shop names
2. Extract likely shop names.
3. Remove obvious non-shop noise like prices, distances, ratings, or UI labels.
4. Infer `BLACK` or `WHITE` from context.
5. If status is ambiguous, ask the user or clearly mark uncertainty.
6. Return FoodFilter-compatible JSON.

## Name normalization

### Use these principles

- Prefer full shop names over premature abbreviation.
- Keep different-language names separate unless the mapping is explicit.
- If a chain brand can be safely normalized, use the standard brand name.
- If the original name is overly long, shorten the main `name` and preserve the original in `notes`.

### Examples

- Good: `海底捞火锅`
- Risky: reducing `海底捞火锅` to `海底捞` too early
- Good: `X餐厅`
- Put original in notes: `X餐厅（朝阳区三里屯店）`

## Confidence handling

Extraction is imperfect. Common failure modes:

- prices or addresses mistaken for shop names
- nicknames or slang names that do not map cleanly
- mixed content where list boundaries are unclear

When confidence is low, note it in `notes`, for example:

```json
{
  "name": "寿司",
  "status": "WHITE",
  "notes": "低置信度，建议核对"
}
```

If there are many extracted shops, or several ambiguous ones, call that out explicitly before final JSON.

## Output contract

When this skill is used, prefer this answer shape:

### If the user wants import-ready data

1. short note about what was extracted and any ambiguity
2. one fenced `json` block containing the final FoodFilter payload

### If the user wants analysis before JSON

1. short list of extracted shops
2. note which ones are uncertain
3. then the final JSON block

## Constraints

- Never output statuses other than `BLACK` or `WHITE`.
- Do not merge multiple shops unless the source clearly refers to the same place.
- Do not hallucinate `userId`; leave a placeholder if none is provided.
- Keep the payload flat and simple.
