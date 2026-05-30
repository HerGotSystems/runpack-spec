# Release Notes — RunPack v1.1

RunPack v1.1 is the current public draft of the portable task packet format formerly called AIP / AI Prompt Protocol.

## What RunPack does

RunPack defines a structured way for software to generate AI tasks that users execute with their own preferred AI tools, instead of forcing every application to run a model API directly.

Apps generate packets. Users run them anywhere. Results can be pasted or imported back.

## Included in this release

- RunPack specification in `spec/`
- example task packets in `examples/`
- browser-readable spec in `reference/`
- public docs and console in `docs/`
- repository governance and contribution files for public collaboration

## Highlights in v1.1

- renamed the public project from AIP Protocol to RunPack
- clarified packet structure and task framing
- added compatibility fields: `runpack_version`, `legacy_aip_version`, `aip_version`, and `packet_type`
- added stronger examples for plain text, JSON mode, retry packets, and stateful sessions
- separated public-facing documentation from the core specification
- prepared the repository for public review and proposals

## Status

**Public draft.**

RunPack v1.1 is suitable for experimentation, reference implementations, discussion, and community feedback. It should be treated as a draft standard rather than a frozen final specification.

## Compatibility note

AIP remains the v1.x compatibility layer. Existing `aip_version` fields should not be removed yet.
