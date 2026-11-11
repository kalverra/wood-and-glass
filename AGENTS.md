# AGENTS.md

Static portfolio site for a wood and glass business. Hugo + the `gallery` theme (git submodule). Served at
<https://woodand.glass> from Cloudflare. For the human version, see [CONTRIBUTING.md](./CONTRIBUTING.md).

## Commands

```sh
mise install                              # once; pinned Hugo extended + linters
mise run setup                            # once; installs the git hooks
git submodule update --init --recursive   # once; the theme is a submodule
mise run serve                            # dev server on http://localhost:1313
mise run build                            # the build. This is the test. CI runs it.
mise run lint                             # typos, secrets, yaml/toml, whitespace/EOF
```

Use Hugo **extended** (`hugo version` must show `+extended`). `mise.toml` pins `hugo-extended`, so `hugo` on PATH is
the right build.

To build without touching the repo, write elsewhere: `hugo --minify --destination "$TMPDIR/out"`.

## Verify your change

There are no unit tests. Before you say a change works:

1. `mise run build` exits 0 with no errors or warnings you caused.
2. `mise run lint` passes.
3. For layout, CSS, or content changes, look at the rendered page (`mise run serve`), not just the build.

## Layout

- `data/events.yaml` — events list. Dates are ISO 8601 with offset (`2026-04-18T09:00:00-04:00`). Rendered by
  `layouts/partials/events.html`.
- `content/wood/`, `content/glass/` — galleries. Images sit beside each `index.md`.
- `content/_index.md` — home page.
- `layouts/` — template overrides. Change the site here.
- `assets/css/custom.css` — styles.
- `hugo.toml` — site config. `wrangler.toml` — Cloudflare config.
- `mise.toml` — pinned tool versions and the tasks above.
- `lefthook.yml` — git hooks. Installed by `mise run setup`.
- `themes/gallery` — submodule. **Do not edit.** Override in `layouts/` instead.
- `public/`, `resources/` — build output. Gitignored or generated. Don't commit or hand-edit.

## Deploy

- CI (`.github/workflows/build.yml`) runs `hugo --minify` on PRs and on pushes to `main`. It builds only. It does not
  deploy.
- Deploys are handled by a Cloudflare Worker named `wood-and-glass`, with static assets (`wrangler.toml` serves
  `./public`). It is served at `woodand.glass` (the custom domain) and at `wood-and-glass.<account-subdomain>.workers.dev`.
  Previews use `<version>-wood-and-glass.<account-subdomain>.workers.dev`. The account subdomain is private. Don't
  write it into files in this public repo.
- The publish step lives in Cloudflare, not in this repo. Don't add a deploy step to CI unless asked.
- Manual deploy: `hugo --minify && wrangler deploy`. **Don't run this yourself.** It publishes to production and needs
  the owner's Cloudflare login. Tell the user the command and let them run it.

## Rules

- Work on a branch and open a PR to `main`. Don't push to `main`.
- Don't commit `public/`, `resources/`, `.DS_Store`, machine-local config like `lefthook-local.yml`, or secrets.
- Events edits are the most common change. Keep the YAML shape of existing entries. Don't invent dates or venues; ask.
- Keep photos web-sized. Hugo resizes them, but huge originals slow builds.
- `.gitmodules` lists `themes/hugo-myportfolio-theme`, but the site uses `gallery`. The extra entry is stale. Ignore it
  unless asked to clean up.
