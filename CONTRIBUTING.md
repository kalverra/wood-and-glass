# Contributing

Wood and Glass is a static site built with [Hugo](https://gohugo.io/) and the
[gallery theme](https://github.com/nicokaiser/hugo-theme-gallery). It lives at <https://woodand.glass>.

## Setup

You need:

- [Git](https://git-scm.com/)
- [Hugo **extended**](https://gohugo.io/installation/) (the theme needs image processing). Check with `hugo version`;
  it should say `+extended`.
- [pre-commit](https://pre-commit.com/) (optional, but it runs the same style checks as review)

Clone with submodules. The theme is a git submodule, and the site won't build without it.

```sh
git clone --recurse-submodules <repo-url>
cd wood-and-glass
pre-commit install
```

Already cloned without `--recurse-submodules`?

```sh
git submodule update --init --recursive
```

## Run it locally

```sh
hugo server -D
```

Open <http://localhost:1313>. It reloads when you save a file.

## Test it

There are no unit tests. The test is that the site builds. CI runs the same command:

```sh
hugo --minify
```

If it exits clean, you're good. Then look at the pages in the browser. Also run the pre-commit checks (YAML, TOML,
trailing whitespace, end-of-file):

```sh
pre-commit run --all-files
```

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
2. Make the change and check it with `hugo server -D`.
3. Run `hugo --minify` and `pre-commit run --all-files`.
4. Push and open a PR to `main`.

CI builds every PR. [CODEOWNERS](./.github/CODEOWNERS) assigns review automatically.

## Deploy

Cloudflare deploys the site. It runs as a Worker named `wood-and-glass`, with static assets.
[wrangler.toml](./wrangler.toml) points it at the `public/` folder, which is Hugo's build output. The Worker is
reachable at two addresses:

- <https://woodand.glass> (production domain)
- a `workers.dev` address, `wood-and-glass.<account-subdomain>.workers.dev` (find the real one in the Cloudflare
  dashboard under the Worker's settings)

Preview deployments get their own address, `<version>-wood-and-glass.<account-subdomain>.workers.dev`. Both
`workers.dev` URLs are public.

**Normal flow:** merge to `main`. GitHub Actions ([build.yml](./.github/workflows/build.yml)) runs `hugo --minify` on
every push to `main` and on every PR. That job only checks that the site builds. It does not publish. The publish
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
- **Images missing or Hugo complains about image processing:** you have the non-extended Hugo. Install the extended
  build.
- **Date in `events.yaml` doesn't parse:** use full ISO 8601 with offset, like `2026-04-18T09:00:00-04:00`.
