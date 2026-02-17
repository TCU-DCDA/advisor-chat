# AI Assistant Reference - AddRan Advisor

> This file provides context for AI coding assistants (GitHub Copilot, Claude, etc.) working on this project.

**Live Site:** https://sandra.digitcu.org

## Project Overview

A Firebase-based chatbot helping students explore majors, minors, and certificates in TCU's AddRan College of Liberal Arts. Uses Claude API (Sonnet) for conversational AI.

## Key Files to Know

| File | Purpose |
|------|---------|
| `functions/index.js` | Main API endpoint — handles chat requests, builds context, calls Claude |
| `functions/manifest-loader.js` | Fetches and caches wizard manifests (English, DCDA) with TTL + fallbacks |
| `functions/manifest-to-context.js` | Converts wizard manifests into Sandra context blocks |
| `functions/wizard-registry.json` | Registry of manifest URLs per department wizard |
| `public/app.js` | Frontend JavaScript — chat UI logic, message handling |
| `public/index.html` | Chat interface HTML |
| `functions/programs.csv` | Program data (majors, minors, certificates with URLs) |
| `functions/core-curriculum.json` | TCU Core Curriculum requirements |
| `functions/support-resources.csv` | Redirect URLs for off-topic questions |
| `functions/program-data/` | Static fallback JSON files (used when manifests unavailable) |

## System Prompt Location

The chatbot's personality and rules are defined in `functions/index.js` in the `SYSTEM_PROMPT` constant (~line 108). Key behaviors:

- Persona: "Sandra" — friendly AddRan advisor
- Keep responses under 100 words
- Use bold and bullet lists for formatting (no markdown headers)
- Only use provided data — never make up locations/phone numbers
- Redirect off-topic questions with appropriate URLs

## Data Flow

1. User sends message → `POST /api`
2. Function loads: programs.csv, core-curriculum.json, support-resources.csv
3. Manifest loader fetches live wizard manifests (English, DCDA) with 1-hour TTL caching; falls back to Firestore cache → static `program-data/*.json`
4. Manifest-to-context converts valid manifests into Sandra context blocks
5. Fetches abbreviations from Firestore (`config/abbreviations`)
6. Builds context string with all program data
7. Calls Claude with system prompt + context + conversation history
8. Logs conversation to Firestore (`conversations` collection)
9. Returns response + updated conversation history

## When Modifying

- **Adding programs:** Update `functions/programs.csv` and add a JSON file in `functions/program-data/`
- **Updating wizard department data:** Data flows automatically via manifests from English/DCDA wizards; update static fallbacks in `functions/program-data/` as backup
- **Changing personality:** Edit `SYSTEM_PROMPT` in `functions/index.js`
- **Adding redirect topics:** Update `functions/support-resources.csv`
- **Adding a new wizard department:** Add entry to `functions/wizard-registry.json` and add static fallback files

## Sensitive Topics to Redirect

- Academic probation
- Mental health concerns
- Financial hardship
- Any non-AddRan academic questions

## Local Development

```bash
cd functions && npm install && cd ..
firebase emulators:start
```

Requires `functions/.secret.local` with `ANTHROPIC_API_KEY=...`
