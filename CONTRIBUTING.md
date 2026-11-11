# Contributing

Wood and Glass is a static site built with [Hugo](https://gohugo.io/) and the
[gallery theme](https://github.com/nicokaiser/hugo-theme-gallery). It lives at <https://woodand.glass>.

## Setup

You need:

- [Git](https://git-scm.com/)
- [mise](https://mise.jdx.dev/) — it installs the pinned Hugo **extended** build and the linters

Clone with submodules. The theme is a git submodule, and the site won't build without it.

```sh
git clone --recurse-submodules <repo-url>
cd wood-and-glass
mise install    # installs tools from mise.toml
mise run setup  # installs the git hooks
```

Already cloned without `--recurse-submodules`?

```sh
git submodule update --init --recursive
```

Hugo extended matters: the theme processes SCSS. `mise.toml` pins `hugo-extended`, so `hugo version` shows `+extended`.

## Run it locally

```sh
mise run serve
```

Open <http://localhost:1313>. It reloads when you save a file.

## Test it

There are no unit tests. The test is that the site builds. CI runs the same check:

```sh
mise run build
```

If it exits clean, you're good. Then look at the pages in the browser. Also run the lint checks (spelling, secrets,
YAML, TOML, trailing whitespace, end-of-file):

```sh
mise run lint
```

Git hooks run the same checks on commit and build the site on push. `mise run check` does both.

## Where things go

| What                 | Where                                       |
| -------------------- | ------------------------------------------- |
| Events list          | `data/events.yaml`                          |
| Wood gallery photos  | `content/wood/` (images next to `index.md`) |
| Glass gallery photos | `content/glass/`                            |
| Home page text       | `content/_index.md`                         |
| Site settings        | `hugo.toml`                                 |
| Templates            | `layouts/` (these override the theme)       |
| Styles               | `assets/css/custom.css`                     |
| Favicon, logo        | `static/`                                   |

Don't edit anything in `themes/gallery`. It's a submodule. Override templates in `layouts/` instead.

To add a photo, drop the image into the right `content/` folder. The gallery picks it up.

## Update events

Events live in [data/events.yaml](./data/events.yaml). Add, edit, or remove entries and open a PR. Each entry looks
like this:

```yaml
- title: Clarke County Recreation Center Show (Day 1)
  date: 2025-11-01T09:00:00-04:00   # start, ISO 8601 with offset
  end: 2025-11-01T16:00:00-04:00    # end, same format
  venue: Clarke County Recreation Center
  url: "https://maps.app.goo.gl/VFvDGctUGcDUSmBD7"   # link, usually a map
  city: Berryville, VA
```

You can edit the file right in GitHub's web UI (pencil icon → propose changes) if you don't want to set up locally.

## Make a change

1. Branch off `main`.
2. Make the change and check it with `mise run serve`.
3. Run `mise run check` (lint plus build).
4. Push and open a PR to `main`.

CI runs `mise run check` on every PR. [CODEOWNERS](./.github/CODEOWNERS) assigns review automatically.

## Deploy

Cloudflare deploys the site. It runs as a Worker named `wood-and-glass`, with static assets.
[wrangler.toml](./wrangler.toml) points it at the `public/` folder, which is Hugo's build output. The Worker is
reachable at two addresses:

- <https://woodand.glass> (production domain)
- a `workers.dev` address, `wood-and-glass.<account-subdomain>.workers.dev` (find the real one in the Cloudflare
  dashboard under the Worker's settings)

Preview deployments get their own address, `<version>-wood-and-glass.<account-subdomain>.workers.dev`. Both
`workers.dev` URLs are public.

**Normal flow:** merge to `main`. GitHub Actions ([build.yml](./.github/workflows/build.yml)) runs `mise run check` on
every push to `main` and on every PR. That job lints and builds. It does not publish. The publish
happens in Cloudflare. Check the Worker's deployments in the dashboard to see what is live.

**Manual deploy** (needs [wrangler](https://developers.cloudflare.com/workers/wrangler/) and a login with access to the
Cloudflare account):

```sh
wrangler login
hugo --minify
wrangler deploy
```

Deploy from an up-to-date `main`, not a feature branch.

`public/` is gitignored. Never commit it.

## Common problems

- **Build fails with a missing theme or layout:** the submodule isn't checked out. Run
  `git submodule update --init --recursive`.
- **`hugo: command not found`, or the wrong Hugo version:** run `mise install`. Don't install Hugo another way, or you
  may get the non-extended build.
- **Images missing or Hugo complains about image processing:** you have the non-extended Hugo. `mise install` gives
  you extended; check `hugo version` shows `+extended`.
- **Git hooks don't run:** run `mise run setup`.
- **Date in `events.yaml` doesn't parse:** use full ISO 8601 with offset, like `2026-04-18T09:00:00-04:00`.
