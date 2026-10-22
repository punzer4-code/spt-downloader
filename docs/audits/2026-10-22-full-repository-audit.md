# Full repository audit — `punzer4-code/spt-downloader`

| Field | Value |
| --- | --- |
| Audit date | 2026-10-22 |
| Repository audited | `punzer4-code/spt-downloader` (public fork) |
| Upstream | `cepeter-job/spt-downloader` |
| Commit audited | `44969fcd73534153f5b656ec5ee9f5c8f11f42f4` ("Merge pull request #72 …") |
| Fork vs upstream `main` | **Identical** (`ahead_by: 0, behind_by: 0`) at audit time |
| CI evidence for this commit | Upstream run [39728441622](https://github.com/cepeter-job/spt-downloader/actions/runs/39728441622) — **success**, 13m06s, all 9 jobs green |
| Auditor method | Static/source audit + GitHub API + CI evidence (see Limitations) |

## Executive summary

This codebase is, by a wide margin, more disciplined than typical internal
automation tools: hash-locked dependencies, SHA-pinned Actions, provenance
attestation, an anti-skip test-evidence system, exact-origin credential
isolation, and a read-only browsing discipline that is enforced in code rather
than merely documented. At the audited commit the full Windows test suite,
dependency audit, packaging, and attestation pipeline are green upstream.

**No critical or high-severity code vulnerabilities were found.** The material
risks are not in the application code; they are in **repository governance and
distribution, especially for this fork**:

1. The fork was created today (2026-10-22) and inherits **none** of upstream's
   protections: no branch ruleset, Actions not enabled (0 workflows listed),
   Dependabot alerts disabled, no CodeQL. `main` on the fork is currently
   unprotected.
2. **There is no durable release anywhere.** Zero tags, zero GitHub Releases
   (fork and upstream). The README tells users to download a CI artifact,
   which expires after **14 days**. The promotion workflow exists but has
   never produced a published release.
3. **231 references point at the upstream owner** (`cepeter-job`), including
   the README download link, the SECURITY.md private-reporting URL, and a
   source URL embedded in the shipped executable's README by
   `build_desktop.py`. A fork build would silently direct its users — and
   its vulnerability reports — to upstream.

Overall grades:

| Area | Grade | One-line basis |
| --- | --- | --- |
| Application security design | **A** | Origin-pinned credentials, request guard, no write APIs, formula-safe export, PDF custody checks |
| Supply chain / CI | **A** | Hash-locked installs, SHA-pinned actions, SBOM + attestation, pip-audit gate, least-privilege tokens |
| Test discipline | **A−** | 1,395 test functions, evidence manifests that make silent skips impossible; no coverage metric |
| Code maintainability | **B** | Compact style, low annotation (7%), a few very complex functions, no lint/type gate |
| Documentation | **A−** | 69 Markdown files, 0/376 broken internal links; fork identity not updated |
| Release & distribution | **C** | No release, no tag, unsigned EXE, 14-day artifact expiry as the only channel |
| Fork readiness (this repo) | **D** | Unprotected `main`, Actions off, alerts off, identity still upstream's |

---

## 1. Repository identity and GitHub posture

### Facts

- `punzer4-code/spt-downloader` is a **public fork** of
  `cepeter-job/spt-downloader`, created `2026-10-22T05:33:32Z`, 2,110 KB.
- Description: **empty**. Topics: **none**. Homepage: **none**. Stars/forks: 0.
- License detected: GPL-3.0 (matches `LICENSE`).
- Issues: **disabled** (both fork and upstream). Wiki/discussions: disabled.
  Projects: enabled. `delete_branch_on_merge`: false.
- Fork branches: only `main`. Upstream branches: `main`,
  `fix/devops-audit-20261022`, `feat/ui-delivery-completion` (open **draft**
  PR #71), `feat/tk-accessibility-candidate` (recent runs failing/in-progress
  — WIP, not in this snapshot).
- Fork Actions: API returns `total_count: 0` workflows → **Actions are not
  enabled/initialized on the fork**, so the three workflow files in the tree
  currently do nothing here.
- Fork rulesets: `[]` → **`main` has no protection on the fork**. Upstream
  has an active ruleset ("main melalui PR dan pemeriksaan wajib", id
  24914050): PR-only, no force-push/deletion, strict required checks
  `Windows tests and dependency audit` + `CodeQL`, thread resolution
  required, but `required_approving_review_count: 0` (no human review
  enforced — reasonable for a solo maintainer, worth knowing).
- `.github/rulesets/main.json` in the tree documents upstream's ruleset but
  does **not** activate anything by itself — a fork owner must import/apply
  it manually in Settings.
- Dependabot alerts on the fork: **disabled** (API 403 "Dependabot alerts are
  disabled for this repository"), even though `.github/dependabot.yml` is
  present and well-formed (weekly, grouped, pip + github-actions).
- CodeQL upstream runs via GitHub **default setup** (runs named `CodeQL`,
  `dynamic`) — there is no CodeQL workflow/config in the repository, so the
  fork has no code scanning and copying upstream's ruleset verbatim would
  require a `CodeQL` check context that will never report on the fork.
- Several admin APIs (branch protection, Actions permissions, code-scanning
  and secret-scanning alert lists) returned `403 Resource not accessible by
  integration` from this sandbox; per SECURITY.md's own guidance that proves
  nothing either way and must be checked in the GitHub UI by an admin.

### Assessment

Upstream's posture is strong; **none of it carries over automatically**.
Anyone treating this fork as "the same repo, same safety" would be wrong
until ruleset + Actions + alerts + scanning are deliberately enabled here.

## 2. History and development process

- **212 commits in 13 days** (2026-10-09 → 2026-10-22), all on the path to
  the current `main`. Peak days: 58 commits (Oct 13), 43 (Oct 21).
- Authors: one human maintainer under three identities
  (`cepeter97@gmail.com` ×117, noreply ×84) — effectively a **single-
  maintainer project (bus factor = 1)** — plus `ChatGPT Coding Agent` (×8)
  and `dependabot[bot]` (×3). AI-assisted development is visible and, to its
  credit, not disguised.
- Commit style: conventional prefixes (`test` 67, `fix` 49, `feat` 37,
  `docs` 21, merges 23). Test-first cadence is real, not cosmetic —
  *test* is the most common commit type.
- Upstream workflow is PR-based: PRs numbered up to #72 in 13 days,
  including explicitly-labelled throwaway evidence PRs ("do not merge")
  that were closed unmerged. The audited HEAD merge commit is
  GitHub-signed (`verified: true`).
- No tags in entire history. No `CHANGELOG.md`; release history must be
  reconstructed from PR titles.

## 3. Architecture and code structure

### Layout (353 tracked files, ~5.5 MB working tree)

| Area | Files | Lines (Python) |
| --- | ---: | ---: |
| CLI + capture/export engine (`coretax*.py`, root) | 24 | — |
| Desktop package (`spt_desktop/`) | 22 | — |
| CI/build scripts (`scripts/`, `build_desktop.py`, `spt_gui.py`) | 13 | — |
| **Production Python total** | **57** | **18,376** |
| Tests (`tests/`, incl. `support_*.py` helpers) | 202 | 31,855 |
| Documentation (`*.md`) | 69 | ~7,568 |

Test-to-production line ratio ≈ **1.7 : 1**.

### Boundaries (verified statically)

- Entry points: `coretax.py` (CLI + offline re-export), `spt_gui.py`
  (frozen GUI + `--self-test`), `spt_desktop/worker.py` (isolated worker
  process), `build_desktop.py` (PyInstaller).
- The AGENTS.md rule *"GUI must not import Playwright/openpyxl"* holds:
  `spt_desktop/controller.py` only mentions them in a comment;
  `worker.py` imports Playwright lazily inside `playwright_factory()`;
  Tkinter is confined to `app.py`/`ui_*`/`spt_gui.py`.
- `WORKER_FILES` (42 entries) is an explicit allowlist. Reachability
  analysis from `coretax.py` + `worker.py` found **no worker-required
  module missing from the allowlist and no unused entry**; GUI-only modules
  (`app/controller/runtime/ui_*`) are correctly excluded.
- Runtime bootstrap (`spt_desktop/runtime.py`) re-verifies interpreter
  identity (CPython, exact patch 3.14.8, 64-bit, GIL enabled, *managed*
  under the app workspace), rejects PATH `uv` for frozen builds, installs
  with `--require-hashes --only-binary :all:`, fingerprints worker sources,
  and refuses to leak raw subprocess stderr (proxy/index URLs can contain
  secrets) into errors.

### Maintainability findings

- **Complexity hotspots** (McCabe via Ruff C901 @15 → 53 functions exceed):
  `collect_inventory` **61** (299 lines, `coretax_browser.py:769`),
  `collect_details` 41, `adapt_corporate_detail` 38, `capture_detail` 34,
  plus `selection_dialog` (289 lines) and `overwrite_dialog` (193) in
  `spt_desktop/app.py`. The two largest modules are `coretax_browser.py`
  (1,579 lines) and `spt_desktop/app.py` (1,013). These are the files where
  regressions will be most expensive; they are also heavily test-covered,
  which partially offsets the risk.
- **Style**: deliberately compact (semicolon-joined statements, 237 lines
  >120 chars in production code). Consistent, but raises review cost for
  new contributors. There is **no formatter/linter/type-checker gate**
  anywhere (no `pyproject.toml`, `ruff.toml`, pre-commit, or CI step).
  A one-off Ruff pass (broad ruleset) reports 199 findings in production
  code — dominated by unsorted imports (65) and blind `except Exception`
  (52), much of it intentional defensive style — and 664 in tests, where
  164× F811 is the normal pytest fixture-shadowing pattern (noise).
- **Typing/docstrings**: 7% of 609 production functions carry annotations;
  18% have docstrings. `tuple[str, bytes] | None`-style modern syntax is
  used in newer modules (`credentials.py` is exemplary), older engine
  modules are largely unannotated.
- One genuine lint hit worth fixing: **F821/B023-style late binding** at
  `spt_gui.py:79` — `root.after(250, lambda: settled.set(True))` inside a
  loop that later `del settled`. It works today only because
  `wait_variable` blocks until the callback fires; it is fragile to
  reordering. (In self-test code, so impact is test reliability, not user
  data.)
- Dead weight is minimal: 0 TODO/FIXME in the tree, 4 unused imports.
- **Duplicate binaries**: the three template workbooks in `resources/`
  are byte-identical copies of `tests/fixtures/*.xlsx` (same MD5s,
  ~42 KB duplicated). Drift risk if one copy is ever edited without the
  other; a single source of truth plus a test that the fixture matches the
  shipped resource would be safer (or vice versa).
- Tests use `sys.path.insert` (13 files) instead of packaging/`conftest.py`
  — works, and CI enforces collection parity between Linux and Windows,
  but it's a smell that will bite if the project is ever packaged.
- There is **no `pyproject.toml`**: the project is not pip-installable;
  it runs as scripts from a checkout or as the frozen EXE. That's a
  legitimate choice for this tool, but it means no standard metadata,
  no `requires-python`, no console entry points.

## 4. Security audit

Scope note: this is a tool that handles taxpayer data and a government
intranet portal (`coretax.intranet.pajak.go.id`, SSO via
`satu.intranet.pajak.go.id`). It only functions with authorized intranet
access; nothing in the code attempts to bypass that.

### Strengths verified in code

- **No secrets in tree or history.** Full-history scan (212 commits, added
  lines) for token/key/password patterns: clean. All 15/16-digit strings
  in the tree are synthetic (`0000000000000001`), precision constants
  (2^53−1), or placeholders. `.gitignore` blocks `*.xlsx/.json/.pdf/.env/
  *.pem/*.key`, browser profiles, cookies, and output folders. The seven
  tracked `.xlsx` files are layout templates; I inspected their internals —
  shared strings contain only form labels, `corporate_format.xlsx` even
  carries `description: "Sanitized corporate layout; no taxpayer values"`.
- **Credentials** (`spt_desktop/credentials.py`): Windows Credential
  Manager only, targets pinned to an exact-origin allowlist, `Login.password`
  excluded from `repr`, blob size/parity validated, write buffer zeroed
  after `CredWriteW`. Preferences files store no password/token/cookie.
- **Auth request guard** (`spt_desktop/auth.py`): a Playwright route
  interceptor aborts any request outside approved origins and any
  cross-origin write; the one allowed code-exchange is consumed-once,
  deadline-bound, fetched with `max_redirects=0`, and aborted on any
  redirect/URL drift. Transport exceptions are deliberately swallowed
  because they can embed auth codes. No CAPTCHA/MFA bypass anywhere;
  unknown forms fall back to visible manual login.
- **Read-only discipline**: no `fetch` writes are originated by the tool
  (the POST references in code *classify and validate* the portal's own
  UI-generated requests); PDF download is GET-only with
  `redirect:'error'`, `same-origin`, content-type, `%PDF-`/`%%EOF`
  signature, size cap, and SHA-256 custody re-verification before
  publication (`coretax_pdf.py`, `coretax_attachments.py`).
- **Excel safety**: source text is forced to literal text cells
  (`data_type='s'` — anti formula-injection), only HTTP/HTTPS become live
  hyperlinks, >32,767-char cells and precision-unsafe numbers abort the
  export explicitly rather than truncating/rounding taxpayer figures.
- **Diagnostics**: category-constant allowlist logging; URLs, NPWP, form
  content and raw stdout/stderr never cross into logs (there is even a
  canary test using `'SYNTHETIC PRIVATE VALUE MUST NOT BE LOGGED'`).
- **Process safety**: Windows Job Objects with suspended-child containment,
  15-second forced cleanup, no `shell=True` anywhere, every `subprocess`
  call uses argv lists with timeouts; no `eval/exec/pickle/yaml.load`;
  no `verify=False`. Zero bare `except:` in production code.
- Output publication is paranoid in the good way: scope/fingerprint/race
  re-verification, case-insensitive collision rejection, crash-recovery
  journal with byte-level custody (`publication_recovery.py`).

### Security findings / residual risks

| # | Severity | Finding |
| --- | --- | --- |
| S-1 | Medium (fork) | SECURITY.md's private-report link and maintainer (`@cepeter`) point to upstream. Fork users have **no valid private disclosure channel** for fork-specific issues until this is rewritten and private vulnerability reporting is enabled on the fork. |
| S-2 | Low (known) | `SPTDownloader.exe` is **not Authenticode-signed**. Honestly documented (README, SECURITY, remediation doc); checksum+attestation mitigate tampering but not publisher trust/SmartScreen friction. |
| S-3 | Low | The app's only support contact is a **personal government mailbox** hard-coded as the bug-report footer (`sharmila@kemenkeu.go.id`, in `ui_copy.py` and asserted by tests/build). Continuity/privacy risk if the person changes roles; a role/shared mailbox is preferable. |
| S-4 | Low (accepted, documented) | `run.cmd` auto-reuses a browser debugging endpoint on `127.0.0.1:9235` when one answers. README warns about local-process exposure; loopback-only origin validation exists in code. Acceptable, but on shared/multi-user Windows endpoints another local process could race that port — keep the warning prominent. |
| S-5 | Info | 52 blind `except Exception` and 10 `try/except/pass` in production code (2 are truly silent-pass). Mostly justified (teardown, transport secrecy) and often commented as such, but each is a place real defects can hide; a lint allowlist would freeze the current set. |
| S-6 | Info | Tracked `.xlsx`/`.json` files exist despite ignore rules (force-added templates + ruleset). The ignore net works for *new* files, but `git add -f` habits around taxpayer-shaped formats deserve a pre-push secret scan (e.g. gitleaks) — none is configured in-repo. GitHub secret-scanning status on either repo could not be confirmed from this sandbox (API 403). |

No evidence of exfiltration paths, telemetry, update-check phone-home, or
third-party endpoints beyond package indexes and the two approved
intranet origins (plus `learn.microsoft.com` documentation links in
comments).

## 5. Dependencies and supply chain

- Direct runtime deps: exactly two — `playwright==1.64.0`,
  `openpyxl==3.1.5`. Both are the **latest PyPI releases** at audit date,
  as are `pytest 9.1.1`, `pyinstaller 6.22.3`, `pip-audit 2.10.1`,
  `PyYAML 6.0.3`, `greenlet 3.5.6`, `et-xmlfile 2.0.0`.
- Slightly behind: `uv 0.13.4` vs `0.13.5` (one patch — Dependabot will
  propose it; pinning uv is itself deliberate), and transitive `pyee
  14.0.0` vs `15.0.0` (pyee 15 needs Python ≥3.12 and will arrive with a
  future Playwright bump; not actionable independently).
- Four hash-locked lockfiles (`locks/{runtime,test,build,audit}.txt`,
  952 lines, every package SHA-256-hashed: 95–378 hashes each), generated
  by `scripts/update_locks.py` with the pinned uv, drift-checked in CI
  (`update_locks --check`), and even re-parsed at *runtime* by the
  desktop bootstrapper, which rejects any pin lacking a hash.
- CI runs `pip-audit --strict` over build+test+auditor environments
  (job green at HEAD; the audit report artifact itself couldn't be
  downloaded from this sandbox due to egress limits, but the gate is
  `if: mode != 'docs'` and the job passed).
- Actions are **all pinned to full commit SHAs** (checkout v7.0.1,
  upload/download-artifact, setup-uv with binary checksum, attest),
  `persist-credentials: false` everywhere, workflow-level
  `permissions: contents: read` with write scopes granted per-job only
  where needed (`attest`: id-token/attestations; `publish`: contents;
  `cleanup`: contents+PR-read). The one `curl` (actionlint) is
  SHA-256-verified before execution.
- The package job produces a 7-file allowlisted artifact with
  `build-info.json`, a CycloneDX-style distribution SBOM of the actual
  packaged bytes, `SHA256SUMS.txt`, then **re-downloads its own artifact
  by ID and re-verifies** before build-provenance attestation
  (including attesting the artifact ZIP digest for rollback).
- Python itself is pinned to the exact patch **CPython 3.14.8 x64/GIL**
  (`PYTHON_PROFILES`), provisioned by uv-managed installs — no reliance
  on whatever Python happens to be on PATH. Note the code itself uses
  no 3.12+ syntax (it byte-compiles under 3.11); the pin is a support
  policy, not a language requirement. That makes the policy portable
  but also means every CPython patch CVE forces a coordinated pin bump
  across README/docs/workflows/`setup.cmd` (many literal `3.14.8`
  mentions, albeit generated from one source of truth at runtime).

This is supply-chain practice well above the norm for a project this size.

## 6. Testing

- **1,395 test functions** in 175 `test_*.py` files with 631
  `parametrize` decorators (static count; collected items are higher).
- Two workloads: a reviewed **portable** subset (23 files) on Linux and
  the **full** suite on Windows (Tk, Credential Manager, Job Objects,
  real Edge against local synthetic HTML fixtures — never live taxpayer
  data). Deterministic SHA-256-based 2-shard partitioning.
- The evidence system is the standout: each run emits a manifest of
  *all collected* node IDs and per-phase outcomes; the package job
  refuses to build unless portable+Windows collectors agree on the full
  inventory, shards exactly tile it, and **every test passed setup/call/
  teardown exactly once — skips and xfails fail the gate**
  (`scripts/ci_tests.py`). This makes the classic "green because tests
  silently didn't run" failure mode structurally impossible.
- HEAD commit evidence (upstream run 39728441622): portable 49s,
  Windows shards 10m30s/11m04s, dependency audit 27s, package 1m01s,
  attestation 18s — all green. Recent `main` history is mostly green
  (one failure at PR #66's merge run and one cancelled run among the
  last 12 — normal for this velocity, and the required-check gate is on
  PRs; note a red *post-merge* main run did occur once, worth a
  "main must stay green" alert, which SECURITY.md assigns to the
  maintainer manually).
- Gaps: **no coverage measurement or threshold** (`.coverage` is
  gitignored but nothing produces it in CI), no mutation/fuzz testing,
  and per project policy **no live acceptance** is automated —
  `docs/penerimaan-manual.md` keeps that a human, authorized activity.
  Given the evidence-manifest rigor, adding a coverage *report* (not
  necessarily a hard gate) is cheap, missing signal.
- Per `AGENTS.md`, test execution is GitHub-runner-only; this audit
  therefore did **not** run pytest locally and relies on the CI evidence
  above (plus local syntax compilation of every file, which passed).

## 7. CI/CD workflows

| Workflow | Trigger | Notes |
| --- | --- | --- |
| `windows-build.yml` (543 lines) | push/PR to `main`, dispatch, weekly Mon 03:17 UTC | route → portable + Windows shards + audit → package → attest → `validate` gate → branch cleanup. Docs-only changes take a short route (`ci_route`), defaulting to full on any doubt. Per-main-push concurrency slots prevent docs pushes cancelling code builds. |
| `release-promotion.yml` (125) | manual dispatch only | Verifies an exact prior artifact (run/attempt/commit/Python/artifact-ID + provenance) and only publishes with explicit `publish=true` on `main`; never overwrites an existing tag. **Never yet used to publish.** |
| `windows-shard-benchmark.yml` (178) | manual dispatch only | Performance comparison harness, isolated from the required check. |

Observations:

- The `cleanup` job (`contents: write`) auto-deletes merged branches via
  `scripts/cleanup_branches.py`, guarded to heads already contained in
  the tested `main` — and a same-day fix (in the audited commit) ensures
  stale-branch cleanup can't fail a verified build. Still, it is the
  broadest standing write permission in the pipeline; keep it scoped.
- Artifact retention is uniformly **14 days**, including the only
  user-facing download. Combined with §1's "no releases", a user
  following the README three weeks after the last `main` push may find
  every link dead. This is the single most user-visible defect found.
- Caches are save-gated to `main` pushes, keyed by lockfile globs, with
  a cold mode for honest timing — solid cache-poisoning hygiene.
- Everything is useless **on this fork until Actions are enabled**, and
  the `attest`/`cleanup` jobs additionally need the fork's Actions
  permissions to allow OIDC/attestations and contents-write.

## 8. Documentation

- 69 Markdown files (~7.6k lines): user README (Indonesian, thorough),
  contributor index, architecture/output contracts, CI & release,
  troubleshooting, privacy maintenance, manual acceptance, UI/UX
  baselines, plus a `docs/superpowers/` archive of specs/plans/evidence
  and dated internal audits (`docs/audits/`). **0 of 376 internal
  relative links are broken.**
- The docs are unusually honest about limits: unsigned EXE, non-atomic
  multi-file publication, "COMPLETE means the confirmed subset",
  synthetic tests ≠ live acceptance. The previous internal DevOps audit
  and its remediation map (2026-10-22, in the audited commit) list five
  controls still requiring out-of-repo evidence — Authenticode, first
  durable release, admin alert verification, endpoint ACL/encryption
  validation, live acceptance. This audit concurs with all five.
- Gaps: no `CONTRIBUTING.md` file (guidance lives in `docs/` + AGENTS.md
  — GitHub won't surface it in the PR flow), no `CODE_OF_CONDUCT.md`,
  no PR/issue templates (issues are disabled anyway), no `CHANGELOG.md`.
  Upstream community profile: 50%.
- Language mix: user-facing docs in Indonesian, code comments mixed
  ID/EN, security-critical comments often in English. Appropriate for
  the audience; external contributors face a language barrier that is
  presumably intentional.
- **Fork identity**: 231 `cepeter-job` references, 0 `punzer4-code`.
  Beyond docs, `build_desktop.py:118` embeds the upstream URL into the
  shipped `README.txt`. For a pure mirror this is correct; for an
  independent fork it is a trust/support bug (see S-1).

## 9. Packaging and platform

- PyInstaller `--onefile --windowed --noupx`, explicit `--add-data`
  allowlist (= `WORKER_FILES` + license + build-info), Playwright/
  openpyxl/pytest excluded from the GUI binary, bundled pinned `uv.exe`
  provisions the isolated worker at `%LOCALAPPDATA%\SPTDownloader`.
- Windows 10/11 x64 only, by design and clearly stated. The CLI path
  (`setup.cmd`/`run.cmd`) enforces uv 0.13.4 + managed CPython 3.14.8
  rather than mutating system Python.
- First-run requires package-index network access — inside a restricted
  government intranet this is the most likely deployment friction;
  proxy/TLS env is preserved by `bootstrap_environment`, and failures
  surface as retryable setup errors rather than partial installs.

## 10. Legal and data-protection posture

- GPL-3.0 `LICENSE` present and detected by GitHub; README and the
  shipped artifact carry it. **No per-file license/SPDX headers** (0/45
  production files) — not required, but headers protect the notice when
  files are copied out of context, which matters for GPL compliance of
  *downstream* forks (including this one).
- The GUI states "for personal use only"; AGENTS.md codifies "commit
  synthetic data only", which history confirms. Workbooks/checkpoints
  are documented as confidential, unencrypted, local-only.
- As a public fork of a tool targeting a tax-authority intranet, the
  repo inevitably publishes internal hostnames and portal paths. That
  ship sailed upstream (also public); the fork adds no new exposure,
  but the fork owner should consciously re-confirm that public is the
  intended visibility for their copy.

## Findings register (prioritized)

| ID | Sev | Area | Finding |
| --- | --- | --- | --- |
| F-1 | **Medium** | Fork governance | Fork `main` is unprotected: no ruleset applied, Actions disabled (0 workflows), Dependabot alerts disabled, no code scanning. Upstream protections do not inherit. |
| F-2 | **Medium** | Distribution | No tags/releases anywhere; sole download channel is a CI artifact that expires in 14 days. README instructions silently rot. Promotion workflow exists but unused. |
| F-3 | **Medium** | Fork identity | 231 upstream-owner references incl. SECURITY reporting URL, README download link, and URL embedded in the built EXE's README. Fork users/reporters are directed to upstream. |
| F-4 | Low | People | Bus factor 1; sole support/security contact is one personal government mailbox hard-coded in the UI and build assertions. |
| F-5 | Low | Code quality | No lint/format/type/coverage gates; 53 functions exceed McCabe 15 (max 61); 7% annotation coverage. High test volume mitigates but doesn't replace these signals. |
| F-6 | Low | Scanning portability | CodeQL is GitHub default-setup upstream, not in-repo config; fork has none, and upstream's ruleset requires its check context. |
| F-7 | Low | Hygiene | 3 template workbooks duplicated byte-for-byte in `resources/` and `tests/fixtures/` (drift risk); tests use `sys.path.insert`; no `pyproject.toml`/packaging metadata. |
| F-8 | Low | Robustness | Late-binding lambda at `spt_gui.py:79` (F821) in self-test; 52 blind excepts (mostly justified) with no lint allowlist to freeze them. |
| F-9 | Low (known) | Trust | Executable unsigned (Authenticode pending per upstream remediation doc). |
| F-10 | Info | Community | Issues disabled on fork+upstream; no CONTRIBUTING/CHANGELOG/templates; repo description/topics empty on the fork. |
| F-11 | Info | Freshness | `uv` one patch behind (0.13.4→0.13.5); transitive `pyee` one major behind via Playwright pin. Everything else is latest. |
| F-12 | Info | Process | One red post-merge `main` run in recent history (PR #66 era) — required checks gate PRs, but `main` health after merge relies on maintainer attention. |

## Recommendations

**For the fork owner (`punzer4-code`), in order:**

1. **Decide the fork's purpose** — mirror, contribution staging, or
   independent line — because it determines everything below. If it's a
   mirror, keep upstream identity and just enable sync. If independent:
2. **Turn on the safety net**: Settings → enable Actions; import
   `.github/rulesets/main.json` as an active ruleset (adjust the CodeQL
   context per #4); enable Dependabot alerts + secret scanning/push
   protection + private vulnerability reporting; confirm notification
   routing actually reaches a human.
3. **Fix identity**: replace `cepeter-job` → `punzer4-code` in README,
   SECURITY.md (reporting URL + maintainer), and `build_desktop.py`'s
   embedded README; set repo description/topics. Decide the support
   mailbox that replaces the hard-coded footer (requires touching
   `ui_copy.py` + its asserting tests).
4. **Add code scanning as code**: commit a CodeQL workflow (or enable
   default setup on the fork) so the ruleset's required check can pass.
5. **Publish one durable release**: run `release-promotion.yml` with
   `publish=true` against a verified `main` artifact (e.g. upstream run
   39728441622's artifact lineage, or the fork's own first green run),
   then point the README at the Release instead of a 14-day artifact.
6. **Cheap quality gates**: add a `ruff.toml` with a *ratchet* config
   (current findings baselined, new violations fail), plus a coverage
   report artifact in the portable job. Fix the `spt_gui.py:79` lambda
   (`lambda s=settled: s.set(True)`). Deduplicate the fixture workbooks.
7. **Keep upstream in sync deliberately**: the upstream moves fast
   (72 PRs/13 days, draft PR #71 pending). Either enable the fork's
   "Sync fork" cadence or accept divergence explicitly — silent drift
   is how forks inherit unpatched behavior near tax deadlines.

**Worth sending upstream as contributions:** the Ruff baseline, fixture
deduplication, coverage reporting, and per-file SPDX headers — all are
fork-neutral improvements.

## Methodology and limitations

Performed: full tree + unshallowed history review (212 commits);
secret/PII pattern scans over tree and all added lines in history;
tracked-binary (XLSX) content inspection; dangerous-API greps
(eval/exec/subprocess/TLS/pickle); AST metrics (annotations, docstrings,
function length, McCabe via Ruff 0.17.1); import-reachability vs
`WORKER_FILES`; lockfile↔requirements consistency; PyPI latest-version
comparison (2026-10-22); workflow permission/pin review; Markdown
internal-link validation (376 links); GitHub API review of both repos
(fork relation, branches, rulesets, runs, PRs, community profile).

Not performed, by policy or sandbox limits: **no local test execution**
(`AGENTS.md` reserves execution evidence to GitHub runners — upstream
run 39728441622 for this exact commit is the execution evidence);
managed CPython 3.14.8 could not be downloaded in this sandbox (TLS
issuer rejection), so local checks used Python 3.11 AST/compilation
only; CI artifact/log *bytes* could not be downloaded here (storage
host outside the sandbox egress allowlist), so artifact contents are
assessed from workflow definitions + job results, not byte inspection;
several GitHub admin APIs returned 403 to this integration (branch
protection detail, secret/code-scanning and Dependabot alert *lists*,
CodeQL default-setup) — the fork owner should verify those toggles in
Settings → Security; no live Coretax access was used or attempted, and
endpoint/ACL/encryption claims remain, as upstream's own remediation
doc states, matters for on-site validation.

## Appendix — key metrics

| Metric | Value |
| --- | --- |
| Tracked files | 353 |
| Production Python | 57 files / 18,376 LOC |
| Test Python | 202 files / 31,855 LOC |
| Test functions (static) | 1,395 (+631 parametrize) in 175 files |
| Docs | 69 files / ~7,568 lines; 0/376 broken internal links |
| Commits / period | 212 / 2026-10-09 → 2026-10-22 |
| Dependencies | 2 direct runtime; 6 pkgs in runtime lock; all hash-pinned; all latest except uv (−1 patch) & transitive pyee |
| Largest modules | `coretax_browser.py` 1,579; `spt_desktop/app.py` 1,013 |
| Max function complexity | `collect_inventory` McCabe 61 |
| TODO/FIXME in tree | 0 |
| Secrets found (tree+history) | 0 |
| Tags / Releases | 0 / 0 |
| HEAD CI (upstream) | run 39728441622, success, 13m06s, 9/9 jobs |
