# RunPack — task packets for user-run AI

![Status: Public Draft](https://img.shields.io/badge/status-public_draft-1f6feb)
![Version: 1.1](https://img.shields.io/badge/version-1.1-111111)
![License: CC BY 4.0](https://img.shields.io/badge/license-CC_BY_4.0-0a7f3f)

**RunPack** is an open packet format for AI-agnostic software.

Apps generate structured task packets.  
Users run them with any AI they already have.  
Results can be pasted or imported back into the app.

No app-owner API keys. No public inference costs. No vendor lock-in.

> Make a pack. Run it anywhere.

---

## 30-second explanation

Most AI apps call a model API directly:

```text
App → API provider → AI result
```

RunPack flips that:

```text
App → RunPack packet → User's AI → Result → App
```

The app does not need to host inference. It only needs to create a clear task packet and understand the result.

---

## Why this matters

A public app with 1,000 daily active users running AI tasks
costs the developer nothing with RunPack. The same workflow
on a hosted API costs real money every day, whether users
get value or not.

RunPack shifts inference cost to the people who already
chose to pay for AI — which is increasingly everyone.
ChatGPT and Claude subscriptions are mainstream. The user
who already has AI is the normal user now, not the exception.

---

## Minimal packet

```json
{
  "runpack_version": "1.1",
  "legacy_aip_version": "1.1",
  "aip_version": "1.1",
  "packet_type": "runpack_task",
  "task_id": "doc-001",
  "task": "document_summary",
  "input": {
    "text": "[USER_INPUT]\nPaste text here.\n[/USER_INPUT]"
  },
  "rules": ["3_sentences_maximum", "plain_language", "no_preamble"],
  "output_format": "text"
}
```

The compatibility fields are deliberate. **RunPack is the public name. AIP remains the v1.x compatibility layer.**

---

## Who this is for

RunPack is useful when you are building:

- small public tools that should not pay for everyone else's AI usage
- browser-first apps with manual or semi-automated AI handoff
- educational tools, creative tools, analysis tools, games, and workflow generators
- software that should work with ChatGPT, Claude, Gemini, local models, and future systems
- apps where the user should see and control what is sent to AI

### RunPack is not the right fit if:

- Your app needs sub-second AI responses — the manual-paste
  flow adds latency by design
- Your workflow is fully automated with no human in the loop
- You need guaranteed output consistency across all users —
  different AI models produce different results for the same packet
- Your product value *is* the AI curation — if you have
  proprietary prompting or fine-tuning that defines your product,
  keeping that in-house makes sense

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

## Quick start for app builders

1. Pick a task type, such as `document_summary`, `classify`, `rewrite`, or a custom namespaced type.
2. Put the user's data inside `input`.
3. Put behavioural constraints inside `rules`.
4. Define the expected output with `output_format` and, for JSON, a `return` schema.
5. Let the user copy/open the packet in their AI.
6. Validate the result before importing it back.

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

## Tools using RunPack

- **HumanX** — claim pressure analysis packets via RunPack
- **Omniforge System Forge** — generates RunPack v1.1 packets 
  for system_generation and framework_expansion via the RunPack Packet modal

Add your tool here when it can generate or consume RunPack packets.

---

## What's in this repo

```text
docs/
  index.html           — front door / GitHub Pages entry point
  console.html         — interactive RunPack generator
  AIP-landing.html     — legacy path, RunPack-first landing page
spec/
  AIP-v1.1.md          — legacy filename, canonical RunPack v1.1 spec until path migration
examples/
  basic-task.txt       — minimal block format task
  json-mode-task.json  — JSON mode task packet
  json-mode-result.json — JSON mode result packet
  stateful-session.txt — multi-turn session with history
  retry-packet.txt     — retry packets for all six error codes
reference/
  AIP-v1.1-spec.html   — legacy path, RunPack-first rendered spec
```

File names still contain `AIP` during the transition. Public wording is now RunPack.

---

## Legacy name

RunPack was previously called **AIP / AI Prompt Protocol**. The old name sounded like another abstract AI protocol. The new name describes the actual object: a pack of instructions and data that the user can run anywhere.

Existing `aip_version` fields are retained for v1.x compatibility and should not be removed yet.

---

## Contributing

See `CONTRIBUTING.md` for the full process.

- Spec issue? Open a `spec-clarification` or `spec-bug` issue.
- New task type? Open a `task-type-proposal` issue and include a working example.
- Structural change? Open a `major-proposal` issue. It requires independent implementation.

---

## License

CC BY 4.0 — free to read, use, implement, fork, and build on.

---

Build once. Run with any AI.
