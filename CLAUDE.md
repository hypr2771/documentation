# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is the Datadog Documentation repository, built using Hugo (static site generator) and published to docs.datadoghq.com. The codebase consists of markdown content, Hugo themes, Node.js build scripts, and Python automation tools.

## Development Commands

### Build and Development
- `yarn start` or `make start` - Full build including external dependencies and run development server
- `yarn run start` - Basic Hugo development server (port 1313)
- `make start-no-pre-build` - Skip external dependencies, build and run
- `make start-preserve-build` - Keep existing build scripts for local testing
- `make start-docker` - Build and run via Docker container
- `make start-sources` - Run site with websites_sources_data (integrations previews)

### Build Commands
- `yarn build` or `make build` - Production build
- `yarn run build:hugo` - Run Hugo build only
- `yarn run build:preview` - Build with preview environment
- `yarn run build:hugo:live` - Build for live/production deployment
- `make build-cdocs` - Compile .mdoc.md files to HTML
- `make watch-cdocs` - Compile .mdoc.md files and watch for changes
- `make build-llms-txt` - Build llms.txt documentation

### Testing and Quality
- `yarn run jest-test` - Run JavaScript tests
- `make clean` - Clean generated files
- `make clean-all` - Clean everything (environment, repos, generated files)

### Dependencies
- `yarn install` - Install Node.js dependencies
- `make dependencies` - Install all dependencies (Node + Python + external repos)
- `make hugpython` - Set up Python virtual environment

### Utilities
- `make find-int int=<integration-name>` - Find source location of an integration

## Architecture

### Content Structure
- `content/en/` - English documentation (primary language)
- `content/{fr,es,ja,ko}/` - Translated content (managed externally)
- `layouts/` - Hugo templates and shortcodes
- `static/` - Static assets, images, fonts
- `assets/` - SCSS stylesheets and JavaScript

### Build System
- **Hugo**: Static site generator with custom themes
- **Node.js**: Package management, build automation, asset processing
- **Python**: Content processing, integration data fetching, translations
- **Makefile**: Orchestrates complex build processes and external dependencies

### External Dependencies
The build system automatically fetches:
- API client examples from multiple language repositories
- Integration metadata and documentation
- Vector integration data via CUE files
- Some documentation is sourced from GitHub using the `pull_config.yaml` file at `local/bin/py/build/configurations/pull_config.yaml`.
- Some documentation is sourced from a go module called `websites-sources`

### Configuration
- `config/` - Hugo configuration for different environments (development, preview, live)
- `package.json` - Node.js dependencies and scripts
- `Makefile` + `Makefile.config` - Build orchestration (copy `Makefile.config.example` to `Makefile.config` for local setup)
- Environment-specific parameters in `config/{env}/params.yaml`
- `go.mod` - Hugo modules configuration (currently using `github.com/DataDog/websites-modules`)

## Branch and PR Guidelines

### Branch Naming
CRITICAL: Always use format `<name>/<description>` with forward slash. Without this:
- GitLab pipeline won't run
- No branch preview will be generated
- CI will fail

### Content Guidelines
Follow the CONTRIBUTING.md style guide:
- Use American English (en_US)
- Be plain, direct, and concise
- Provide explicit instructions with examples
- Use imperative voice for instructions
- Don't edit translated content directly (managed externally)

## Hugo-Specific Details

### Shortcodes
Extensive library in `layouts/shortcodes/` for:
- Code examples across multiple languages (`programming-lang`, `programming-lang-wrapper`)
- Integration-specific content (`integrations`, `integration-items`)
- API documentation (`apicode`, `apicontent`, `api-scopes`)
- Security and compliance sections
- UI components (`tabs`, `tab`, `collapse`, `callout`, `code-block`)
- Multi-region content (`site-region`, `region-param`)

### Content Types
- **Regular markdown pages** - Standard `.md` files with frontmatter
- **CDOCS files** - `.mdoc.md` files that get compiled to HTML before Hugo build (see `make build-cdocs`)
- **API reference** - Auto-generated from OpenAPI specs via `yarn run build:apiPages`
- **Integration pages** - Auto-populated from external repos and `websites-sources` module
- **Multi-language code examples** - Sourced from datadog-api-client-* repositories

## Development Notes

### Local Development
- Requires Node.js >= 20.11.0
- Python 3 for build scripts
- Uses Yarn package manager (v3.6.1)
- Development server runs on port 1313

### External Content
- API examples automatically sync from datadog-api-client-* repositories
- Integration data pulled from the `websites-sources` go module.
   - The exception is Dogweb integrations, which are pulled from a GitHub repository.
- Some documentation is sourced from GitHub using the `pull_config.yaml` file at `local/bin/py/build/configurations/pull_config.yaml`.
- Build scripts fetched from external repository during setup

### Build Environments
- **Development**: Local development with drafts
- **Preview**: Branch previews for PRs
- **Live**: Production deployment

### API Examples Architecture
API code examples are managed through a templated system:
- Source repositories: `datadog-api-client-{go,java,python,ruby,typescript,rust}`
- When building, examples are cloned to `examples/` directory based on SDK versions in `data/sdk_versions.json`
- Examples are copied from SDK repos into `content/en/api` directory structure
- On master branch: uses tagged SDK versions
- On feature branches: attempts to use matching branch name, falls back to SDK version
- Clean individual language examples with targets like `examples/clean-python-examples`

### Markdown and Hugo Notes
- Uses Goldmark markdown parser (CommonMark 0.29 compliant)
- HTML is allowed in markdown (via `markup.goldmark.renderer.unsafe: true`)
- File/directory names must be lowercase (build server is case-sensitive, local macOS is not)
- Ignored files during build: `.mdoc.md` source files, `integrations_data/`, `hugpython/`, `examples/`, `local/`
- Hugo timeout set to 300s for large builds
- GitInfo enabled for accurate lastmod dates in sitemap

### Build Scripts System
The documentation build relies on external build scripts that are fetched dynamically:
- Build scripts are downloaded from an external repo during `make setup-build-scripts`
- Configuration in `Makefile.config` (create from `Makefile.config.example`)
- Key scripts end up in `local/bin/py/build/` and `local/bin/js/`
- Important configs preserved during updates: `pull_config.yaml`, `pull_config_preview.yaml`, `integration_merge.yaml`
- Use `make start-preserve-build` to test local changes to build scripts without refetching

### Deployment and CI/CD
- Merging to `master` triggers automatic deployment (typically starts within 10 minutes, takes ~35 minutes)
- Branch previews generated automatically for PRs with correct branch naming (`<name>/<description>`)
- Preview builds use `pull_config_preview.yaml`, production uses `pull_config.yaml`
- Docker-based builds available via `make start-docker` (uses `public.ecr.aws/x2b9z2t7/webops/site-build:latest`)
- Environment variables for preview S3 content: `FF_S3_PATH` (defaults to "staging")