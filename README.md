# writs

Writs is an embeddable scripting language for TTRPGs.

## Install

```sh
# macOS / Linux
curl --proto '=https' --tlsv1.2 -LsSf https://github.com/mrbandler/writs/releases/latest/download/writs-cli-installer.sh | sh
```

```powershell
# Windows
powershell -ExecutionPolicy Bypass -c "irm https://github.com/mrbandler/writs/releases/latest/download/writs-cli-installer.ps1 | iex"
```

```sh
# Homebrew (macOS / Linux)
brew install mrbandler/tap/writs
```

Windows installer (MSI): [writs-cli-x86_64-pc-windows-msvc.msi](https://github.com/mrbandler/writs/releases/latest/download/writs-cli-x86_64-pc-windows-msvc.msi)

```sh
# Nix: try it without installing, or install it
nix run github:mrbandler/writs -- --help
nix profile install github:mrbandler/writs
```

```sh
# Prebuilt binary via cargo-binstall
cargo binstall --git https://github.com/mrbandler/writs writs-cli
```

## Development

With Nix: `direnv allow` (or `devenv shell`). Everything, including the git hooks, is set up.

Without Nix (macOS, Linux, Windows): install [rustup](https://rustup.rs),
[just](https://github.com/casey/just#installation) and
[cargo-binstall](https://github.com/cargo-bins/cargo-binstall#installation), then run `just setup`.

`just` lists all tasks; `just check` runs everything CI runs.

## Releasing

Releases are automated with [release-plz](https://release-plz.dev):

1. Every PR is squash-merged, so each PR becomes one commit on `main` whose message is the PR
   title. Titles must be [conventional commits](https://www.conventionalcommits.org) (checked
   by the `PR Title` workflow): `fix:` bumps the patch version, `feat:` the minor, `feat!:` the
   major.
2. After every merge, release-plz updates a single open "chore: release" PR with the version
   bumps and the new `CHANGELOG.md` entries. Don't edit it; it collects changes until you are
   ready to release.
3. Merging that PR publishes the crates to crates.io and pushes `<crate>-v<version>` tags.
4. The `writs-cli-v<version>` tag triggers [dist](https://opensource.axo.dev/cargo-dist/),
   which builds the `writs` command, the installers and the GitHub Release, and updates the
   Homebrew formula. The CLI crate (`writs-cli`) itself is not published to crates.io.

### One-time setup per repository

Until this is done, the release-plz and docs workflow runs fail.

1. **GitHub App** (create once per account, install on each repo). Actions run with the
   built-in `GITHUB_TOKEN`, and GitHub deliberately does not start other workflows from events
   that token causes. With it, CI would never run on the release PR (so the required checks
   never pass and it can't be merged) and the release tag would never trigger dist. A token
   from your own App does trigger workflows.
   - Settings → Developer settings → GitHub Apps → New GitHub App. No webhook needed.
     Repository permissions: *Contents* and *Pull requests* read/write.
   - Generate a private key, then install the App on this repo.
   - Add the secrets: `gh secret set APP_CLIENT_ID` (the App's Client ID) and
     `gh secret set APP_PRIVATE_KEY < key.pem`.

   A fine-grained personal access token works too, but it acts as you and expires; the App
   only has the permissions you gave it, on the repos it is installed on.
2. **First publish by hand**: `cargo publish -p writs`. crates.io only lets you
   configure a trusted publisher for a crate that already exists.
3. **Trusted publisher** on crates.io (crate → Settings → Trusted Publishing) for each crate:
   this repo, workflow `release-plz.yml`, environment `release`. The release job then gets a
   short-lived publish token through GitHub's OIDC, and no crates.io token is stored anywhere.
   The environment pins publishing to the one job that declares `environment: release`;
   GitHub creates it on the first run. Add protection rules to it (e.g. only `main` may deploy)
   under Settings → Environments if you want.
4. **Repository settings**: `just gh-setup` applies the ruleset for `main` (PR-only, squash,
   required checks), squash-only merges with the PR title as commit message, and GitHub Pages.
   GitHub does not copy these from the template. Skip it if you already ran it.
5. **Homebrew**: create the public repo `mrbandler/homebrew-tap` (once per account, shared by all
   projects), create a fine-grained token with *Contents* read/write on it, and add it as a
   secret: `gh secret set HOMEBREW_TAP_TOKEN`. dist pushes the formula on every release.

## License

Licensed under `MIT`. See the `LICENSE*` files.
