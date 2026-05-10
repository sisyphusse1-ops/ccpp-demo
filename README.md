# ccpp-demo

**Live demo of [claude-code-pro-pack](https://github.com/sisyphusse1-ops/claude-code-pro-pack) + [cc-audit](https://github.com/sisyphusse1-ops/cc-audit) running as GitHub Actions.**

Every commit that touches `CLAUDE.md` or `AGENTS.md` gets automatically linted. Leaked API keys fail the build. Compliance scores land in the PR summary.

## What this repo proves

Two composite actions, both `uses: ...@v1`, ship end-to-end:

- **`sisyphusse1-ops/claude-code-pro-pack@v1`** — installs the 12-rule baseline (`CLAUDE.md` + `AGENTS.md`) into your repo
- **`sisyphusse1-ops/cc-audit@v1`** — lints the baseline on every PR, fails the build on leaked secrets

## Real runs

Check the [Actions tab](https://github.com/sisyphusse1-ops/ccpp-demo/actions) — both workflows have completed successfully:

- ✅ **Install pro-pack** → [run 25642550464](https://github.com/sisyphusse1-ops/ccpp-demo/actions/runs/25642550464) — copied `CLAUDE.md` and `AGENTS.md` into the repo
- ✅ **cc-audit** → [run 25642562320](https://github.com/sisyphusse1-ops/ccpp-demo/actions/runs/25642562320) — scored the baseline against itself

Click into either run for the full summary.

## The workflow files

**`.github/workflows/install-pack.yml`** — one-time install (triggered manually via `workflow_dispatch`):

```yaml
name: Install pro-pack
on: workflow_dispatch
jobs:
  install:
    runs-on: ubuntu-latest
    permissions: { contents: write }
    steps:
      - uses: actions/checkout@v4
      - uses: sisyphusse1-ops/claude-code-pro-pack@v1
        with:
          flavor: both
      - name: Commit
        run: |
          git config user.email 'bot@users.noreply.github.com'
          git config user.name 'pro-pack-installer'
          git add -A
          git diff --cached --quiet || git commit -m "chore: install claude-code-pro-pack v1"
          git push
```

**`.github/workflows/audit.yml`** — runs on every PR/push touching the behavior files:

```yaml
name: cc-audit
on:
  pull_request:
    paths: [ 'CLAUDE.md', 'AGENTS.md' ]
  push:
    paths: [ 'CLAUDE.md', 'AGENTS.md' ]
  workflow_dispatch:
jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: sisyphusse1-ops/cc-audit@v1
```

## Try it yourself

Fork this repo. Open Actions → "Install pro-pack" → Run workflow. Watch it commit `CLAUDE.md` + `AGENTS.md` to your fork. Next push to either of those files triggers the audit automatically.

Or just drop the two workflow files into any existing repo you have.

## Why bother

- **Prevent drift** — CLAUDE.md compliance decays as people casually edit it
- **Catch leaks** — cc-audit fails the build on `sk-...`, `ghp_...`, `AKIA...`, live postgres URLs
- **Onboard faster** — new contributors see the 12-rule baseline + its coverage in PR summaries

Full writeup: [dev.to post](https://dev.to/sisyphusse1ops/i-shipped-cc-audit-as-a-github-action-now-your-claudemd-gets-linted-on-every-pr-5fal)

Data from scanning 492 real CLAUDE.md files (median compliance 3/12): [dev.to post](https://dev.to/sisyphusse1ops/i-scored-92-public-claudemd-files-against-a-12-rule-baseline-median-score-512-2971)

## License

MIT on all three repos.
