---
icon: lucide/tag
---

# Naming Conventions


## Versioning

We use different version schemas depending on the repo type:

#### Data release-locked repositories

`platform-webapp`, `platform-api`, `platform-ai-api`, `platform-mcp`, `pipeline`,
`platform-deployment-nextgen`, and other repos that are released in lockstep with
the data release calendar follow a [**CalVer**](https://calver.org/) scheme:

```
YY.MM.n
```

Where:

* `YY.MM` **will always be the release name** (e.g. `26.06` for the June 2026
    release).

!!! warning "The month is fixed!"
    Even if we would release a patch for `26.06` in July 2026, the version would
    still be `26.06.n`, not `26.07.n`.

* `n` will always start at `0` for the first release of a given `YY.MM` and will only
    get incremented when we need to publish a revision of the relevant artifact (e.g.
    a bugfix in the webapp or a revision of the data).


#### Independent repositories

`otter`, `gentropy`, `ontoma`, `clinical-mining`, and similar repos follow a
[**SemVer**](https://semver.org/) scheme:

```
MAJOR.MINOR.PATCH
```

Those are free to evolve independently of the data release calendar, and are not
tied to a specific data release.

## Repositories

### Repo names

- All lowercase.
- Hyphens, not underscores, are used as word separators. (e.g. `platform-api`,
not `platform_api`).
- Avoid prefixes like `ot-` or `opentargets-` in repo names; the org context is
  sufficient.
- Avoid suffixes like `-python` or `-py` in repo names; the language is
  implicit in the package layout.


### Branches

- We don't use **Conventional Commits**, enforced via `commitlint` in pre-commit.
- **No direct commits to `main`** (`no-commit-to-branch` hook); everything
  goes through a PR.
- Branch naming and the full flow live in the KB
  [git workflow](https://opentargets.org/technical-knowledge-base/shared-practices/git-workflow/).

## Repo layout

- **`src/` layout**, package at `src/<name>`, built with **Hatchling**.
- **Tests** run with `pytest`; `coverage` config lives in `pyproject.toml`.
- CLI entry points are declared under `[project.scripts]` (e.g. `otter`,
  `pos`, `p2j`).
- Constant metadata across repos: license `Apache-2.0`, author
  *Open Targets Core Team* `<devs@opentargets.org>`.

## Formatting & linting

Rules live in `pyproject.toml` (`[tool.ruff]`) and `.pre-commit-config.yaml` —
not restated here. The house style is shared verbatim across repos:

| Setting | Value | Source |
|---|---|---|
| Formatter + linter | `ruff` (format + lint) | `[tool.ruff]` |
| Line length | `120` | `[tool.ruff]` |
| Quotes | single inline · double for multiline & docstrings | `[tool.ruff.lint.flake8-quotes]` |
| Docstrings | Google convention | `[tool.ruff.lint.pydocstyle]` |
| Dep hygiene | `deptry` | `[tool.deptry]` |
| Package manager | `uv` | lockfile |

### Running checks

=== "Monorepo (`pipeline`)"

    ```sh
    make dev     # install deps + pre-commit hook
    make lint    # ruff + ty across all packages
    make test    # pytest across all packages
    ```

    Each target takes a package suffix, e.g. `make test-pts`.

=== "Standalone repo"

    ```sh
    uv sync                          # install deps
    uv run pre-commit install        # once
    uv run ruff check && uv run ruff format --check
    uv run pytest
    ```

## Versioning & releases

- **CalVer**: `YY.MM.patch`.
- In `pipeline`, `make pr` computes the next release-candidate version and
  bumps `pyproject.toml`; `make build` tags from the branch + commit SHA and
  triggers the build/publish workflow. Don't hand-edit versions or tags.

## New repo checklist

- [ ] `src/<name>` layout, Hatchling build backend
- [ ] `Apache-2.0`, Core Team author metadata
- [ ] Shared `[tool.ruff]` block copied in
- [ ] `.pre-commit-config.yaml` with `commitlint` + `no-commit-to-branch`
- [ ] `[project.scripts]` entry point if it's a CLI
- [ ] CalVer version string

---

!!! note "Reviewer notes — resolve before publishing (then delete this block)"
    Real divergences found across repos. The page above states one "house"
    answer for each; confirm or decide, since a reference guide that hides the
    split just moves friction downstream.

    1. **Type checker** — `otter`/`pos`/`pipeline` use `ty`; `gentropy` uses `mypy`.
    2. **Test directory** — `otter` uses `test/`; `pos` uses `src/test/`.
    3. **Python floor** — `otter` `>=3.11`, `pos`/`pipeline` `>=3.13` (all target `py313` in ruff).
    4. **Version padding** — `otter` `26.6.1` vs `pos` `26.03.1`. Pick one.
    5. **pre-commit coverage** — present in `gentropy`/`gentroutils`, absent from `otter`/`pos` root. If it's the enforcement mechanism, it should be everywhere.
    6. **Docs toolchain** — `otter` builds docs with Sphinx; the org site is Zensical. Decide whether per-repo Sphinx is intentional or should converge.

    *Optional:* with `pymdownx.snippets` (`url_download = true`) the shared ruff
    block could be pulled straight from a canonical repo instead of copied, so
    it can never drift. Fragile if the source moves — decide if it's worth it.


controlled vocs:

team:
  - open-targets
subteam:
  - data
  - backend
  - frontend
product:
  - platform
  - ppp
tool:
  - pis
  - pts
  - etl
  - etl-backend # phasing out
  - pos
  - orchestrator
  - standalone
environment:
  - development
  - staging
  - production
color:
  - blue
  - green
created_by:
  - terraform
  - orchestrator
