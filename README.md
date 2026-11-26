# codeowners-cli

A small terminal tool that answers four questions about your GitHub organizations:

- Which organizations am I in?
- Who are the code owners of each organization?
- Which repos does a given code owner have?
- Which code owners does a given repo have?

It reads `CODEOWNERS` files through the GitHub CLI, caches the parsed result locally and answers from the cache. It is read-only: it never writes to GitHub. No other dependencies, no LLM, no API costs.

## Requirements

- [GitHub CLI](https://cli.github.com) (`gh`), logged in with `gh auth login`, token scopes `repo` and `read:org`
- Python 3.9 or newer

The tool works with whichever account `gh` is logged in with. It sees the organizations of that account and the repos that account may read.

## Install

```sh
git clone <this repo> ~/codeowners-cli
ln -s ~/codeowners-cli/codeowners /usr/local/bin/codeowners   # or any directory on your PATH
```

## Interactive view

Run `codeowners` without arguments for a three-pane terminal view: organizations on the left, owners or repos in the middle, details of the selected entry on the right. It needs no extra packages (Python's built-in `curses`).

```
 codeowners  ·  all orgs  ·  [Owners] Repos
 Organizations   │ Owners (3)              │ Details
▸ All orgs     3 │▸ @acme/platform       2 │ @acme/platform
  acme         3 │  @acme/backend        2 │ 2 repos in 1 org
                 │  @alice               1 │ default owner (*) of 1
                 │                         │
                 │                         │ acme  (2)
                 │                         │   api   *
                 │                         │   web   /src/, *.md
```

| Key | Action |
|---|---|
| `j` `k` / arrows, `PgUp` `PgDn`, `g` `G` | move |
| `Tab`, `h` `l` / left right, `Enter` | switch pane |
| `/` | filter by text, `Enter` keeps it, `Esc` clears it |
| `m` | switch between Owners and Repos view |
| `u` | Repos view: only repos without a code owner |
| `a` | show or hide archived repos |
| `o` | open the selected owner or repo on GitHub in your browser |
| `r` | reload from GitHub (selected org, or all after a confirmation) |
| `?` / `q` | help / quit |

Repos without a code owner are shown in yellow. In the Repos view the details list the CODEOWNERS file, all owners and every rule. In the Owners view they list the owner's repos per organization and the patterns the owner covers.

## Usage

```
codeowners                    interactive view (same as `codeowners tui`)
codeowners orgs               organizations you belong to, with cache state
codeowners sync               (re)load CODEOWNERS of all organizations into the cache
codeowners owners             code owners per organization, with repo count
codeowners by-owner [NAME]    repos per code owner (NAME filters by substring)
codeowners by-repo [NAME]     code owners per repo (NAME filters by substring)
codeowners missing            repos without a code owner
```

Options for all commands:

| Option | Meaning |
|---|---|
| `--org ORG` | Limit to this organization. Repeatable. Any organization works, also ones you are not a member of (public repos only). |
| `--refresh` | Reload from GitHub instead of using the cache. |
| `--include-archived` | Include archived repos (hidden by default). |
| `--json` | Machine-readable output. |

### Example

```
$ codeowners owners --org acme

acme  (3 repos, 3 owners)
  OWNER            REPOS
  @acme/backend    2
  @acme/platform   2
  @alice           1
```

```
$ codeowners by-repo
REPO              OWNERS
acme/api          @acme/backend, @acme/platform
acme/web          @acme/platform, @alice
acme/legacy-tool  (no code owner)
```

## How it works

- Per organization one paginated GraphQL query (30 repos per page) returns the repo list and the CODEOWNERS candidates.
- Per repo the first existing file wins, in the order GitHub uses: `.github/CODEOWNERS`, `CODEOWNERS`, `docs/CODEOWNERS`, read from the default branch.
- Comments are stripped. Owners can be `@user`, `@org/team` or an email address. Owners are compared case-insensitively.
- A repo's owners are the union of all owners over all rules in its file, not only the `*` rule.
- The cache lives in `~/.cache/codeowners-cli/` (or `$XDG_CACHE_HOME/codeowners-cli/`), one JSON file per organization. A first sync of about 1000 repos takes around two minutes, later queries are instant. Delete the folder to start fresh.

## Limits

- Only organizations, not personal repos.
- If an organization enforces SAML SSO, the token must be authorized for it (`gh auth refresh`, then authorize in the browser). Otherwise the organization can show up with 0 repos.
- Repos you cannot read are missing from the results without an error.
- GitHub Enterprise Server is not tested.

## License

MIT, see [LICENSE](LICENSE).
