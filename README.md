# FoodFilter Skill

FoodFilter Skill is a lightweight skill file for AI agents such as Codex or Claude.

It helps an agent read restaurant-related content from links, screenshots, OCR text, social posts, videos, or manual lists, then convert the result into FoodFilter-compatible import JSON.

This repository currently contains only the skill definition. The full FoodFilter Agent runtime is not included yet.

## What It Does

- Extracts likely restaurant names from messy content
- Filters obvious UI noise, prices, ratings, addresses, and non-shop text
- Normalizes restaurant names for FoodFilter workflows
- Produces import-ready JSON for FoodFilter blacklist and whitelist records
- Keeps the output schema simple and compatible with FoodFilter App

## Install For Codex

Create a local skill folder:

```bash
mkdir -p ~/.codex/skills/foodfilter-skill
```

Download the skill file:

```bash
curl -L https://raw.githubusercontent.com/ReinhartL/foodfilter_agent/main/foodfilter_skill.md -o ~/.codex/skills/foodfilter-skill/SKILL.md
```

Restart Codex after installation.

## Usage

After installation, ask your agent to use `foodfilter-skill` when handling restaurant extraction or FoodFilter import tasks.

Example:

```text
Use foodfilter-skill to extract restaurants from this Xiaohongshu link and generate FoodFilter import JSON.
```

Expected output shape:

```json
{
  "version": "1.0",
  "userId": "<current-user-id>",
  "shops": [
    {
      "name": "餐厅名称",
      "status": "WHITE",
      "notes": ""
    }
  ]
}
```

`status` must be either `BLACK` or `WHITE`.

## Repository Contents

- `foodfilter_skill.md`: the skill source file
- `LICENSE.md`: non-commercial license notice

## License

This project is source-available for personal, research, and non-commercial use.

Commercial use is not permitted without written permission from Shanghai Raykit IntelliCore Artificial Intelligence Technology Co., Ltd..
See `LICENSE.md` for details.
