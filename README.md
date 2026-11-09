# mojo-semver

<!-- badges -->
[![ci](https://github.com/amarbaro/mojo-semver/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/amarbaro/mojo-semver/actions/workflows/ci.yml) [![latest release](https://img.shields.io/github/v/release/amarbaro/mojo-semver)](https://github.com/amarbaro/mojo-semver/releases/latest) [![license](https://img.shields.io/github/license/amarbaro/mojo-semver)](https://github.com/amarbaro/mojo-semver/blob/main/LICENSE) [![checks](https://img.shields.io/endpoint?url=https://raw.githubusercontent.com/amarbaro/mojo-semver/badges/checks.json)](https://github.com/amarbaro/mojo-semver/actions/workflows/ci.yml)
<!-- /badges -->

Semantic Versioning 2.0.0 for Mojo, with the exact rules of Python [semver](https://pypi.org/project/semver/) 3.0: strict parse, precedence, `match`, the `bump_*` family. No regex engine and no dependencies; 212 lines of code (`mojo-loc`).

Install: `pixi workspace channel add https://conda.modular.com/max https://prefix.dev/amarbaro/amarbaro`, then `pixi add mojo-semver` (import `semver`). From source: `git clone https://github.com/amarbaro/mojo-semver`, then `mojo run -I mojo-semver/src app.mojo`.
Toolchains: Mojo 1.2 stable, and the nightly 1.3.0.dev2026110805 in the weekly [mojo-libs](https://github.com/amarbaro/mojo-libs) matrix (36 libraries on stable, 35 on nightly).

```mojo
from semver import parse


def main() raises:
    var v = parse("1.2.3-rc.1+build.5")
    print(v < parse("1.2.3"))              # True: a prerelease sorts first
    print(v.match(">=1.2.3-rc.0"))         # True
    print(v.bump_prerelease())             # 1.2.3-rc.2
    print(v.bump_minor())                  # 1.3.0
```

| Call | Does |
|---|---|
| `parse(s) -> Version` | Strict `major.minor.patch[-pre][+build]`; raises `"<s> is not valid SemVer string"`. |
| `is_valid(s) -> Bool` | `parse` without raising. |
| `compare(a, b) -> Int` | Parse both, then `-1`, `0` or `1`. |
| `Version(major, minor, patch, prerelease, build)` | Fields; an empty `prerelease`/`build` means absent. This constructor does not validate (negative numbers, bad identifiers pass): build versions from text with `parse`. |
| `v.compare(o)`, `==`, `!=`, `<`, `<=`, `>`, `>=` | SemVer precedence; build metadata ignored. |
| `v.match(expr)` | `<`, `>`, `<=`, `>=`, `==`, `!=` then a version; a bare version means `==`; anything else raises. |
| `v.bump_major()`, `bump_minor()`, `bump_patch()` | Next version, prerelease and build dropped. |
| `v.bump_prerelease(token="rc")` | Increment the last number in the prerelease, or start `<token>.1` (`token=""` gives `1`). Build dropped. |

**Checked against Python semver 3.0.4:** 474 parse results (the SemVer spec's valid and invalid examples plus 400 generated versions), a 60 x 60 precedence matrix plus 880 more comparisons, 15 match expressions per version, every bump variant, edge cases (trailing newline, 20-digit and leading-zero bumps) and the `match` error texts, all byte-identical (`scripts/check.sh`, each scenario twice). 6 of 6 mutants caught. Builds and passes on Mojo 1.2.0 and the 1.3 nightly.

**Speed** (7800X3D): parse 42 ns, compare 9.4 ns; Python semver 1,579 ns and 549 ns (`tests/bench.mojo`, `tools/bench_oracle.py`).

**Limits:** major, minor and patch must fit in 18 digits (Python accepts any size). Digits are ASCII only: Python's regex also accepts Unicode decimal digits (`1١.0.0` parses there), this parser refuses them. One trailing newline is accepted, as in Python. Caret and tilde ranges are not offered, because Python semver has none to check them against.

- API: [docs/api.md](docs/api.md) · design: [docs/design.md](docs/design.md) · agent skill: [skills/mojo-semver/SKILL.md](skills/mojo-semver/SKILL.md)
- License: Apache-2.0 (portions ported from python-semver, BSD-3-Clause: see NOTICE and LICENSE-semver)
