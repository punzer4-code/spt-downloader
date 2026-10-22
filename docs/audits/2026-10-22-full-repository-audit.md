# Full repository audit — `cepeter-job/spt-downloader` (upstream)

| Field | Value |
| --- | --- |
| Audit date | 2026-10-22 |
| Repository audited | **`cepeter-job/spt-downloader`** (upstream, public) |
| Commit audited | `44969fcd73534153f5b656ec5ee9f5c8f11f42f4` — verified as upstream `main` HEAD at audit time |
| Audit route | Identical mirror checkout at the same SHA (the `punzer4-code` fork: `ahead 0 / behind 0`) + live GitHub API review of upstream |
| CI evidence for this commit | Run [39728441622](https://github.com/cepeter-job/spt-downloader/actions/runs/39728441622) — **success**, 13m06s, all 9 jobs green |
| Auditor method | Static/source audit + GitHub API + CI evidence (see Limitations) |
| Revision note | This revision re-bases the audit on upstream per request; an earlier same-day draft framed findings around the fork and is superseded. Code-level evidence is unchanged (identical commit); governance findings below are upstream's. |

## Executive summary

This codebase is, by a wide margin, more disciplined than typical internal
automation tools: hash-locked dependencies, SHA-pinned Actions, provenance
attestation, an anti-skip test-evidence system, exact-origin credential
isolation, and a read-only browsing discipline enforced in code rather than
merely documented. At the audited commit the full Windows test suite,
dependency audit, packaging, and attestation pipeline are green, and `main`
is protected by an active ruleset requiring PRs plus CI/CodeQL checks.

**No critical or high-severity *code* vulnerabilities were found.** One
high-priority *process* finding and a few mediums deserve attention:

1. **The documented vulnerability-reporting channel is dead.**
   `SECURITY.md` directs reporters to GitHub's "Report a vulnerability"
   form, but the API confirms **private vulnerability reporting is disabled
   (`"enabled": false`)**, and Issues are disabled too. An external
   reporter following the instructions hits a dead end — for a tool that
   handles taxpayer data, that is exactly how a real flaw gets posted
   publicly or never reported. It is a two-minute settings fix.
2. **There is no durable release.** Zero tags, zero GitHub Releases. The
   README tells users to download a CI artifact, which expires after
   **14 days**. The promotion workflow exists but has never published.
3. **Merge authority is single-operator.** The ruleset requires **0
   approving reviews**, there is no `CODEOWNERS`, and PR "reviews" sampled
   are bot comments (`github-code-quality[bot]`) and the author's own
   comments. The automated gates are excellent — and they are the *only*
   review. With AI-agent-authored commits already in history, an
   independent human approval would materially raise assurance.

Overall grades:

| Area | Grade | One-line basis |
| --- | --- | --- |
| Application security design | **A** | Origin-pinned credentials, request guard, no write APIs, formula-safe export, PDF custody checks |
| Disclosure & security process | **D** | SECURITY.md's private channel is disabled; Issues off; email is the only working path |
| Supply chain / CI | **A** | Hash-locked installs, SHA-pinned actions, SBOM + attestation, pip-audit gate, least-privilege tokens |
| Test discipline | **A−** | 1,395 test functions, evidence manifests that make silent skips impossible; no coverage metric |
| Repository governance | **B+** | Active ruleset + required checks, but 0 required approvals, no CODEOWNERS, bus factor 1 |
| Code maintainability | **B** | Compact style, low annotation (7%), a few very complex functions, no lint/type gate |
| Documentation | **A−** | 69 Markdown files, 0/376 broken internal links; no CONTRIBUTING/CHANGELOG |
| Release & distribution | **C** | No release, no tag, unsigned EXE, 14-day artifact expiry as the only channel |

---

## 1. Repository identity and GitHub posture (upstream)

### Facts

- `cepeter-job/spt-downloader`: **public**, created `2026-10-09T03:24:20Z`
  (13 days old at audit), 2,110 KB, GPL-3.0 detected. Description: **null**.
  Topics: **none**. Homepage: **none**. Stars/watchers: 0. Forks: 1.
- Issues: **disabled**. Wiki/discussions/downloads: disabled. Projects:
  enabled. Environments: none. `open_issues_count: 1` is the open draft PR.
- `main` is **protected** (`protected: true`) by the active ruleset
  *"main melalui PR dan pemeriksaan wajib"* (id 24914050, created
  2026-10-12; also codified in `.github/rulesets/main.json`):
  PR-only merges, deletion and non-fast-forward blocked, stale reviews
  dismissed on push, review threads must resolve, strict required status
  checks — `Windows tests and dependency audit` and `CodeQL` — **but
  `required_approving_review_count: 0` and `require_code_owner_review:
  false`**, and there is no `CODEOWNERS` file anywhere.
- Branches at audit time (4): `main` @ `44969fc`;
  `feat/ui-delivery-completion` (open **draft PR #71**, 11 commits ahead,
  22 files; checks passing/pending during the audit);
  `feat/tk-accessibility-candidate` (**55 commits ahead, 42 files**, no PR
  open, long repeatedly-failing WIP runs); `feat/tk-display-diagnostic`
  (new diagnostic branch). The merged `fix/devops-audit-20261022` branch
  had already been auto-deleted — direct evidence the CI branch-cleanup
  job works as designed.
- Code scanning: CodeQL **default setup** (dynamic workflows `CodeQL` and
  `CodeQL - Code Quality`, analyzing Python + Actions, green on `main`
  and PRs). There is no CodeQL config/workflow in the repository, so the
  setup is not version-controlled, reviewable, or portable.
- Dependabot **version updates are active** (`Dependabot Updates` workflow
  present; `.github/dependabot.yml` weekly/grouped for pip + Actions;
  PR #56 merged a Dependabot `uv` bump). **Dependabot alerts** status
  could not be confirmed from this sandbox (API 403). Dependency Graph
  workflow is active.
- **Private vulnerability reporting: `"enabled": false`** (confirmed via
  API — see Executive summary #1). Secret-scanning/push-protection status
  is not visible to non-admin API access (`security_and_analysis`
  returns null); SECURITY.md itself instructs the maintainer to verify
  these in the UI, and this audit cannot close that item.
- 16 workflows are registered, of which only 3 exist on `main`. The rest
  are branch-local candidate/diagnostic workflows and at least five
  explicitly **"(temporary)" evidence workflows still marked active**
  (`Phase 2 … (temporary)`, `PR20 and PR50 … (temporary)`, `T5 …
  (temporary)`, …) whose branches are long gone. They can't run without
  their files, but they clutter the Actions surface and invite confusion.
- Community profile: **50%** (README + license detected; no contributing
  file, code of conduct, or templates surfaced by GitHub).

### Assessment

Branch protection, required checks, and automated dependency updates are
genuinely in place — this is a well-governed repository for a solo project.
The gaps are concentrated where solo projects usually have them: nobody
else must approve, nobody else is notified, and the one documented
security-contact mechanism was never switched on.

## 2. History and development process

- **212 commits in 13 days** (2026-10-09 → 2026-10-22) on the path to
  `main`. Peak days: 58 commits (Oct 13), 43 (Oct 21).
- **72 PRs: 62 merged, 9 closed unmerged, 1 open (draft #71).** The
  unmerged set is largely deliberate throwaway evidence PRs labelled
  "do not merge" — an unusual, rigorous pattern for proving behavior on
  runners without landing the proving code.
- Authors: one human maintainer under three identities
  (`cepeter97@gmail.com` ×117, noreply ×84) — **bus factor = 1** — plus
  `ChatGPT Coding Agent` (×8) and `dependabot[bot]` (×3). AI assistance
  is visible in history rather than disguised. Merged-by is the author
  in sampled PRs; review objects are `github-code-quality[bot]` comments
  and author self-comments (e.g. PR #72: 6 review objects, 0 approvals,
  0 independent humans). The ruleset's thread-resolution requirement at
  least forces those bot threads to be addressed.
- Commit style: conventional prefixes (`test` 67, `fix` 49, `feat` 37,
  `docs` 21, merges 23). *test* being the most common type matches the
  test-first behavior the CI evidence system enforces. The audited HEAD
  merge commit is GitHub-signed (`verified: true`).
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
  new contributors — which compounds the single-reviewer problem in §1.
  There is **no formatter/linter/type-checker gate** anywhere (no
  `pyproject.toml`, `ruff.toml`, pre-commit, or CI step). A one-off Ruff
  pass (broad ruleset) reports 199 findings in production code — dominated
  by unsorted imports (65) and blind `except Exception` (52), much of it
  intentional defensive style — and 664 in tests, where 164× F811 is the
  normal pytest fixture-shadowing pattern (noise).
- **Typing/docstrings**: 7% of 609 production functions carry annotations;
  18% have docstrings. `tuple[str, bytes] | None`-style modern syntax is
  used in newer modules (`credentials.py` is exemplary); older engine
  modules are largely unannotated.
- One genuine lint hit worth fixing: **F821/B023-style late binding** at
  `spt_gui.py:79` — `root.after(250, lambda: settled.set(True))` inside a
  loop that later `del settled`. It works today only because
  `wait_variable` blocks until the callback fires; fragile to reordering.
  (Self-test code, so impact is test reliability, not user data.)
- Dead weight is minimal: 0 TODO/FIXME in the tree, 4 unused imports.
- **Duplicate binaries**: the three template workbooks in `resources/` are
  byte-identical copies of `tests/fixtures/*.xlsx` (same MD5s, ~42 KB
  duplicated). Drift risk if one copy is ever edited without the other.
- Tests use `sys.path.insert` (13 files) instead of packaging/`conftest.py`
  — works, and CI enforces collection parity between Linux and Windows,
  but it will bite if the project is ever packaged.
- There is **no `pyproject.toml`**: the project is not pip-installable; it
  runs as scripts from a checkout or as the frozen EXE. Legitimate for
  this tool, but means no standard metadata or `requires-python`.
- The owner/repo string `cepeter-job/spt-downloader` is hard-coded in
  **231 places**, including the shipped EXE's README via
  `build_desktop.py`. Correct today; a repository transfer or rename
  would need a sweep (and any fork inherits upstream pointers verbatim).

## 4. Security audit

Scope note: this tool handles taxpayer data against a government intranet
portal (`coretax.intranet.pajak.go.id`, SSO via
`satu.intranet.pajak.go.id`). It only functions with authorized intranet
access; nothing in the code attempts to bypass that.

### Strengths verified in code

- **No secrets in tree or history.** Full-history scan (212 commits, added
  lines) for token/key/password patterns: clean. All 15/16-digit strings
  in the tree are synthetic (`0000000000000001`), precision constants
  (2^53−1), or placeholders. `.gitignore` blocks `*.xlsx/.json/.pdf/.env/
  *.pem/*.key`, browser profiles, cookies, and output folders. The seven
  tracked `.xlsx` files are layout templates; their internals contain only
  form labels (`corporate_format.xlsx` even carries `description:
  "Sanitized corporate layout; no taxpayer values"`).
- **Credentials** (`spt_desktop/credentials.py`): Windows Credential
  Manager only, targets pinned to an exact-origin allowlist,
  `Login.password` excluded from `repr`, blob size/parity validated,
  write buffer zeroed after `CredWriteW`. Preferences store no
  password/token/cookie.
- **Auth request guard** (`spt_desktop/auth.py`): a Playwright route
  interceptor aborts any request outside approved origins and any
  cross-origin write; the one allowed code-exchange is consumed-once,
  deadline-bound, fetched with `max_redirects=0`, aborted on redirect/URL
  drift. Transport exceptions are deliberately swallowed because they can
  embed auth codes. No CAPTCHA/MFA bypass anywhere; unknown forms fall
  back to visible manual login.
- **Read-only discipline**: the tool originates no write requests (POST
  references *classify and validate* the portal's own UI-generated
  requests); PDF download is GET-only with `redirect:'error'`,
  same-origin, content-type, `%PDF-`/`%%EOF` signature, size cap, and
  SHA-256 custody re-verification before publication.
- **Excel safety**: source text forced to literal text cells
  (`data_type='s'` — anti formula-injection), only HTTP/HTTPS become live
  hyperlinks, >32,767-char cells and precision-unsafe numbers abort the
  export explicitly rather than truncating taxpayer figures.
- **Diagnostics**: category-constant allowlist logging; URLs, NPWP, form
  content and raw stdout/stderr never cross into logs (canary test:
  `'SYNTHETIC PRIVATE VALUE MUST NOT BE LOGGED'`).
- **Process safety**: Windows Job Objects with suspended-child
  containment, 15-second forced cleanup, no `shell=True`, argv-list
  subprocess calls with timeouts; no `eval/exec/pickle/yaml.load`;
  no `verify=False`; zero bare `except:` in production code.
- Output publication is paranoid in the good way: scope/fingerprint/race
  re-verification, case-insensitive collision rejection, crash-recovery
  journal with byte-level custody.
- No exfiltration paths, telemetry, or phone-home beyond package indexes
  and the two approved intranet origins.

### Security findings / residual risks

| # | Severity | Finding |
| --- | --- | --- |
| S-1 | **High (process)** | **Private vulnerability reporting is disabled** (`"enabled": false`) while `SECURITY.md` sends reporters to that form; Issues are also disabled. The fallback text ("ask the maintainer via a private channel you already have") only works for insiders. For anyone else the disclosure path is a dead end. *Not a code vulnerability* — but the highest-priority fix in this report. |
| S-2 | Medium | Alert **delivery** is unverified end-to-end: Dependabot-alerts and secret-scanning toggles aren't confirmable via API from outside, and upstream's own remediation doc lists "verify backlog, routing and actual notification delivery" as outstanding. Combined with bus-factor 1, a critical alert could sit unread. |
| S-3 | Low (known) | `SPTDownloader.exe` is **not Authenticode-signed**. Honestly documented; checksum+attestation mitigate tampering but not publisher trust/SmartScreen friction. |
| S-4 | Low | Solo support contact is a **personal government mailbox** hard-coded as the app bug-report footer (`sharmila@kemenkeu.go.id`, asserted by tests/build). Continuity/privacy risk on role change; a role/shared mailbox is preferable. (Note the mailbox owner differs from the repo maintainer handle — support and maintenance are already split across two people with no documented backup for either.) |
| S-5 | Low (accepted, documented) | `run.cmd` auto-reuses a browser debugging endpoint on `127.0.0.1:9235` when one answers. Loopback-only validation exists and README warns about local-process exposure; on shared Windows endpoints another local process could race that port. |
| S-6 | Info | 52 blind `except Exception`, 10 `try/except/pass` in production code. Mostly justified and commented (teardown, transport secrecy); a lint allowlist would freeze the current set. |
| S-7 | Info | Tracked `.xlsx`/`.json` exist despite ignore rules (force-added templates + ruleset). The ignore net works for *new* files; consider an in-repo pre-push/CI secret scan (e.g. gitleaks) so protection doesn't depend solely on GitHub-side toggles whose status is unverified (S-2). |

## 5. Dependencies and supply chain

- Direct runtime deps: exactly two — `playwright==1.64.0`,
  `openpyxl==3.1.5`. Both are the **latest PyPI releases** at audit date,
  as are `pytest 9.1.1`, `pyinstaller 6.22.3`, `pip-audit 2.10.1`,
  `PyYAML 6.0.3`, `greenlet 3.5.6`, `et-xmlfile 2.0.0`.
- Slightly behind: `uv 0.13.4` vs `0.13.5` (one patch; Dependabot is
  active and will propose it), transitive `pyee 14.0.0` vs `15.0.0`
  (arrives with a future Playwright bump; not independently actionable).
- Four hash-locked lockfiles (`locks/{runtime,test,build,audit}.txt`,
  every package SHA-256-hashed), generated by `scripts/update_locks.py`
  with the pinned uv, drift-checked in CI, and re-parsed at *runtime* by
  the desktop bootstrapper, which rejects any pin lacking a hash.
- CI runs `pip-audit --strict` over build+test+auditor environments
  (green at HEAD; the report artifact's bytes couldn't be downloaded
  from this sandbox due to egress limits, but the gate ran and passed).
- Actions **all pinned to full commit SHAs**, `persist-credentials:
  false` everywhere, workflow-level `contents: read` with write scopes
  per-job only (`attest`, `publish`, `cleanup`). The one `curl`
  (actionlint) is SHA-256-verified before execution.
- The package job emits a 7-file allowlisted artifact with
  `build-info.json`, a distribution SBOM of the actual packaged bytes,
  and `SHA256SUMS.txt`, then **re-downloads its own artifact by ID and
  re-verifies** before build-provenance attestation (including the
  artifact ZIP digest for rollback).
- Python pinned to exact patch **CPython 3.14.8 x64/GIL** via uv-managed
  installs. The code uses no 3.12+ syntax (byte-compiles under 3.11), so
  the pin is support policy, not language need — every CPython patch
  CVE will force a coordinated bump, which the single
  `PYTHON_PROFILES` source of truth makes mechanical.

Supply-chain practice here is well above the norm for a project this size.

## 6. Testing

- **1,395 test functions** in 175 `test_*.py` files, 631 `parametrize`
  decorators (static count; collected items are higher).
- Two workloads: a reviewed **portable** subset (23 files) on Linux and
  the **full** suite on Windows (Tk, Credential Manager, Job Objects,
  real Edge against local synthetic HTML fixtures — never live taxpayer
  data). Deterministic SHA-256-based 2-shard partitioning.
- The evidence system is the standout: each run emits a manifest of
  *all collected* node IDs and per-phase outcomes; the package job
  refuses to build unless portable+Windows collectors agree on the full
  inventory, shards exactly tile it, and **every test passed setup/call/
  teardown exactly once — skips and xfails fail the gate**. "Green
  because tests silently didn't run" is structurally impossible.
- HEAD evidence (run 39728441622): portable 49s, Windows shards
  10m30s/11m04s, audit 27s, package 1m01s, attestation 18s — all green.
  Recent `main` runs are mostly green (one failure — PR #66's post-merge
  run — and one cancellation among the last 12). Note the asymmetry:
  required checks gate the *PR*; a red *post-merge* `main` run has
  happened once, and nothing automated escalates it (SECURITY.md assigns
  watching `main` failures to the maintainer).
- Gaps: **no coverage measurement or threshold** in CI, no mutation/fuzz
  testing, and by policy no automated live acceptance
  (`docs/penerimaan-manual.md`). A coverage *report* artifact is cheap,
  missing signal given how much else is measured.
- Per `AGENTS.md`, test execution is GitHub-runner-only; this audit did
  not run pytest locally and relies on the CI evidence above (plus local
  syntax compilation of every file, which passed).

## 7. CI/CD workflows

Workflows on `main`:

| Workflow | Trigger | Notes |
| --- | --- | --- |
| `windows-build.yml` (543 lines) | push/PR to `main`, dispatch, weekly Mon 03:17 UTC | route → portable + Windows shards + audit → package → attest → `validate` gate → branch cleanup. Docs-only short route defaults to full on doubt. Per-push concurrency slots stop docs pushes cancelling code builds. |
| `release-promotion.yml` (125) | manual dispatch only | Verifies exact prior artifact + provenance; publishes only with explicit `publish=true` on `main`; never overwrites a tag. **Never yet used to publish.** |
| `windows-shard-benchmark.yml` (178) | manual dispatch only | Sharding performance harness, isolated from the required check. |

Plus branch-local workflows on WIP branches (`launcher-candidate.yml`,
`ui-delivery-diagnostic.yml`, display-diagnostic) and the dynamic CodeQL
pair.

Observations:

- **CI volume/cost**: of the last 100 runs, 44 succeeded, **42 failed**,
  10 cancelled — driven overwhelmingly by the candidate branch
  (`Isolated launcher candidate` alone = 40 of 100 runs, several failing
  after 19–25 minutes). This is test-first development executed on
  Windows runners (the AGENTS.md runner-only policy *requires* it), so
  red WIP runs are expected — but the burn rate is high, four heavyweight
  runs were in flight simultaneously during this audit, and if this repo
  ever moves to a metered/private setting the Windows-minutes bill will
  be the first shock. The portable/docs routing already helps; pushing
  candidate iterations less frequently (or squash-and-push) would too.
- **WIP divergence**: `feat/tk-accessibility-candidate` is 55 commits /
  42 files ahead with no PR and a long red streak. The longer it lives,
  the more its eventual merge (or abandonment) costs; landing it in
  slices behind the usual PR gates would de-risk it. Draft PR #71
  (11 commits) is the healthier pattern.
- The `cleanup` job (`contents: write`) auto-deletes merged branches,
  guarded to heads contained in the tested `main` — observed working
  during this audit. It remains the broadest standing write permission;
  keep it scoped as-is.
- Artifact retention is uniformly **14 days**, including the only
  user-facing download (§8). Most user-visible defect in the delivery
  chain.
- Caches are save-gated to `main` pushes, keyed by lockfile globs, with
  an honest cold mode — solid cache-poisoning hygiene.
- Stale "(temporary)" workflow registrations (§1) should be disabled in
  the Actions UI; they cannot be removed from git because their files
  are already gone from every branch.

## 8. Documentation and release/distribution

- 69 Markdown files (~7.6k lines): Indonesian user README (thorough),
  contributor index, architecture/output contracts, CI & release,
  troubleshooting, privacy maintenance, manual acceptance, UI/UX
  baselines, `docs/superpowers/` spec/plan/evidence archive, dated
  internal audits. **0 of 376 internal relative links broken.**
- The docs are unusually honest about limits: unsigned EXE, non-atomic
  multi-file publication, "COMPLETE means the confirmed subset",
  synthetic tests ≠ live acceptance. The internal DevOps audit +
  remediation (2026-10-22, in the audited commit) lists five controls
  still needing out-of-repo evidence — Authenticode, first durable
  release, admin alert verification, endpoint ACL/encryption validation,
  live acceptance. **This audit concurs with all five**, and §4 S-1/S-2
  show item three is not merely unverified but currently off.
- Gaps: no `CONTRIBUTING.md` *file* (guidance lives in `docs/` +
  AGENTS.md, which GitHub doesn't surface in the PR flow), no
  `CODE_OF_CONDUCT.md`, no PR template, no `CHANGELOG.md`. Repo has no
  description/topics — discoverability is zero, which may be intentional.
- **Distribution**: PyInstaller `--onefile --windowed --noupx` GUI +
  bundled pinned `uv.exe` provisioning an isolated worker at
  `%LOCALAPPDATA%\SPTDownloader`; Windows 10/11 x64 only, clearly stated.
  First run needs package-index access (intranet proxy/TLS preserved by
  `bootstrap_environment`). But publication stops at "CI artifact with
  14-day retention": no tag has ever been cut, `release-promotion.yml`
  has never published, and the README's download instructions will
  silently rot three weeks after any pause in `main` activity. The
  machinery to fix this is built, tested, and unused.

## 9. Legal and data-protection posture

- GPL-3.0 `LICENSE` present and detected; README and shipped artifact
  carry it. **No per-file license/SPDX headers** (0/45 production
  files) — worth adding so the notice survives files copied out of
  context (forks included).
- GUI states "for personal use only"; AGENTS.md codifies "commit
  synthetic data only", which history confirms. Workbooks/checkpoints
  are documented as confidential, unencrypted, local-only.
- The public repository necessarily publishes intranet hostnames and
  portal paths. That is an existing, presumably accepted exposure; it
  is noted here only so it remains a conscious decision.

## Findings register (prioritized, upstream)

| ID | Sev | Area | Finding |
| --- | --- | --- | --- |
| U-1 | **High (process)** | Disclosure | Private vulnerability reporting disabled while SECURITY.md directs reporters to it; Issues also disabled. External disclosure path is a dead end. (§1, §4 S-1) |
| U-2 | **Medium** | Distribution | No tags/releases; sole download channel expires in 14 days; promotion workflow unused. (§8) |
| U-3 | **Medium** | Governance | 0 required approvals + no CODEOWNERS + self-merge; bot/self comments are the only "review". Automated gates carry the full assurance load, incl. for AI-authored commits. (§1–2) |
| U-4 | Medium | Security ops | Dependabot-alert / secret-scanning toggle state and alert *delivery* unverified (upstream's own remediation leaves this open); bus factor 1. (§4 S-2) |
| U-5 | Low | CI efficiency | 42/100 recent runs failed (candidate-branch iterations, several 19–25 min); 55-commit unmerged candidate branch raises integration cost. (§7) |
| U-6 | Low | Governance hygiene | ≥5 stale "(temporary)" workflows still active in the Actions registry; CodeQL is default-setup, not versioned config. (§1, §7) |
| U-7 | Low | Code quality | No lint/format/type/coverage gates; 53 functions > McCabe 15 (max 61); 7% annotation. Mitigated by test volume. (§3, §6) |
| U-8 | Low | People | Personal government mailbox as sole in-app support contact; maintainer ≠ support contact; no documented backup for either. (§4 S-4) |
| U-9 | Low (known) | Trust | Executable unsigned (Authenticode pending in upstream roadmap). |
| U-10 | Info | Community | Issues disabled; no CONTRIBUTING file/CHANGELOG/CoC/PR template; no description/topics; community profile 50%. |
| U-11 | Info | Portability | `cepeter-job/spt-downloader` hard-coded ×231 incl. shipped binary README; transfer/rename would need a sweep. |
| U-12 | Info | Hygiene | `uv` one patch behind; transitive `pyee` behind; duplicate fixture binaries; `sys.path.insert` in tests; `spt_gui.py:79` late-binding lambda; one historic red post-merge `main` run with no automated escalation. |

## Recommendations (for the upstream maintainer, in order)

1. **Enable private vulnerability reporting now** (Settings → Security),
   then click through SECURITY.md as an outsider to prove the path works.
   Decide whether Issues stay off; if so, make the email path explicit
   as the public alternative. *(Fixes U-1.)*
2. **Verify and record the alert chain**: Dependabot alerts, secret
   scanning + push protection, and an actual test notification; note the
   date/result per SECURITY.md's own checklist. Name a backup human.
   *(U-4, U-8.)*
3. **Cut the first durable release** with `release-promotion.yml`
   (`publish=true` against run 39728441622's verified artifact) and
   point the README at the Release. Start a `CHANGELOG.md` at the same
   tag. *(U-2.)*
4. **Add one independent approval**: a `CODEOWNERS` file plus
   `required_approving_review_count: 1` with a second human (even if
   frequently the support-contact owner), at minimum for
   `spt_desktop/auth.py`, `credentials.py`, `output*.py`, workflows.
   If headcount makes this impossible, document that explicitly as an
   accepted risk in SECURITY.md. *(U-3.)*
5. **Tame WIP mechanics**: disable the stale "(temporary)" workflows in
   the UI; land `feat/tk-accessibility-candidate` in slices or retire
   it; consider `concurrency: cancel-in-progress` for candidate-branch
   push workflows to cut Windows minutes. *(U-5, U-6.)*
6. **Version the CodeQL config** (advanced setup workflow in-repo) so
   queries/languages are reviewable and survive transfer/forking.
   *(U-6.)*
7. **Cheap quality ratchets**: baseline `ruff.toml` (freeze current
   findings, fail on new), coverage *report* artifact in the portable
   job, fix `spt_gui.py:79` (`lambda s=settled: s.set(True)`), dedupe
   fixture workbooks, add SPDX headers. *(U-7, U-12, §9.)*
8. **Escalate red `main` automatically** (failed-workflow notification
   to a watched channel), since required checks only gate PRs. *(U-12.)*

## Methodology and limitations

Performed: full tree + unshallowed history review (212 commits);
secret/PII pattern scans over tree and all added lines in history;
tracked-binary (XLSX) content inspection; dangerous-API greps
(eval/exec/subprocess/TLS/pickle); AST metrics (annotations, docstrings,
function length, McCabe via Ruff 0.17.1); import-reachability vs
`WORKER_FILES`; lockfile↔requirements consistency; PyPI latest-version
comparison (2026-10-22); workflow permission/pin review; Markdown
internal-link validation (376 links); live GitHub API review of upstream
(ruleset detail, PVR status, branches/PRs/reviews/run statistics,
community profile), confirming upstream `main` == audited SHA.

Not performed, by policy or sandbox limits: **no local test execution**
(`AGENTS.md` reserves execution evidence to GitHub runners — run
39728441622 for this exact commit is the execution evidence); WIP-branch
code (`feat/*`) is outside the audited tree and was assessed only for
process risk; CI artifact/log *bytes* could not be downloaded here
(storage host outside egress allowlist), so artifact contents are assessed
from workflow definitions + job results; several admin APIs returned 403
to this integration (secret/code-scanning & Dependabot alert lists,
CodeQL default-setup detail, Actions permissions) — items S-2/U-4 remain
for an admin to close in Settings; no live Coretax access was used or
attempted; endpoint ACL/encryption/restore validation remains, as
upstream's own remediation doc states, an on-site activity.

## Appendix — key metrics

| Metric | Value |
| --- | --- |
| Repo age / commits | 13 days / 212 commits on `main` path |
| PRs | 72 total: 62 merged, 9 closed-unmerged, 1 open draft (#71) |
| Independent human approvals (sampled merged PRs) | 0 (bot + self comments only) |
| Tracked files | 353 |
| Production Python | 57 files / 18,376 LOC |
| Test Python | 202 files / 31,855 LOC; 1,395 test functions (+631 parametrize) |
| Docs | 69 files / ~7,568 lines; 0/376 broken internal links |
| Dependencies | 2 direct runtime; all latest except `uv` (−1 patch), transitive `pyee` |
| Largest modules | `coretax_browser.py` 1,579; `spt_desktop/app.py` 1,013 |
| Max function complexity | `collect_inventory` McCabe 61 |
| TODO/FIXME; secrets (tree+history) | 0; 0 |
| Tags / Releases | 0 / 0 |
| Private vulnerability reporting | **disabled** |
| Recent runs (last 100) | 44 success / 42 failure / 10 cancelled (WIP-branch driven) |
| HEAD CI (`main` @ 44969fc) | run 39728441622, success, 13m06s, 9/9 jobs |
