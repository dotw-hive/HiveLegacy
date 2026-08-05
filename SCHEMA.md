# Hive Legacy Archive Schema — Field Intent

This document explains **why** each field in a Hive Legacy archive exists, not just its type or shape. For structural validation, see [`hive-legacy.schema.json`](./hive-legacy.schema.json). For a general introduction to the project, see [`README.md`](./README.md).

This document is written for anyone — human or AI — trying to understand *why the archive is shaped the way it is*, especially someone encountering it years from now with no other context.

---

## Top-level fields

### `schema`

Purpose: Identifies the file as a Hive Legacy archive, distinct from any other JSON a future reader might encounter. A fixed string, always `"hive-legacy-archive"`.

### `version`

Purpose: Records which version of the archive format this file follows. Versioning exists from the start so the format can grow — new fields, richer coverage — without breaking archives already in the wild. A `version` of `"0.1"` and one of `"0.2"` are both valid, permanent, honest records of what that archive contained at the time it was made; neither is retroactively "wrong."

### `generated_at`

Purpose: The timestamp this specific archive was produced. This is permanent and never changes, even if the archive is later enriched. If someone generates multiple archives of the same account over time, `generated_at` is what distinguishes them and tells the story of *when* each snapshot was taken.

### `generator`

Purpose: Names the tool that produced this archive (e.g. `"Hive Legacy MVP"`). Provenance — a future reader should never have to guess what created a file.

### `enrichment`

Purpose: Records whether, when, and by what a second, additive pass was run against this archive.

Values:
- `null` — this archive has not been enriched. Explicitly `null`, not omitted, because an omitted field is ambiguous (did the generator not know about enrichment, or was it deliberately skipped?), while `null` is a clear, literal statement: this was checked, and it hasn't happened.
- An object — enrichment has occurred. Contains its own `generator`, `version`, and `enriched_at`, mirroring the top-level provenance fields, so the enrichment event is documented with the same completeness as the original generation.

Naming note: called `enrichment`, not `amendment`. "Amend" implies correcting or altering existing data. Enrichment only adds new fields (like `trx_id`/block number) to entries that already exist — it never changes what Generate originally captured. The name should not imply the archive's original content is being revised.

### `account`

Purpose: Identifies which Hive account this archive belongs to, and anchors it in time.

Fields:
- `name` — the account's username.
- `created` — the account's on-chain creation date. Included deliberately to give a future archivist or researcher "a definite start position" for the account's history — a fixed point the whole record can be measured against.

**Deliberately excluded:** profile bio, "about" text, or other mutable profile fields. `created` is a permanent, immutable, on-chain fact. Bio/about text is mutable — it changes whenever the user edits their profile, and a snapshot of it would represent *current state at generation time*, not a fixed historical fact. Including it would blur the line between "preserving pointers to what happened" (the project's actual purpose) and "occasionally capturing incidental present-state data" (out of scope). If this is ever reconsidered, it should be added deliberately and separately, not folded quietly into `account`.

### `manifest`

Purpose: Describes what information is intentionally present, partially present, or absent in **this specific archive file** — nothing more, nothing less. A future tool or reader should consult `manifest` before assuming any given kind of data exists, rather than inferring it from the entries alone.

Values, per field: `"full"`, `"partial"`, or `"none"`.

Why a three-state value instead of true/false: some data isn't a clean yes/no. Titles, for example, are neither fully present nor fully absent — they're structurally truncated by the API this tool queries. A boolean can't express that; a tri-state can.

Critical design rule: **the manifest is strictly literal to this file, never aspirational.** It never describes the project's roadmap or future intentions — only what is actually, currently true of the archive it's attached to. Roadmap and future plans belong in `README.md`, read by humans, not embedded in a data file meant to be trusted literally by machines. Two archives of the same account, generated months apart with different tool capabilities, will correctly show different manifests — each one honestly describing its own contents.

Why some fields are explicitly listed as `"none"` rather than omitted entirely: this project deliberately excludes categories of on-chain data that exist on the Hive blockchain but fall outside its purpose (see individual field notes below). A future person cross-referencing this archive against the actual blockchain or a full mirror might reasonably expect to find, say, vote data or reward data. Explicitly stating `"votes": "none"` tells them definitively: *this was never captured, by design — not a bug, not an oversight, not something specific to this account.* This closes the gap between what the archive contains and what the full chain contains, so absence doesn't get misread as an error.

Manifest fields:

- **`account_metadata`** — Purpose: coverage of the `account` block itself (name, created).
- **`posts`** / **`comments`** — Purpose: coverage of the core content pointers that make up `entries`.
- **`titles`** — Meaning: `"partial"` indicates title text may be truncated due to limitations of the Hive API used during generation. The permlink remains the reliable identifier regardless of title completeness.
- **`last_update`** — Purpose: whether the archive records if the original author edited a post after publication. This is preserved as part of the author's own publishing history — it reflects the author's own action, not an external reaction to their content. Currently `"none"`; planned for future capture.
- **`children`** — Purpose: whether the archive records the number of replies a post had at generation time. This is preserved as evidence that discussion occurred, not as a social or popularity metric — it tells a future reader "this generated conversation," not "this was well-received." Currently `"none"`; planned for future capture.
- **`bodies`** — Full post/comment text. Permanently `"none"` by design — see Philosophy in `README.md`. This is a pointer archive, not a content mirror.
- **`transactions`** / **`blocks`** — `trx_id` and block number. `"none"` in a base (Generate-only) archive; becomes `"full"` after an Enrich pass. Deferred from the default flow because obtaining them requires walking an account's full on-chain operation history, a heavier and slower process than the default archive generation.
- **`rewards`** — Payout values, curation rewards, beneficiaries, cashout timing. Permanently `"none"` by design. This is economic data — how the market responded to the content — not a record of what the author did or authored.
- **`votes`** — Vote records, vote weight, curation data. Permanently `"none"` by design. This is *other accounts'* actions directed at the content, not the author's own record.
- **`reblogs`** — Who resteemed/reblogged the content. Permanently `"none"` by design, for the same reason as `votes`: it's someone else's action, not the author's.

The guiding question behind every inclusion/exclusion decision in the manifest: **does this field describe something the account owner actually did or authored, or does it describe how others reacted to it?** The former is in scope; the latter is deliberately not.

### `summary`

Purpose: A quick, human-scannable count of what's in `entries` — `post_count`, `comment_count`, `total_entries` — without requiring a reader to parse the full array first.

### `entries`

Purpose: The actual index. An array of posts and comments, sorted chronologically, each entry carrying the identifiers needed to locate and verify the original content on-chain (`author`, `permlink`, `created`), plus light descriptive metadata (`category`, `title`, `tags` for posts; `parent_author`, `parent_permlink` for comments).

Entries are flat and typed (`type: "post"` or `type: "comment"`) rather than nested by category, so a downstream reader can filter by type without needing to understand every possible shape up front.

---

## Schema Evolution

Hive Legacy intentionally uses two different validation strategies, depending on what a given object is for:

- **Archive metadata objects** (`account`, `manifest`, `summary`, `enrichment`) are **closed** — an archive with an unexpected field in any of these is invalid. These objects define the structure and capabilities of the archive itself, and their value depends on being exhaustive. `manifest` in particular only works as a trust mechanism — a reader relying on it to know what's present, partial, or absent — if it's complete and can't silently grow an undocumented field.

- **Archive entry objects** (each item in `entries`) are **intentionally left open**. Entries are the part of the schema most likely to grow — `trx_id` and `block` already exist today anticipating the Enrich pass, and future versions of Hive Legacy may add further per-entry fields. Leaving entries open means a newer archive with additional fields still validates against an older schema, and an older validator doesn't need to be updated just to accept archives written by a newer, still-compatible version of the tool.

This is a deliberate split, not an oversight: closed where completeness is the point, open where growth is expected. If a future contributor notices this asymmetry, this section is the record that it was chosen on purpose, and why.

## A note on future changes

If a field's purpose or scope is ever reconsidered, update this document alongside the change — and explain *why* the change was made, the same way this document explains why the current fields exist. The goal is for a reader in any future year to understand not just what the archive contains, but the reasoning behind it.
