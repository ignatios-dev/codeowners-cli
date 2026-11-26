# codeowners-cli

Small terminal tool: which GitHub orgs do I have, who are their code owners, which owner has which repos, which repo has which owners.

Needs `gh` (logged in, scope `repo` + `read:org`) and Python 3. No other dependencies, no LLM, no token cost.

```
codeowners orgs               # organizations + cache state
codeowners sync               # (re)load CODEOWNERS of all orgs (~2 min for ~1000 repos)
codeowners owners             # code owners per organization, with repo count
codeowners by-owner [NAME]    # repos per owner (NAME = substring filter)
codeowners by-repo [NAME]     # owners per repo (NAME = substring filter)
codeowners missing            # repos without a code owner
```

Common flags: `--org ORG` (repeatable), `--refresh`, `--include-archived`, `--json`.

The first query per org loads from GitHub and caches in `~/.cache/codeowners-cli/`; later queries are instant. Use `sync` or `--refresh` to update.
Lookup order per repo follows GitHub: `.github/CODEOWNERS`, `CODEOWNERS`, `docs/CODEOWNERS` on the default branch.

Install: `ln -s ~/Development/codeowners-cli/codeowners /opt/homebrew/bin/codeowners`
