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

Hive Legacy is built as two separate, decoupled tools sharing one archive format:

**Generate** — the core tool. Enter a Hive username, preview the account (post/comment count, creation date), then pull the account's full post and comment history via public Hive API nodes into a portable JSON archive.

**Enrich** *(planned)* — a second, optional pass that takes an existing Hive Legacy archive and adds deeper detail — starting with `trx_id` and block number per entry — by walking the account's full on-chain history and matching it back to the archive by `author`/`permlink`. Enrichment is additive only; it never changes what Generate already captured, only adds to it. See `enrichment` in `SCHEMA.md` for how this is recorded.

The split exists so the fast, lightweight default path (Generate) never has to carry the cost of the slower, heavier one (Enrich) — most people only need the former.

## Archive format

The archive is JSON. Full field-by-field documentation, including *why* each field exists, lives in [`SCHEMA.md`](./SCHEMA.md). A machine-readable [`hive-legacy.schema.json`](./hive-legacy.schema.json) is also provided for automated validation.

Current scope is **posts and comments only** — the content a user actually authored. Wallet/transaction history is intentionally out of scope for Generate, since most Hive frontends already provide transaction history export through their own wallet views. `trx_id`/block number are deferred to the optional Enrich pass rather than the default flow, to keep Generate fast and light on public nodes.

## Node etiquette

Hive Legacy talks to public, largely volunteer-run Hive API nodes. Pulling a large account's full history means hundreds of sequential requests in a short window — not something to do carelessly.

- **Considerate mode** (a short delay between requests) is offered as a toggle, and the tool will suggest it for large accounts.
- **Fast mode** is available for anyone who wants it — the choice is always the user's, this is a suggestion, not a restriction.
- If you operate a Hive node and have thoughts on request patterns that would or wouldn't concern you, feedback is genuinely welcome — this behavior was designed carefully but without direct node-operator input, and may be adjusted based on real feedback.

## FAQ

**Why isn't there a delay by default?**
There is — Considerate mode is the default. See "Node etiquette" above.

**Why doesn't the archive include post bodies?**
By design. Hive Legacy preserves pointers and metadata, not content — see Philosophy above.

**Why are some titles truncated?**
The Hive API this tool queries returns titles pre-truncated for long posts. The permlink is the reliable identifier regardless. See `manifest.titles` in `SCHEMA.md`.

**Can I archive someone else's account, or only my own?**
Yes — this only reads public blockchain data, the same as any Hive frontend. No login or account ownership is required to generate an archive for any public username.

## Roadmap

- [ ] Enrich tool (see "Generate vs. Enrich" above)
- [ ] HTML viewer export (offline, human-friendly view of an archive)
- [ ] Markdown / CSV / PDF export
- [ ] ZIP packaging
- [ ] Optional "fetch full titles" mode (off by default) — backfills untruncated titles via a per-post lookup; warns the user this means significantly more API calls and run time
- [ ] Statistics and timeline generation
- [ ] A **SoloHive Legacy theme** — reads a Hive Legacy archive and renders an offline, human-friendly version of a user's Hive history

## Relationship to SoloHive

Hive Legacy preserves the map to the content. SoloHive displays the content. They're built to remain independent, but a future SoloHive theme could read a Hive Legacy archive directly.

## Notes on running against large accounts

Tested against accounts ranging from ~100 entries to 13,000+ entries (spanning pre-Hive-fork Steem history through present day) with no node throttling observed at any tested size. Uses a fallback list of public API nodes with automatic retry if one fails to respond.

## License

MIT (or update to match your other `dotw-hive` repos)
