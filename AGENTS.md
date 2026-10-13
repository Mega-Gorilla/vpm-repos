# Repository Guidelines

## Project Structure & Module Organization

This repository publishes the VHub VPM listing. Package implementations live in `Mega-Gorilla/KawaPlayer` and `Mega-Gorilla/VhubPlaylistViewer`.

- `source.json`: listing metadata and package repository references.
- `Website/`: Scriban templates (`index.html`, `app.js`), CSS, banner, and favicon.
- `.github/workflows/build-listing.yml`: release collection, listing generation, and GitHub Pages deployment.
- `CLAUDE.md`: detailed rendering and maintenance instructions.

There is no application build manifest or test directory here.

## Build, Test, and Development Commands

Clone `vrchat-community/package-list-action` outside this repository. From that checkout, build its generator with the .NET SDK:

```powershell
dotnet build PackageBuilder/PackageBuilder.csproj -f net8.0
./PackageBuilder/bin/Debug/PackageBuilder.exe BuildMultiPackageListing --root "<action-clone>" --package-listing-source-folder "<repo>" --list-publish-directory "<scratch>"
```

The generator downloads release assets and renders the listing. **Local generation deletes the output directory first**; use a dedicated scratch directory outside the repository. With only a newer .NET runtime, set `$env:DOTNET_ROLL_FORWARD = 'Major'`.

Serve the generated output over HTTP:

```powershell
python -m http.server 8000 --directory "<scratch>"
```

From this repository, `gh run list --workflow build-listing.yml` checks CI status; `gh workflow run build-listing.yml` triggers publication from the default branch.

## Coding Style & Naming Conventions

Match existing formatting: four spaces in JSON; two spaces in HTML, JavaScript, CSS, and YAML. Use camelCase JavaScript variables and descriptive CSS classes. Preserve Scriban blocks and existing filenames. No formatter or linter is configured; raw `app.js` cannot be parsed as ordinary JavaScript before rendering.

## Testing Guidelines

No automated test framework or coverage threshold is configured. Validate configuration with `Get-Content -Raw source.json | ConvertFrom-Json`, then generate the listing. Check package versions and browser console errors. For UI changes, verify Copy, search, help, Package Info, downloads, light/dark themes, and VCC links. Confirm manual-add instructions remain usable when JavaScript fails. Record actual verification and any untested behavior.

## Commit & Pull Request Guidelines

Follow the existing `feat:`, `fix:`, `docs:`, and `chore:` commit prefixes. Use concise descriptions. PRs should explain the change, link relevant issues, report validation and remaining checks, and include screenshots for visual changes. Relevant changes merged into `main` trigger deployment; documentation-only changes do not.

## Configuration Safeguards

Keep listing ID `com.vhub.vpm` and URL `https://mega-gorilla.github.io/vpm-repos/index.json` stable. New `githubRepos` entries require releases with a ZIP asset that has `package.json` at its root. Keep Fluent UI pinned to the same `2.6.1` URL in both templates and retain plain HTML VCC links. Exclude generated `index.json` and `ci/` from commits.
