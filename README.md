# Hive Legacy

A single-file, no-build-step tool for preserving the *map* to a Hive account's history — not the content itself, but the identifiers needed to locate, verify, and rediscover it later.

## Mission

The Hive blockchain is already the permanent source of truth for posts, comments, and every other public account action. Hive Legacy doesn't archive or replace that — it generates a portable, human- and machine-readable index of a user's Hive journey: their own posts, their own comments, and the pointers needed to find both again.

The goal is to help preserve:

- A user's own Hive journey
- The work of past Hive users
- Accounts belonging to people who are no longer active, or who have passed away
- Important community history

## What it is not

- **Not a content archive.** It does not store post bodies, images, or full comment text. It stores `author` + `permlink` + `created` + light metadata — enough to relocate and verify content on-chain, not a copy of the content itself.
- **Not tied to any frontend.** No PeakD/Ecency/Hive.blog URLs anywhere in the output — only blockchain-level identifiers, so the archive stays useful even if every current frontend disappears tomorrow.
- **Not a replacement for the blockchain.** The chain remains the permanent record. Hive Legacy is the catalog card, not the vault.

## How it works

1. Enter a Hive username.
2. Look up the account (post/comment count, creation date, reputation) as a quick preview before committing to a full run.
3. Generate the archive — paginates through the account's full post and comment history via public Hive API nodes.
4. Download a single JSON file.

No login, no Keychain, no write access to the chain — this only ever reads public data.

## Running it

Single file, no dependencies, no build step. Either:

- Open `index.html` directly in a browser, or
- Serve it locally to avoid `file://` quirks around blob downloads in some browsers:
  ```
  python3 -m http.server 8000
  ```
  then visit `http://localhost:8000/index.html`

It also runs as a drop-in static file on any host, same as HiveWrite and HiveDrop.

## Archive format (schema v0.1)

JSON is the primary, foundational format. Other export formats (HTML viewer, Markdown, CSV, PDF, ZIP) are planned but not yet implemented — see Roadmap.

```json
{
  "schema": "hive-legacy-archive",
  "version": "0.1",
  "generated_at": "2026-07-30T19:43:17.474Z",
  "generator": "Hive Legacy MVP",
  "account": {
    "name": "username"
  },
  "summary": {
    "post_count": 97,
    "comment_count": 21,
    "total_entries": 118
  },
  "entries": [
    {
      "type": "post",
      "author": "username",
      "permlink": "post-permlink",
      "created": "2021-05-01T00:00:00",
      "category": "hive-139531",
      "title": "Post title",
      "tags": ["hive-139531"]
    },
    {
      "type": "comment",
      "author": "username",
      "permlink": "comment-permlink",
      "created": "2021-05-01T00:05:00",
      "category": "hive-139531",
      "parent_author": "someone",
      "parent_permlink": "their-post-permlink"
    }
  ]
}
```

`entries` is sorted chronologically. `type` is either `post` or `comment`, distinguished by whether `parent_author` is populated on-chain.

### Design principles behind the schema

- **Blockchain-level identifiers only.** `author` + `permlink` (+ `created` as a tiebreaker/timestamp) is enough for any future tool to relocate or verify the original content — no frontend URLs.
- **Versioned from day one.** `schema` and `version` fields exist so the format can evolve without breaking archives already in the wild, and so it can potentially grow into a shared spec other Hive tools could support.
- **Flat and typed.** Entries aren't nested by category or op-type, so a downstream reader can filter by `type` without needing to understand every possible shape up front.

## Current scope (MVP)

Focused specifically on **posts and comments** — the content a user actually authored. Wallet/transaction history is intentionally out of scope, since most Hive frontends already provide transaction history export through their own wallet views.

Not included in v0.1 (by design, for now):
- Transfers, votes, and other wallet-level operations
- `trx_id` / block number per entry
- `custom_json` operations
- Community/profile-level metadata beyond what's shown in the lookup preview

## Roadmap

- [ ] HTML viewer export (offline, human-friendly view of an archive)
- [ ] Markdown / CSV / PDF export
- [ ] ZIP packaging
- [ ] Account profile metadata in the archive itself (not just the lookup preview)
- [ ] Optional `trx_id` / block backfill per entry
- [ ] Statistics and timeline generation
- [ ] A **SoloHive Legacy theme** — reads a Hive Legacy archive and renders an offline, human-friendly version of a user's Hive history
- [ ] A documented archive specification other Hive applications could adopt

## Relationship to SoloHive

Hive Legacy preserves the map to the content. SoloHive displays the content. They're built to remain independent, but a future SoloHive theme could read a Hive Legacy archive directly.

## Notes on running against large accounts

Tested against accounts ranging from ~100 entries to 3,000+ entries (spanning pre-Hive-fork Steem history through present day) with no node throttling observed. Uses a fallback list of public API nodes with automatic retry if one fails to respond.

## License

MIT (or update to match your other `dotw-hive` repos)
