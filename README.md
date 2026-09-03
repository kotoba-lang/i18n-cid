# i18n-cid

**Content-addressed i18n catalogs for kotoba-lang — the two seams between
[`kotoba-lang/i18n`](https://github.com/kotoba-lang/i18n) catalogs and the
IPLD / kotobase content plane.**

The Unison-like property of kotoba (a definition is identified by its content,
names are mutable references) applied to translation: a catalog's identity is
its CID, identical translations share blocks byte-for-byte, and a one-string
edit costs exactly one new block + a parent-link update.

## Piece 1 — catalog blocks (`i18n-cid.core`)

- `catalog->block` — an `i18n.core/register!`-shape catalog becomes a
  canonical DAG-CBOR block (`{:cid :bytes :node}`). Keyword keys are
  stringified (`:app/title` -> `"app/title"`), nested select-maps are walked,
  keyword values are encoded losslessly as `{"$kw" ".."}`.
- `locale-index->block` — a project index block: `{"format" "i18n-catalog/1"
  "project" .. "locales" {"ja" "<cid>" ...}}`.
- `dag->catalog` / `index->locales` — round-trip, restoring keywords.
- `lake-claim` / `lake-claim-index` — admit a block into the
  [`kotobase-lake`](https://github.com/kotoba-lang/kotobase-lake) catalog
  plane, so Datalog / SQL / SPARQL / Cypher over the kotobase datom plane can
  ask what catalogs exist, for which project/locale, without a second index.

Honesty boundary: this library encodes and addresses catalog bytes. It does
not pin them (IPFS), announce them (IPNI), or authorize writes (Biscuit).

## Piece 2 — the seam (`i18n-cid.seam`)

- `register-from-block!` / `register-index!` — runtime path: fetch catalogs
  from any block store (kotobase as-of query, IPFS gateway, local disk) and
  merge into `i18n.core`. Same merge semantics as `register!` (later wins), so
  static resources and CID catalogs coexist.
- `write-catalog-resource!` — build-time path: materialize a catalog block as
  `i18n/messages/<locale>.edn` so `i18n.messages/defmessages` reads it at
  macroexpansion. JVM-only. Commit the artifact; compile-time key checking is
  unchanged.

## Test

```sh
clojure -M:test   # 7 tests / 17 assertions, green
```
