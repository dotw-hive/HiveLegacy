# Hive Legacy

A single-file, no-build-step tool for preserving the *map* to a Hive account's history — not the content itself, but the identifiers needed to locate, verify, and rediscover it later.

## Philosophy

Hive Legacy preserves index cards, not the library. The Hive blockchain is already the permanent source of truth for posts, comments, and every other public account action — Hive Legacy doesn't archive or replace that. It generates a portable, human- and machine-readable index of a user's Hive journey: their own posts, their own comments, and the pointers needed to find both again.

The goal is to help preserve:

- A user's own Hive journey
- The work of past Hive users
- Accounts belonging to people who are no longer active, or who have passed away
- Important community history

**What it is not:**

- **Not a content archive.** No post bodies, no images, no full comment text — enough to relocate and verify content on-chain, not a copy of it.
- **Not tied to any frontend.** No PeakD/Ecency/Hive.blog URLs anywhere in the output — only blockchain-level identifiers, so the archive stays useful even if every current frontend disappears tomorrow.
- **Not a replacement for the blockchain.** The chain remains the permanent record. Hive Legacy is the catalog card, not the vault.

## Quick Start

Single file, no dependencies, no build step. Either:

- Open `index.html` directly in a browser, or
- Serve it locally to avoid `file://` quirks around blob downloads in some browsers:
  ```
  python3 -m http.server 8000
  ```
  then visit `http://localhost:8000/index.html`

It also runs as a drop-in static file on any host, same as HiveWrite and HiveDrop. No login, no Keychain, no write access to the chain — this only ever reads public data.

## Generate vs. Enrich

Hive Legacy is built as two separate, decoupled tools, presented as tabs, sharing one archive format:

**Generate** — the core tool. Enter a Hive username, preview the account (post/comment count, creation date), then pull the account's full post and comment history via public Hive API nodes into a portable JSON archive.

**Enrich** — a second, optional pass. Upload an existing Hive Legacy archive and Enrich walks the account's full on-chain operation history, matching it back to the archive by `author`/`permlink`, to add:

- `trx_id` and block number per entry — lets any entry be located and verified directly on-chain
- The full, untruncated original post title (upgrading Generate's possibly-truncated version — see `manifest.titles` below)
- `latest_title`, added only when a post's title was edited after publication — the original `title` field always reflects what was first published, `latest_title` reflects what the author ultimately left on the blockchain

Enrichment is additive only; it never changes or overwrites what Generate already captured, only adds to it. See `enrichment` in [`SCHEMA.md`](./SCHEMA.md) for how this is recorded, and the `titles`/`latest_title` entries there for the full reasoning behind the title-handling approach.

Enrich validates any uploaded file against the full archive format before running — malformed JSON, archives from very old versions of this tool, or hand-edited/corrupted files are rejected with a clear message rather than silently producing a broken result.

The Generate/Enrich split exists so the fast, lightweight default path (Generate) never has to carry the cost of the slower, heavier one (Enrich) — most people only need the former.

## Archive format

The archive is JSON. Full field-by-field documentation, including *why* each field exists, lives in [`SCHEMA.md`](./SCHEMA.md). A machine-readable [`hive-legacy.schema.json`](./hive-legacy.schema.json) is also provided for automated validation.

Current scope for Generate is **posts and comments only** — the content a user actually authored. Wallet/transaction history is intentionally out of scope, since most Hive frontends already provide transaction history export through their own wallet views. `trx_id`/block number/full titles are deferred to the optional Enrich pass rather than the default flow, to keep Generate fast and light on public nodes.

## Node etiquette

Hive Legacy talks to public, largely volunteer-run Hive API nodes. Pulling a large account's full history means hundreds (or thousands) of sequential requests in a short window — not something to do carelessly.

- **Fast** and **Considerate** (a short delay between requests) are both available as a toggle on both Generate and Enrich. The choice is always the user's — this is a suggestion, not a restriction.
- **Generate defaults to Fast.** For accounts with 5,000+ combined posts/comments, a notice explains the tradeoff and offers a one-click switch to Considerate, but nothing switches silently — Fast stays selected unless you act.
- **Enrich defaults to Considerate.** Enrich walks an account's full on-chain operation history (not just posts/comments), which is a heavier request pattern than Generate's — the more cautious default reflects that.
- Every request has a timeout, and both Generate and Enrich have a Cancel button — if a node hangs or you change your mind mid-run, you're never stuck waiting it out or needing to refresh the page.
- If you operate a Hive node and have thoughts on request patterns that would or wouldn't concern you, feedback is genuinely welcome — this behavior was designed carefully but without direct node-operator input, and may be adjusted based on real feedback.

## FAQ

**Why isn't there a delay by default?**
Generate defaults to Fast; Enrich defaults to Considerate. See "Node etiquette" above for why they differ.

**Why doesn't the archive include post bodies?**
By design. Hive Legacy preserves pointers and metadata, not content — see Philosophy above.

**Why are some titles truncated?**
Generate's title comes from an API that pre-truncates long titles. The permlink is the reliable identifier regardless. Running Enrich backfills the full original title — see `manifest.titles` in `SCHEMA.md`.

**Why does an entry have both `title` and `latest_title`?**
It means the post was edited after publication. `title` is always what was originally published; `latest_title` is what it was most recently changed to. Only appears after Enrich, and only when the two actually differ.

**I uploaded an archive to Enrich and got a "malformed or invalid" error — what happened?**
Enrich only accepts complete, well-formed Hive Legacy archives. If the file is corrupted, hand-edited, or from a very old version of this tool, it's rejected rather than silently producing a broken result. If you know the account name, generating a fresh archive is the fastest fix.

**Can I archive someone else's account, or only my own?**
Yes — this only reads public blockchain data, the same as any Hive frontend. No login or account ownership is required to generate an archive for any public username.

## Roadmap

- [ ] HTML viewer export (offline, human-friendly view of an archive)
- [ ] Markdown / CSV / PDF export
- [ ] ZIP packaging
- [ ] Statistics and timeline generation
- [ ] A **SoloHive Legacy theme** — reads a Hive Legacy archive and renders an offline, human-friendly version of a user's Hive history

## Relationship to SoloHive

Hive Legacy preserves the map to the content. SoloHive displays the content. They're built to remain independent, but a future SoloHive theme could read a Hive Legacy archive directly.

## Notes on running against large accounts

**Generate** has been tested against accounts ranging from ~100 entries to 13,000+ entries (spanning pre-Hive-fork Steem history through present day) with no node throttling observed at any tested size.

**Enrich** has been tested successfully but not yet stress-tested at that scale — it walks a heavier endpoint (full account operation history, not just posts/comments), so very large accounts may behave differently. Considerate mode's default-on behavior for Enrich is a deliberate precaution given this hasn't been fully proven out yet.

Both tools use a fallback list of public API nodes with automatic retry if one fails or times out.

## License

MIT (or update to match your other `dotw-hive` repos)
