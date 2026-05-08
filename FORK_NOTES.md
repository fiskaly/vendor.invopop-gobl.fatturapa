# Fork notes

fiskaly's fork of [`invopop/gobl.fatturapa`](https://github.com/invopop/gobl.fatturapa).

## Branch layout

| Branch | State |
|---|---|
| `main` | Mirror of upstream `main`. No fiskaly changes. |
| `master` | Fiskaly default branch. Upstream + our patches. **Consumers pin tag `v0.61.1` from this branch.** |

## Divergence

| Tag | Base | Patch |
|---|---|---|
| `v0.61.1` | upstream `v0.61.0` (`0d5be0d`) | `ebb0308` — emit `<SocioUnico>`, make `<StatoLiquidazione>` configurable |

`parties.go`:
- Added `SoleShareholder string xml:"SocioUnico,omitempty"` to the emitter `Registration` struct (placed between `Capital` and `LiquidationState`, matching FatturaPA element order 1.2.4.4 → 1.2.4.5).
- `newRegistration()` now reads `LiquidationState` from input (defaults to `"LN"` only when empty — preserves prior behaviour) and `SoleShareholder` as pass-through.

`parties_test.go`: 4-case table-driven test covering the `LS`/`LN` × `SU`/`SM` IscrizioneREA variations, plus an explicit assertion that `<SocioUnico>` is omitted when unset.

## Why

The two new fields complete the FatturaPA `IscrizioneREA` block for capital companies:
- `<StatoLiquidazione>` was hardcoded to `"LN"` upstream — companies in liquidation got structurally incorrect XML.
- `<SocioUnico>` was never emitted upstream — sole-shareholder SRLs lost their Civil Code Art. 2470 disclosure.

## Dependency on the gobl fork

`go.mod` pins `github.com/invopop/gobl => github.com/fiskaly/vendor.invopop-gobl v0.306.1` via `replace`. That fiskaly tag is upstream `gobl v0.306.0` plus the two-field extension on `org.Registration`. The new fields here read from there.

When the gobl fork bumps its pinned tag, the `replace` line in this fork's `go.mod` must bump too.

## Forward-port

When upstream `gobl.fatturapa` cuts a new tag (e.g. `v0.62.0`):

1. `git fetch upstream --tags`
2. `git checkout master && git rebase v0.62.0` (or branch from the new tag and cherry-pick `ebb0308`).
3. Resolve conflicts in `parties.go` if upstream changed `Registration` / `newRegistration()`.
4. `go test ./...` (needs libxml2 — see *Local test env* below).
5. Push + tag `v0.62.1`. Update consumers' `go.mod` replace.

If/when the upstream PR is accepted, drop this fork.

## Upstream PR

TBD. Source: `master` (commit `ebb0308`).

## Local test env

Tests use `lestrrat-go/libxml2` via cgo. The pinned version is incompatible with libxml2 ≥ 2.15. To run tests locally, install an older libxml2 (e.g. 2.13 or 2.14) and set `PKG_CONFIG_PATH` to point at it. CI uses a controlled libxml2 version.
