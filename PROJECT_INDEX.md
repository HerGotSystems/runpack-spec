# RunPack — Project Index

## Name Transition

RunPack is the public name for the system formerly called AIP Protocol.

Legacy names:

```text
AIP
AIP Protocol
AI Protocol
AI Prompt Protocol
```

Public positioning:

```text
RunPack — task packets for user-run AI
```

Short tagline:

```text
Make a pack. Run it anywhere.
```

## Purpose

RunPack is a portable task packet format.

Apps generate structured task packets. Users run those packets with their own AI systems.

The point is not to create another agent protocol. The point is to make AI tasks portable, readable, copy-pasteable and model-independent.

## Core Rule

Applications do not require direct ownership of AI APIs.

Instead:

```text
App
↓
RunPack
↓
User AI
↓
Result
```

## What RunPack Is

RunPack is:

- a portable task packet
- human-readable
- machine-readable
- AI-model independent
- provider independent
- copy-paste friendly
- useful for BYO-AI workflows

## What RunPack Is Not

RunPack is not:

- an agent protocol
- a hosted AI service
- a model provider
- a tool-calling framework
- a replacement for MCP or agent-to-agent protocols

## Goals

- Model independence
- Provider independence
- Transparent task exchange
- Human-readable and machine-readable formats
- No owner API-credit burn for public apps
- Easy export from apps
- Easy import back into apps later

## Connected Projects

- HumanX
- Omniforge System Forge
- NLDS
- future AI-assisted tools

## Current Problems / Next Work

- Update README header to `RunPack (formerly AIP Protocol)`
- Update spec internal version header to `RunPack v1.1`
- Add one-liner to spec: `Legacy name: AIP / AI Prompt Protocol. Renamed for clarity.`
- Update examples from AIP wording to RunPack wording
- Rename repo from `aip-protocol` to `runpack-spec` only after docs are updated
- Update references in `mv-workspace`, `HumanX`, `Omniforge`, and any public portals

## Naming Decision

Canonical project name:

```text
RunPack
```

Canonical repo name target:

```text
runpack-spec
```

Canonical description:

```text
Portable task packets for user-run AI.
```

## Critical Instruction

Protect AI portability and avoid lock-in to specific AI vendors.

Do not describe RunPack as another AI infrastructure protocol unless comparing it explicitly against agent protocols.

The product metaphor is packet, not protocol.
