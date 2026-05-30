# RunPack — task packets for user-run AI

![Status: Public Draft](https://img.shields.io/badge/status-public_draft-1f6feb)
![Version: 1.1](https://img.shields.io/badge/version-1.1-111111)
![License: CC BY 4.0](https://img.shields.io/badge/license-CC_BY_4.0-0a7f3f)

**RunPack** is an open packet format for AI-agnostic software.

Apps generate structured task packets.  
Users run them with any AI they already have.  
No app-owner API keys. No public inference costs. No vendor lock-in.

Legacy name: **AIP / AI Prompt Protocol**. Renamed to RunPack for clarity.

> Make a pack. Run it anywhere.

---

## Repository rename status

This repository is ready to be renamed:

```text
HerGotSystems/aip-protocol
→ HerGotSystems/runpack-spec
```

GitHub should preserve redirects after the rename, but public docs now use **RunPack** as the primary name. AIP remains only as the v1.x compatibility layer and in legacy filenames that still need a later file-path migration.

---

## Start here

- Try the console — generate a task packet in 30 seconds
- Landing page — what RunPack is and why it exists
- Spec — the full packet specification
- Examples — ready-to-use packet templates

---

## How it works

```text
=== RUNPACK_TASK ===
version: 1.1
legacy_aip_version: 1.1
aip_version: 1.1
packet_type: runpack_task
task: document_summary

input:
  text: [USER_INPUT]
  ... any document ...
  [/USER_INPUT]

rules:
  3_sentences_maximum
  plain_language
  no_preamble

output_format: text
=== END_RUNPACK_TASK ===
```

1. App generates a packet like this.  
2. User pastes it into ChatGPT, Claude, Gemini, or any local model.  
3. Result comes back in a structured format the app can read.

That's it. No model API required by the app. Works with any AI, now and in the future.

---

## Why this exists

Most "AI apps" are wrappers around a model API. That creates the same failure pattern every time:

- developer pays per inference
- app inherits vendor risk
- product breaks when APIs change
- user has no say in which AI they trust

RunPack flips the model.

The app emits a packet. The user's AI executes it. The result comes back structured.

The AI is already in the user's pocket — RunPack just gives it a job to do.

---

## What RunPack is

RunPack is:

- a portable task packet
- human-readable
- machine-readable
- model-independent
- provider-independent
- copy-paste friendly
- useful for BYO-AI workflows

RunPack is not:

- an agent protocol
- a hosted AI service
- a model provider
- a tool-calling framework
- a replacement for MCP or agent-to-agent protocols

---

## Tools using RunPack

Early / in-progress use:

- **HumanX** — RunPack-first claim analysis packets with AIP legacy route retained
- **Omniforge System Forge** — candidate BYO-AI packet workflow for system generation and framework expansion

Add your tool here when it can generate or consume RunPack packets.

---

## What's in this repo

```text
docs/
  index.html          — front door (GitHub Pages entry point)
  console.html        — interactive RunPack generator
  AIP-landing.html    — legacy path, RunPack-first landing page
spec/
  AIP-v1.1.md         — legacy filename, canonical RunPack v1.1 spec until path migration
examples/
  basic-task.txt      — minimal block format task
  json-mode.json      — JSON mode task and result
  stateful-session.txt — multi-turn session with history
  retry-packet.txt    — retry packets for all six error codes
reference/
  AIP-v1.1-spec.html  — legacy path, RunPack-first rendered spec
```

File names still contain `AIP` during the transition. Public wording is now RunPack.

---

## Standard task types

| Type | What it does |
|---|---|
| `document_summary` | Summarise any document |
| `rewrite` | Change the tone or style of text |
| `translate` | Translate to any language |
| `classify` | Sort an item into your categories |
| `lyrics` | Write song lyrics |
| `painting_ideas` | Generate visual concepts |
| `learning_activity` | Create an activity for a learner |
| `story_generation` | Write a short story |
| `logic_reasoning` | Work through a problem |
| `session_summarise` | Compress a conversation history |
| `claim_pressure_analysis` | Analyse a claim using evidence and pressure points |
| `system_generation` | Generate a system blueprint from an idea |
| `framework_expansion` | Expand an existing framework into modules/templates |

Custom types use reverse-domain namespacing:

```text
com.yourapp.task_name
```

---

## Contributing

See `CONTRIBUTING.md` for the full process.

Short version:

- Spec issue? Open a `spec-clarification` or `spec-bug` issue.
- New task type? Open a `task-type-proposal` issue and include a working example.
- Structural change? Open a `major-proposal` issue. It requires independent implementation.

---

## Repo files

| File | Purpose |
|---|---|
| `PROJECT_INDEX.md` | Current project map and rename status |
| `CHANGELOG.md` | What changed and when |
| `CONTRIBUTING.md` | How to propose changes |
| `GOVERNANCE.md` | Stewardship and decision model |
| `SECURITY.md` | How to report packet/spec-level risks |
| `CODE_OF_CONDUCT.md` | How we work together |
| `LICENSE.txt` | CC BY 4.0 — free to use and build on |

---

Build once. Run with any AI.
