# tmp-secure-patch

![CI](https://github.com/WNF12/tmp-secure-patch/actions/workflows/audit.yml/badge.svg)

This project enforces the use of `tmp@0.2.7` across all nested dependencies using npm's
`overrides` field, so that no transitive copy of `tmp` resolves to a vulnerable version.

## 🔐 Why

Two advisories apply. The second supersedes the first, and is the reason this project
no longer pins to `0.2.4`.

| Advisory | Severity | Affected | Fixed in |
|---|---|---|---|
| [GHSA-52f5-9888-hmc6](https://github.com/advisories/GHSA-52f5-9888-hmc6) | Low | `tmp <= 0.2.3` | `0.2.4` |
| [GHSA-ph9p-34f9-6g65](https://github.com/advisories/GHSA-ph9p-34f9-6g65) | High | `tmp < 0.2.6` | `0.2.6` |

- **GHSA-52f5-9888-hmc6** — arbitrary temporary file/directory write via the symbolic
  link `dir` parameter. This is the issue the project was originally created to address.
- **GHSA-ph9p-34f9-6g65** — path traversal via unsanitized `prefix`/`postfix` enabling
  directory escape. Published 2026-05-27. Because it affects everything below `0.2.6`,
  the original `0.2.4` pin became vulnerable in its own right.

Pinning to `0.2.7` (current latest) clears both.

> **Note:** a pin is a claim with an expiry date. Pinning to an exact version to fix one
> advisory means the project stops receiving fixes for the next one. Do not add `tmp` to
> the Dependabot `ignore` list — that combination is what allowed the `0.2.4` pin to go
> stale unnoticed.

## 📦 How

The `overrides` field in `package.json` forces a single resolved copy of `tmp`
throughout the tree, regardless of what intermediate packages request:

```json
"overrides": {
  "tmp": "0.2.7"
}
```

Consumers in this tree (`patch-package`, `solc`) declare `tmp@^0.0.33`; the override
replaces those with `0.2.7` and deduplicates them to one installed copy.

## ✅ What CI checks

The [audit workflow](.github/workflows/audit.yml) runs on every push and pull request
against `main`:

1. A pin check that parses `npm ls tmp --all --json` and requires every resolved copy
   of `tmp` to match the expected version.
2. `npm audit` over the whole tree — fails the build on any advisory.

The pin check **fails closed**: if `tmp` is absent from the tree, or `npm ls` errors,
the step fails rather than reporting success it cannot substantiate. It runs *before*
the audit so that an unrelated advisory cannot mask the one guarantee this project
exists to make.

## 📁 On the dependency list

`dependencies` is intentionally empty. This repository contains no application code —
it exists to demonstrate and enforce the `tmp` override — so it requires nothing
directly. The two tools it does use, `hardhat` and `patch-package`, are declared as
`devDependencies` and bring their own transitive trees.

`tmp` reaches the tree through `patch-package`, which is exactly the situation
`overrides` is designed for: a transitive dependency you do not control, forced to a
version you choose.
