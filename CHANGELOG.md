# Changelog

All notable changes to the RunPack specification are documented here.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).
Versioning follows the MAJOR.MINOR scheme defined in §5 of the spec.

---

## [1.1] — RunPack public draft release

### Added
- Public rename from AIP / AI Prompt Protocol to RunPack.
- § 4 Error handling and retry protocol
  - Six error codes: E01–E06
  - Validation checklist
  - Retry packet format with `retry_for_error`, `original_task_id`, `attempt`, and `correction` fields
  - Retry policy including maximum attempt limits and E04 refusal handling
- § 5 Versioning discipline and governance
  - MAJOR.MINOR version scheme
  - Backwards compatibility rules for parsers
  - 12-month maintenance window for previous MAJOR versions
  - v1.x governance model: steward, proposal process, ratification criteria
- § 6 Context persistence and stateful tasks
  - Session envelope with `session_id` and `turn` counter
  - Rolling history block
  - `session_summarise` task type added to registry
- § 7 Trust model and safety
  - Sanitisation rules for untrusted strings
  - Role-boundary pattern with `[USER_INPUT]` delimiters
  - Explicit out-of-scope items: authentication, data residency, model-side filtering
- Task type registry expanded
  - Added: `document_summary`, `rewrite`, `translate`, `classify`, `logic_reasoning`, `session_summarise`, `claim_pressure_analysis`, `system_generation`, `framework_expansion`
- Appendix A with complete worked examples

### Changed
- `version` / `runpack_version` field clarified: must be a `MAJOR.MINOR` string, not an integer.
- Compatibility fields retained for v1.x: `legacy_aip_version` and `aip_version`.

---

## [1.0] — Foundation release

### Added
- § 1 Motivation and architecture — BYO-AI model, design principles, stack comparison
- § 2 Packet format — block format and JSON mode for task and result packets; required fields; status values (`ok`, `partial`, `refused`, `error`)
- § 3 Standard task type registry — initial 6 types: `npc_dialogue`, `story_generation`, `lyrics`, `image_description`, `painting_ideas`, `learning_activity`
- Custom type namespacing via reverse-domain convention
- `task_id` field as recommended for correlation
- `output_format` values: `text`, `json`, `markdown`
