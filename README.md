# deploy-nsite-to-pages

A reusable GitHub Action that mirrors a Nostr [nsite](https://nsyte.run/)
to GitHub Pages. Pulls the site from the publisher's relays + Blossom
servers and publishes it via `actions/deploy-pages`.

Works for both root nsites (kind 15128) and named nsites (kind 35128).

## Usage

```yaml
name: Mirror nsite to Pages

on:
  schedule: [{ cron: "0 */6 * * *" }]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: true

jobs:
  mirror:
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.mirror.outputs.page_url }}
    steps:
      - uses: actions/checkout@v4
      - id: mirror
        uses: Origami74/deploy-nsite-to-pages@v1
        with:
          pubkey: npub1y0gja7r4re0wyelmvdqa03qmjs62rwvcd8szzt4nf4t2hd43969qj000ly
          # name: my-named-site                          # optional, kind 35128
          # servers: https://blossom.example.com         # optional override
          # relays: wss://relay.example.com              # optional override
```

A more complete example is in [`examples/deploy.yml`](examples/deploy.yml).

## Inputs

| input        | required | default | description                                                          |
| ------------ | -------- | ------- | -------------------------------------------------------------------- |
| `pubkey`     | yes      | —       | npub, hex pubkey, or NIP-05 (`name@host`) of the nsite publisher     |
| `name`       | no       | (root)  | name of a *named* nsite (kind 35128); blank → root site (kind 15128) |
| `output-dir` | no       | `site`  | local directory the nsite is downloaded into and uploaded from       |
| `relays`     | no       | (auto)  | comma-separated relay overrides for the download                     |
| `servers`    | no       | (auto)  | comma-separated Blossom server overrides for the download            |

`relays` and `servers` are usually only needed when the publisher hasn't
gossiped a NIP-65 relay list (kind 10002) or a Blossom server list
(kind 10063) yet.

## Outputs

| output       | description                                                   |
| ------------ | ------------------------------------------------------------- |
| `page_url`   | URL of the published Pages site                               |
| `file_count` | number of files downloaded from the nsite                     |

## Requirements on the calling workflow

- A runner with `curl` + `bash` (any `ubuntu-latest` works)
- `pages: write` and `id-token: write` permissions
- The `github-pages` environment on the deploying job
- An `actions/checkout@v4` step before calling the action

## What the action does

1. Installs the `nsyte` CLI from the official `nsyte.run/get/install.sh`.
2. Verifies it's on PATH.
3. Runs `nsyte download -p <pubkey> [-d <name>] [-r <relays>] [-s <servers>]
   -o <output-dir> --overwrite`.
4. Refuses to publish an empty download (so a relay glitch can't blank the
   existing mirror).
5. `actions/configure-pages` → `actions/upload-pages-artifact` →
   `actions/deploy-pages`.

## Releases

Pin to a major version tag (e.g. `@v1`) for stability, or `@main` to track
the latest. Major version bumps signal breaking input/output changes.

## License

MIT — see [LICENSE](LICENSE).
