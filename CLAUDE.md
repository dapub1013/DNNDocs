# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

DNNDocs is the documentation site for the DNN Platform (formerly DotNetNuke) CMS, published at https://docs.dnncommunity.org/. It uses **DocFX** to combine XML comments from DNN NuGet packages with Markdown articles. The build system uses **NUKE** (a C# build automation tool).

## Build & Run

Prerequisites: .NET SDK 9.0 (see `global.json`), .NET Framework 4.6.2 Developer Pack.

```bash
# Mac/Linux - build and serve locally (default target)
./build.sh

# Windows PowerShell
.\build.ps1

# Windows Command Prompt
build.cmd
```

The default build target is `Serve`, which builds and serves on http://localhost:8080.

### Explicit build targets (pass as argument):

```bash
./build.sh Compile    # Full build without serving
./build.sh Clean      # Clean output directories
./build.sh BuildPlugins  # Build only the DocFX plugins
```

### GitHub Actions tokens (optional)

For contributor/stats features to work locally, create a `.env` file in the repo root:
```
GithubToken=github_pat_xxxxxxxxxx
```

Without this, `SKIP_CONTRIBUTORS=true` is set automatically for non-deploy builds.

## Architecture

### Build Pipeline (NUKE)

`build/Build.cs` defines the build targets in dependency order:
1. **Clean** → clears `docs/`, plugin output, `Dnn.Platform/`
2. **Restore** → NuGet restore for all projects
3. **PullDnnPackages** → downloads DNN NuGet packages into `packages/` (DotNetNuke.Core, etc.)
4. **BuildPlugins** → compiles `plugins/DNNCommunity.DNNDocs.Plugins` and outputs to `templates/dnn-docs/plugins/`
5. **Compile** → runs `docfx metadata` (extracts API docs from DLLs) then `docfx build`
6. **Serve** (default) → runs `docfx --serve --open-browser`
7. **Deploy** → CI-only; pushes compiled `docs/` output to the `site` branch on GitHub Pages

### DocFX Configuration (`docfx.json`)

- **metadata**: Extracts API docs from DLL files in `packages/` → output to `api/`
- **build**: Combines `api/` (generated YAML), `content/` (Markdown), and `apidoc/` (manual API overrides)
- **templates**: Uses `default` + `templates/dnn-docs` (custom template)
- **output**: `docs/` directory
- **overwrite**: `apidoc/**.md` files can override generated API documentation

### Content Structure

```
content/
  getting-started/   # Setup, installation, upgrades, development fundamentals, contribution guides
  features/          # DNN features (content management, extensibility, security, performance)
  reference/         # Assembly/NuGet package reference docs
  labs/              # Experimental content
  tutorials/         # Step-by-step guides
apidoc/              # Markdown overrides for generated API documentation
```

Navigation is defined in `toc.md` (top-level) and `content/**/toc.md` files using `xref:` links to article UIDs.

### Markdown Article Front Matter

Every article should have YAML front matter:
```yaml
---
uid: unique-identifier-here
locale: en
title: Article Title
dnnversion: 09.02.00
related-topics: other-uid, another-uid
---
```

The `uid` must be unique across all articles. The `related-topics` field is a comma-separated list of UIDs that the `MoreLinksBuildStep` plugin uses to generate related-topic links.

### Custom DocFX Plugins (`plugins/DNNCommunity.DNNDocs.Plugins/`)

These implement `IDocumentBuildStep` and run during the DocFX build:

- **MoreLinksBuildStep** (order 50): Processes `related-topics` and `links` front matter fields, resolving UIDs to URLs and injecting HTML link divs into article content.
- **PageStatsBuildStep**: Injects page statistics into article content.
- **RepoStatsBuildStep** (order 2): Calls the GitHub API (via `Octokit`) to fetch contributor and commit data, injecting contributor metadata into article content for display in the template.

Plugins are built to `templates/dnn-docs/plugins/` before DocFX runs.

### Custom Template (`templates/dnn-docs/`)

Mustache/Liquid templates and partials that extend the DocFX default template. Key files:
- `conceptual.html.primary.tmpl` / `.js` — primary article rendering
- `partials/` — navbar, footer, breadcrumb, head, sidebar overrides
- `styles/dnndocs.css` — custom CSS
