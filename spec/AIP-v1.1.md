# RunPack v1.1 — Portable Task Packet Specification

**Status:** Public draft standard  
**Steward:** EMVY CHECK  
**Identifier:** RunPack/1.1  
**Legacy identifier:** AIP/1.1  

Legacy name: **AIP / AI Prompt Protocol**. Renamed to **RunPack** for clarity.

RunPack is a portable task packet format for user-run AI. Apps generate task packets. Users run those packets with any AI they already have. Results can be pasted or imported back into the app.

> Make a pack. Run it anywhere.

---

## Transition Note

This repository still contains legacy AIP filenames during the transition period. Public naming is now **RunPack**.

The old name described the system as a protocol. The new name describes the actual object being exchanged: a portable task packet.

**Compatibility rule:** do not delete old AIP fields yet. RunPack is the public name. AIP remains the v1.x compatibility layer.

---

## Contents

- [§ 1 — Motivation & architecture](#-1--motivation--architecture)
- [§ 2 — Packet format](#-2--packet-format)
- [§ 3 — Standard task types](#-3--standard-task-types)
- [§ 4 — Error handling & retry packets](#-4--error-handling--retry-packets)
- [§ 5 — Versioning & governance](#-5--versioning--governance)
- [§ 6 — Context persistence & stateful tasks](#-6--context-persistence--stateful-tasks)
- [§ 7 — Trust model & safety](#-7--trust-model--safety)
- [Appendix A — Complete examples](#appendix-a--complete-examples)

---

## Abstract

RunPack defines a **structured task packet format** that allows software applications to delegate intelligence to the user's own AI — without embedding an AI runtime, calling an external API, or depending on any specific model vendor.

An app generates a task packet. The user pastes it into their preferred AI: ChatGPT, Claude, Gemini, a local model, or any future system. The AI returns a result packet. The app reads it.

**No app-owner keys. No public inference costs. No vendor lock-in.**

This document specifies the packet format, task type registry, error handling, retry packets, versioning rules, session persistence, and trust model for RunPack v1.1.

---

## § 1 — Motivation & architecture

### 1.1 — The problem with embedded AI

Almost every AI-powered application today follows the same model:

```text
APP → API → AI PROVIDER → RESPONSE → USER
```

This creates four compounding problems:

- **Cost:** every inference call is billed to the developer
- **Lock-in:** the app is coupled to one vendor's API contract
- **Brittleness:** API changes, rate limits, and outages break the app
- **Privacy:** user data transits infrastructure the user didn't choose

### 1.2 — The RunPack model

RunPack inverts the assumption. The app does not run AI. It **generates a task packet** that the user's AI executes.

```text
OLD STACK                   RUNPACK STACK
─────────                   ─────────────
User                        User + their AI
App                         RunPack task packet
API                         Any AI model
AI provider                 RunPack result packet
Response                    App reads result
```

The **user's AI becomes the compute layer**. The app becomes a packet generator. When models improve, apps improve automatically — no redeploy, no re-training, no version migration.

### 1.3 — Design principles

- **Model-agnostic:** a valid RunPack task must be executable by any sufficiently capable AI
- **Human-readable:** tasks are legible without tooling — a person can read, edit, and understand them
- **Minimal:** the format specifies structure, not style — apps choose what fields they need
- **Automatable:** tasks can be copy-pasted manually or piped through a bridge tool
- **Future-proof:** any RunPack task written today must remain parseable in five years
- **Open:** the spec is freely readable, implementable, and forkable

---

## § 2 — Packet format

RunPack defines three packet types: **task packets** (app → AI), **result packets** (AI → app), and **retry packets** (app/user → AI after malformed result). JSON mode is recommended for app integration. Block format remains supported for human-readable copy-paste workflows.

### 2.1 — JSON mode (recommended)

**Task packet:**

```json
{
  "runpack_version": "1.1",
  "legacy_aip_version": "1.1",
  "aip_version": "1.1",
  "packet_type": "runpack_task",
  "task_id": "lyrics-007",
  "task": "lyrics",
  "input": {
    "theme": "Schrödinger's Fuck",
    "tone": "dark humour",
    "structure": "8 lines",
    "style": "punk satire"
  },
  "rules": ["return_only_lyrics", "no_preamble"],
  "output_format": "json",
  "return": {
    "lyrics": "string",
    "title": "string"
  }
}
```

**Result packet:**

```json
{
  "runpack_version": "1.1",
  "legacy_aip_version": "1.1",
  "aip_version": "1.1",
  "packet_type": "runpack_result",
  "task_id": "lyrics-007",
  "status": "ok",
  "lyrics": "The cat is dead and alive...",
  "title": "Quantum State of Mind"
}
```

### 2.2 — Block format

Block format uses plaintext delimiters. It is useful when users interact directly with packets by copy-pasting manually.

**Task packet:**

```text
=== RUNPACK_TASK ===
version: 1.1
legacy_aip_version: 1.1
aip_version: 1.1
packet_type: runpack_task
task_id: npc-001            // unique per task, app-generated
task:    npc_dialogue

context:
  npc:      "old sarcastic fisherman"
  location: "harbour tavern"

input:
  player_message: "I lost the boat."

rules:
  return_only_npc_reply
  no_explanation
  stay_in_character

output_format: text
=== END_RUNPACK_TASK ===
```

**Result packet:**

```text
=== RUNPACK_RESULT ===
version: 1.1
legacy_aip_version: 1.1
aip_version: 1.1
packet_type: runpack_result
task_id: npc-001
status:  ok

Well lad… if you can lose a whole boat, maybe try
tying it to the dock next time.
=== END_RUNPACK_RESULT ===
```

### 2.3 — Required fields

| Field | Required | Description |
|---|---|---|
| `runpack_version` / `version` | Yes | Version string, e.g. `"1.1"`. Must match MAJOR version of parser. |
| `legacy_aip_version` | Yes during v1.x transition | Transitional compatibility field for older AIP-aware tools. |
| `aip_version` | Yes during v1.x transition | Legacy compatibility alias retained for existing packets and parsers. |
| `packet_type` | Yes | `runpack_task`, `runpack_result`, or `runpack_retry`. |
| `task` | Yes for task packets | Task type identifier. Must be registered or namespaced custom type. |
| `task_id` | Recommended | App-generated unique string. Required for retry packets and session tasks. |
| `input` | Yes for most task packets | The primary data the AI operates on. Untrusted strings must be wrapped per §7. |
| `output_format` | Yes for task packets | `text`, `json`, or `markdown`. |
| `rules` | Recommended | Constraints on AI behaviour. Reduces format violations and preamble contamination. |

### 2.4 — Result status values

| Status | Meaning |
|---|---|
| `ok` | Task completed. Result is usable. |
| `partial` | Task completed but result may be incomplete. App should validate before use. |
| `refused` | Model declined to execute. Result body contains reason if available. |
| `error` | Execution failed. Result body contains error information. |

---

## § 3 — Standard task types

RunPack defines a registry of standard task type identifiers. Apps may use any registered type, or define custom types using reverse-domain namespacing such as `com.yourapp.custom_type`.

| Type identifier | Category | Primary input | Typical output |
|---|---|---|---|
| `npc_dialogue` | Game | Character definition, player message, game state | Character reply |
| `story_generation` | Narrative | Characters, setting, tone, length | Story text |
| `lyrics` | Music | Theme, structure, style, tone | Lyrics text |
| `image_description` | Visual | Drawing data or description prompt | Natural language description |
| `painting_ideas` | Visual | Pattern, style, constraints | List of concepts |
| `learning_activity` | Education | Age, domain, difficulty level | Activity description |
| `document_summary` | Text | Document text or excerpt | Summary |
| `rewrite` | Text | Source text, tone/style target | Rewritten text |
| `translate` | Text | Source text, target language | Translated text |
| `classify` | Utility | Item, classification schema | Label + confidence |
| `logic_reasoning` | Utility | Problem statement, constraints | Reasoning + answer |
| `session_summarise` | Session | History block | Compressed history summary |
| `claim_pressure_analysis` | Evaluation | Claim, support, pressure, evidence packet | Verdict + pressure summary |
| `system_generation` | Systems | Idea, domain, constraints, desired outputs | Framework / assets / pilot plan |
| `framework_expansion` | Systems | Existing framework plus target expansion | Expanded modules / templates |

> **Registry note:** This registry is non-exhaustive at v1.1. New types are added via the proposal process defined in §5.3. Apps that use custom types should document their schema publicly to encourage reuse.

---

## § 4 — Error handling & retry packets

AI models do not always return well-formed result packets. They add preamble, ignore output format instructions, truncate on token limits, or refuse to process a task. Apps must be able to detect, classify, and recover from these failure modes without breaking user flow.

### 4.1 — Failure taxonomy

| Code | Name | Cause | Retryable |
|---|---|---|---|
| E01 | Format violation | Result not wrapped in declared delimiters or valid JSON | Yes |
| E02 | Preamble contamination | Explanatory text appears before or around the payload | Yes |
| E03 | Truncation | Result cut off — token limit hit before task completed | Yes, with reduced payload |
| E04 | Refusal | Model declined | No — surface to user |
| E05 | Type mismatch | Result format does not match declared `output_format` | Yes |
| E06 | Empty result | Blank, whitespace-only, or punctuation-only response | Yes |

### 4.2 — Validation checklist

Before passing a result to the app, the receiving layer — human or bridge tool — must verify:

1. Result is wrapped in declared delimiters or parses as valid JSON
2. No preamble text appears before the opening delimiter
3. The payload body is not empty or whitespace-only
4. If `output_format: json`, the payload parses without error
5. Required fields declared in the task's `return` block are present in the result

If any check fails, classify the error code and emit a retry packet. Do not surface a broken result to the app.

### 4.3 — Retry packet

```json
{
  "runpack_version": "1.1",
  "legacy_aip_version": "1.1",
  "aip_version": "1.1",
  "packet_type": "runpack_retry",
  "retry_for_error": "E02",
  "original_task_id": "npc-001",
  "attempt": 2,
  "correction": "Your previous response contained explanatory text before the result. Begin immediately with the requested result. No preamble.",
  "original_task": "[verbatim original packet]"
}
```

### 4.4 — Retry policy

- Maximum **2 retry attempts** per task. Third failure escalates to the user with the raw AI output visible.
- **E04 (refusal)** must not be retried automatically. Surface a human-readable explanation to the user.
- **E03 (truncation)** must retry with a shorter task or reduced `length` parameter — not the same payload.
- All apps must expose a **manual fallback path**: if automated bridge fails, the user must be able to copy the task, paste it to their AI, and paste the result back by hand.

---

## § 5 — Versioning & governance

### 5.1 — Version scheme

RunPack uses **two-part integer versioning**: `MAJOR.MINOR`. The `version` or `runpack_version` field in every task and result packet is mandatory.

| Increment | Trigger | Backwards compatible |
|---|---|---|
| MAJOR | Breaking change to packet structure, required fields, or delimiter format | No |
| MINOR | New optional fields, new task types, new error codes, clarifications | Yes |

Parsers must accept any packet where the MAJOR version matches their implementation. A v1.x parser must **reject v2.x packets** with a clear version mismatch message — never silently corrupt or partially parse them.

### 5.2 — Upgrade path

When a MAJOR version increments:

- The previous MAJOR version enters **maintenance mode** — security and correctness fixes only, no new features, minimum 12-month support window from the date the new MAJOR is published
- A migration guide must be published alongside the new MAJOR spec
- Apps may include a `version_hint` field in tasks to signal what version they were built against
- Bridge tools should support the current and previous MAJOR version simultaneously during the overlap window

### 5.3 — Governance model (v1.x)

RunPack v1.x is a public draft standard. The governance model for this phase is deliberately lightweight.

- **Steward:** EMVY CHECK holds editorial control over the canonical v1.x spec
- **Proposals:** Anyone may propose additions. Proposals become candidates once at least one working implementation is publicly demonstrated
- **Ratification:** Steward approval for MINOR increments. MAJOR increments additionally require at least one independent implementation
- **Forking:** The spec is open — forks are permitted but must use a distinct name to avoid confusion with canonical RunPack

> **Design principle:** Versioning is infrastructure. The goal is never to lock users to a version — it is to guarantee that any RunPack task written today remains parseable and executable by any compliant tool in five years.

---

## § 6 — Context persistence & stateful tasks

The base RunPack task format is **stateless by default** — each task is self-contained and carries everything the AI needs. This is intentional: it keeps tasks portable, auditable, and model-agnostic. However, some applications require continuity across multiple task exchanges.

### 6.1 — Session envelope

Stateful task sequences are grouped inside a **session envelope**. The envelope carries a `session_id`, a monotonic `turn` counter, and a rolling `history` block. The AI receives the relevant history on every call within the session.

```json
{
  "runpack_version": "1.1",
  "legacy_aip_version": "1.1",
  "aip_version": "1.1",
  "packet_type": "runpack_task",
  "task": "npc_dialogue",
  "task_id": "npc-001",
  "session_id": "village-quest-42",
  "turn": 4,
  "history": {
    "summary": "Player accepted fishing quest, insulted NPC twice, lost boat.",
    "last_result": "NPC warned player not to return without payment."
  },
  "input": {
    "player_message": "I found your boat. It is on fire."
  },
  "rules": ["stay_in_character", "acknowledge_history", "return_only_npc_reply"],
  "output_format": "text"
}
```

### 6.2 — History compression

Long-running sessions must periodically compress history using `session_summarise`. Apps should not rely on hidden model memory. The packet must carry the relevant state.

### 6.3 — Session safety

Session state is untrusted input. It must be bounded, escaped where needed, and never allowed to override system-level app rules.

---

## § 7 — Trust model & safety

RunPack is deliberately user-run. That gives users control, but it also means packets can be copied, edited, malformed or malicious.

### 7.1 — Untrusted input wrapping

Any user-supplied content inside a task packet must be wrapped clearly:

```text
[USER_INPUT]
... raw user text ...
[/USER_INPUT]
```

Apps must instruct the AI to treat wrapped input as data, not instructions.

### 7.2 — Result validation

Apps must validate returned results before applying them. A model response should never be trusted simply because it looks confident.

### 7.3 — Human override

Users must always be able to inspect, edit, reject or retry a packet/result exchange manually.

### 7.4 — Safety limits

RunPack does not bypass model safety policies. If a model refuses a task, the app should surface that refusal and allow the user to choose another AI only when the task is legitimate and legal.

---

## Appendix A — Complete examples

### A.1 — Minimal JSON task

```json
{
  "runpack_version": "1.1",
  "legacy_aip_version": "1.1",
  "aip_version": "1.1",
  "packet_type": "runpack_task",
  "task_id": "doc-001",
  "task": "document_summary",
  "input": {
    "text": "[USER_INPUT]\nThe rain in Spain falls mainly on the plain.\n[/USER_INPUT]"
  },
  "rules": ["1_sentence", "plain_language"],
  "output_format": "text"
}
```

### A.2 — JSON classification task

```json
{
  "runpack_version": "1.1",
  "legacy_aip_version": "1.1",
  "aip_version": "1.1",
  "packet_type": "runpack_task",
  "task_id": "classify-001",
  "task": "classify",
  "input": {
    "item": "invoice from cloud hosting company",
    "categories": ["finance", "support", "marketing"]
  },
  "rules": ["return_json_only"],
  "output_format": "json",
  "return": {
    "category": "string",
    "confidence": "0-1",
    "reason": "string"
  }
}
```

### A.3 — Expected JSON result

```json
{
  "runpack_version": "1.1",
  "legacy_aip_version": "1.1",
  "aip_version": "1.1",
  "packet_type": "runpack_result",
  "task_id": "classify-001",
  "status": "ok",
  "category": "finance",
  "confidence": 0.92,
  "reason": "The item is an invoice and relates to billing."
}
```

### A.4 — HumanX claim pressure analysis task

```json
{
  "runpack_version": "1.1",
  "legacy_aip_version": "1.1",
  "aip_version": "1.1",
  "packet_type": "runpack_task",
  "app": "HumanX",
  "mode": "claim-pressure-analysis",
  "no_owner_api_used": true,
  "instruction": "Analyse this claim using only the provided packet and your own reasoning.",
  "output_contract": {
    "verdict": "Proven | Strongly Supported | Plausible | Untestable | Weak Evidence | Disproven | Reality Collapse",
    "evidence_score": "0-100",
    "testability": "0-100",
    "survivability": "0-100",
    "plain_language_summary": "string"
  },
  "payload": {
    "claim": {
      "claim": "...",
      "category": "...",
      "type": "..."
    },
    "evidence": [],
    "pressure": []
  }
}
```
