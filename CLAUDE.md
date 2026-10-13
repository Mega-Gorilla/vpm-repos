# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A VRChat Package Manager (VPM) repo listing for VHub packages, forked from `vrchat-community/template-package-listing`. It has no source code of its own to build or test: CI reads `source.json`, collects every release of the listed packages, and publishes `index.json` plus a landing page to GitHub Pages at https://mega-gorilla.github.io/vpm-repos/. Users add `https://mega-gorilla.github.io/vpm-repos/index.json` to VCC.

The packages themselves live in their own repos (`Mega-Gorilla/KawaPlayer`, `Mega-Gorilla/VhubPlaylistViewer`). Package code changes and releases happen there, not here.

## How the listing is built

`.github/workflows/build-listing.yml` checks out `vrchat-community/package-list-action` (a Nuke/.NET project, default branch, not pinned) into `ci/` and runs its `BuildMultiPackageListing` target with `--list-publish-directory Website`. That target:

1. Reads `source.json` and, for each entry in `githubRepos`, queries the GitHub releases of that repo. A release is picked up only if it has a `package.json` and the package `.zip` as release assets. Entries in `packages` are direct zip URLs instead.
2. Writes `index.json` into `Website/`.
3. Renders `Website/index.html` and `Website/app.js` as **Scriban templates** in place, then the whole `Website/` folder is uploaded as the Pages artifact.

Triggers to keep in mind:
- `push` to `main` runs when `source.json`, anything under `Website/`, or the workflow file changes. Other files (README, CLAUDE.md) don't redeploy.
- `repository_dispatch` lets a package repo trigger a rebuild when it publishes a release. A new version of an already-listed package needs no change here.
- `workflow_dispatch` is for manual runs.

```bash
gh workflow run build-listing.yml
```

```bash
gh run list --workflow build-listing.yml
```

### Rendering locally

The page can't be previewed from the raw `Website/` files, but the action can render it locally (.NET SDK; it targets net6.0/net8.0, so with only a newer runtime set `DOTNET_ROLL_FORWARD=Major`). Clone `vrchat-community/package-list-action` outside this repo, run `dotnet build PackageBuilder/PackageBuilder.csproj -f net8.0`, then:

```bash
PackageBuilder/bin/Debug/PackageBuilder.exe BuildMultiPackageListing --root <action-clone> --package-listing-source-folder <dir with source.json + Website/> --list-publish-directory <empty scratch dir>
```

Outside GitHub Actions the target **wipes `--list-publish-directory` first**, so never point it at `Website/` or the repo. It queries the GitHub API without auth (fine for a couple of repos) and downloads each release zip to hash it. Serve the output dir over HTTP (ES modules don't load from `file://`). Don't commit `ci/` or a generated `index.json`.

## Editing `source.json`

- To add a package, append `"Owner/Repo"` to `githubRepos`. The target repo must already have a release with `package.json` and zip assets.
- `url` must stay `https://mega-gorilla.github.io/vpm-repos/index.json`. It is the address VCC fetches the listing and its updates from. `id` is `com.vhub.vpm`.
- `bannerUrl` is resolved relative to `Website/` (`banner.png`).

## Landing page (`Website/`)

- `index.html` and `app.js` contain Scriban syntax (`{{ }}`, `{{~ ~}}`, `for package in packages`). JS tooling can't parse `app.js` until it's rendered. When you edit, keep the template blocks valid. Scriban built-ins work (e.g. `{{ listingInfo.Url | html.url_encode }}`). Available data: `listingInfo.{Name, Description, Url, Author.{Name,Url,Email}, InfoLink.{Url,Text}, BannerImage, BannerImageUrl}` and per package `{Name, DisplayName, Description, Version, Author.{Name,Url}, Dependencies[].{Name,Version}, Keywords, License, LicenseUrl, Type, ZipUrl}`. Unknown names render as empty strings rather than failing. For example, the template's `package.LicensesUrl` is always empty.
- "Add to VCC" (top and per row) are plain `<a href="vcc://vpm/addRepo?url=...">` links styled like Fluent accent buttons (`.vccAddButton` in `styles.css`), and the manual-add line under the URL bar is plain HTML. Both work even if Fluent or `app.js` fails to load, so keep them out of JS.
- Fluent UI Web Components are pinned to `https://unpkg.com/@fluentui/web-components@2.6.1/dist/web-components.min.js` in both `index.html` and `app.js:1`. The two URLs must stay identical, because a different URL loads a second copy that re-registers the elements. v3 removed `baseLayerLuminance`/`StandardLuminance` and the v2 component APIs, so never use an unversioned URL (that broke the page, see Mega-Gorilla/vpm-repos#3). After changing the version, re-check every handler in `app.js`: Copy, search, help dialog, Package Info, and the ⋯ download menu.
