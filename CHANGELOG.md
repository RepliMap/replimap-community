# Changelog

All notable user-facing changes to RepliMap are documented here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).
RepliMap is pre-1.0: minor versions may reshape commands — deprecations are
always announced with zero-breakage aliases first. Full version history is
visible on [PyPI](https://pypi.org/project/replimap/#history).

---

## [0.4.x] - 2026-07

### Added

- **IaC Coverage** — `replimap codify --coverage-state` / `--coverage-state-dir`
  matches your live account against one or more Terraform state files (local
  paths or `s3://bucket/key`) and reports which resources exist in **none** of
  them. Unmanaged resources are triaged into actionable ClickOps, embedded
  (e.g. EBS volumes owned by an instance), and likely auto-created
  (service-linked roles). Per-type counts and coverage percentage are free;
  the full resource list and `coverage.json` are Pro.
- **Offline license verification** — license entitlements are Ed25519-signed
  and verified locally; no phone-home in the scan path.
- **Pre-flight region validation** — a misspelled region now fails immediately
  with did-you-mean suggestions instead of a raw connection error after
  retries.
- **Resolvable error references** — `replimap explain ERR-EC2-403-A7X9`
  summarizes the local error log (exception, command, reproduction) for any
  runtime error reference the CLI prints.
- **`drift --managed-only`** and managed-first sectioned drift output, so
  managed drift isn't buried under unmanaged inventory on large accounts.

### Changed

- **Drift comparison is free on every tier** — `replimap drift` /
  `drift-offline` no longer require a paid plan (it's experimental and not
  part of any tier); both commands print an honest note that
  `terraform plan` is the better answer for "did what I manage change?",
  and point at IaC Coverage for "what exists in NO state?".
- **Documented tiers are now enforced** — `trust-center` (Team),
  regional audit frameworks `apra_cps234`/`rbnz_bs11`/`nzism` (Sovereign),
  and `remediate` (Pro, same gate as `audit --fix`) previously ran ungated;
  they now match the published feature matrix. `scan --trust-center` on
  lower plans warns and continues the scan without auditing — the scan
  itself never fails.
- **TEAM AWS account limit is now unlimited (fair use)** — matching PRO;
  no account-count-based upgrade trigger remains on any paid plan.
- **Upgrade prompts honesty pass** — upsell panels no longer mention
  unshipped or retired features (PDF exports, drift alerts, retention
  tiers); they list only what the plan actually includes.
- **Self-contained graph HTML** — D3 is inlined and all CDN references are
  removed; the interactive graph renders fully air-gapped, with hierarchical
  VPC/subnet containers, viewport-aware fit, and a collapsible filter panel.
- **Codify honesty pass** — generated output is framed as an *adoption
  starting point* (not "production-ready"); the generated `backend.tf` no
  longer injects `default_tags` that would force a diff on every imported
  resource; the generated README lists exactly the files written.
- **Audit artifacts** land in `~/.replimap/reports/<timestamp>/` by default
  instead of the current working directory, and the browser no longer
  auto-opens (pass `--open`).
- **Outbound-surface cleanup** — every code path that contacted an external
  host (other than AWS and license validation) was removed from the package;
  right-sizing suggestions are now always computed locally.
- **Type-namespaced graph storage** — same-named resources of different types
  no longer overwrite each other; existing graph caches are rebuilt
  automatically on the next command.

### Fixed

- **Per-item retry in parallel scanning** — a transient AWS error can no
  longer silently drop a resource from the scan; exhausted items are reported
  in the scan summary and the cache is marked partial.
- **A failed scan can no longer overwrite a good graph cache** with an empty
  one.
- Interactive MFA prompts in non-interactive environments now produce a clear
  "re-authenticate profile X" message instead of a bare `EOFError`.

### Deprecated (zero breakage — old spellings keep working with a notice)

- `--no-cache` → `--no-cred-cache` (it controls the *credential* cache).
- `-v` short flag for `--vpc` on VPC-scoped commands (collided with global
  `--verbose`).
- `graph --security` / `graph --sg-rules` — never had an effect; hidden, warn
  when used, removed in 0.5.0.

---

## [0.3.x] - 2026-01

### Added

- `codify` command — Terraform generation with a 13-stage transformation
  pipeline: lifecycle protection, security-group rule splitting,
  secrets-to-variables extraction, hardcoded-ID-to-reference conversion, and
  Terraform 1.5+ import block generation.
- Interactive D3 dependency graph visualization.
- Local scan caching with `--cache`.

### Fixed

- EC2 `root_block_device` handling that could trigger unintended instance
  replacement.
- AWS system tag (`aws:*`) filtering to prevent provider rejection errors.
- Security-group circular dependencies via automatic rule splitting.

---

## [0.2.x and earlier]

- Core scanners (VPC, Subnet, Security Group, EC2, RDS, S3, and more),
  graph-based dependency resolution, Terraform HCL output, dependency
  tracking, `audit`, `deps`, `cost`, and `iam` commands.
