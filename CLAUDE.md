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
- `push` to `main` runs **only when `source.json` changes** (`paths: source.json`). Changes to `Website/` or the workflow alone do not redeploy. Run the workflow by hand to publish them.
- `repository_dispatch` lets a package repo trigger a rebuild when it publishes a release. A new version of an already-listed package needs no change here.
- `workflow_dispatch` is for manual runs.

```bash
gh workflow run build-listing.yml
```

```bash
gh run list --workflow build-listing.yml
```

You can't build the listing locally without cloning the action repo into `ci/` and having the .NET SDK and a `GITHUB_TOKEN`. In practice, check changes by running the workflow and looking at the deployed page. `ci/` and the generated `Website/index.json` are build output and must not be committed.

## Editing `source.json`

- To add a package, append `"Owner/Repo"` to `githubRepos`. The target repo must already have a release with `package.json` and zip assets.
- `url` must stay `https://mega-gorilla.github.io/vpm-repos/index.json`. It is the address VCC fetches the listing and its updates from. `id` is `com.vhub.vpm`.
- `bannerUrl` is resolved relative to `Website/` (`banner.png`).

## Landing page (`Website/`)

- `index.html` and `app.js` contain Scriban syntax (`{{ }}`, `{{~ ~}}`, `for package in packages`). Opening them directly in a browser shows raw tags, and JS tooling can't parse `app.js` as-is. When you edit, keep the template blocks valid. Available data: `listingInfo.{Name, Description, Url, Author.{Name,Url,Email}, InfoLink.{Url,Text}, BannerImage, BannerImageUrl}` and per package `{Name, DisplayName, Description, Version, Author, Dependencies, Keywords, License, LicensesUrl, Type, ZipUrl}`.
- `app.js` builds a `PACKAGES` map from the template. The "Add to VCC" buttons open `vcc://vpm/addRepo?url=<encoded LISTING_URL>`.
- Fluent UI Web Components load from unpkg **without a version** in both `index.html` (script tag) and `app.js:1` (`import { baseLayerLuminance, StandardLuminance }`). The unversioned URL now resolves to v3, which drops those exports. As a result `app.js` fails at import time and no button handlers get registered (open issue Mega-Gorilla/vpm-repos#3). Any fix must pin both URLs to the same version (v2 still has the exports) and then re-check every handler in `app.js`: Add to VCC, Copy, search, help dialog, and Package Info.
