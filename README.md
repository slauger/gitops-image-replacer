# gitops-image-replacer

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![PyPI version](https://img.shields.io/pypi/v/gitops-image-replacer.svg)](https://pypi.org/project/gitops-image-replacer/)
[![PyPI downloads](https://img.shields.io/pypi/dm/gitops-image-replacer.svg)](https://pypi.org/project/gitops-image-replacer/)

A lightweight CLI tool that automates container image updates in GitOps repositories. Replace image tags and/or digests across multiple GitHub repositories with a single command, enabling automated deployment workflows.

## Features

- **Flexible Modes**: Dry-run for validation, apply mode for commits
- **CI/CD Integration**: Built-in CI mode with `GIT_REF` pattern matching
- **Complete Image Support**: Handles tags, digests, and combined references (e.g., `registry.io/ns/app:1.2.3@sha256:...`)
- **Multiple Repositories**: Update images across any number of repos and files
- **Configuration Formats**: JSON (default) and YAML support
- **Performance Optimized**: Response caching eliminates duplicate API calls
- **Safe Operations**: Regex escaping prevents unintended replacements
- **Robust HTTP**: Automatic retries, timeouts, and error handling
- **Clean Logging**: Minimal output by default, verbose mode for debugging

## Requirements

- Python 3.8+
- A GitHub/GitHub Enterprise token with content read/write access

## Installation

### Via pip (recommended)

```bash
# Install from PyPI
pip install gitops-image-replacer

# Verify installation
gitops-image-replacer --help
```

### From source

```bash
# Clone repository
git clone https://github.com/slauger/gitops-image-replacer.git
cd gitops-image-replacer

# Install in development mode
pip install -e .

# Or run directly
python -m gitops_image_replacer --help
```

## Quick Start

1. Create a configuration file (default: `gitops-image-replacer.json`).
2. Run a dry-run:
   ```bash
   gitops-image-replacer docker.io/example/app:2.0.0

   gitops-image-replacer docker.io/example/app:2.0.0@sha256:4bcff63911fcb4448bd4fdacec207030997caf25e9bea4045fa6c8c44de311d1

   gitops-image-replacer docker.io/example/app@sha256:4bcff63911fcb4448bd4fdacec207030997caf25e9bea4045fa6c8c44de311d1

   gitops-image-replacer --config gitops-image-replacer.json docker.io/example/app:2.0.0
   ```
3. Apply changes (commit to target repos):
   ```bash
   gitops-image-replacer --apply docker.io/example/app:2.0.0

   gitops-image-replacer --config gitops-image-replacer.json --apply docker.io/example/app:2.0.0
   ```

## CLI

```text
usage: gitops-image-replacer [-h] [--config <file>] [--apply] [--ci]
                              [--name <string>] [--email <string>]
                              [--message <string>] [--api <string>]
                              [--verbose]
                              <string>
```

- `--config` Path to the configuration file (default: `gitops-image-replacer.json`). JSON recommended.
- `--apply` Apply changes (commit). Without this flag the tool runs in dry-run.
- `--ci` CI mode: validates `GIT_REF` against `when`/`except` regex patterns from config.
- `--name` Commit author name (default: env `GIT_COMMIT_NAME` or `Replacer Bot`).
- `--email` Commit author email (default: env `GIT_COMMIT_EMAIL` or `replacer-bot@localhost.localdomain`).
- `--message` Commit message template (default: `fix: update image to {}`).
- `--api` GitHub API URL (default: env `GITHUB_API_URL` or `https://github.com/api/v3`).
- `--verbose` Print file contents and desired state (use with care in CI logs).
- Positional: `image` (e.g., `docker.io/foo/bar:2.0.0`). Tag and digest are optional.

### Environment

- `GITHUB_TOKEN` **(required)** – token with access to read/write repository contents.
- `GIT_REF` *(required when `--ci`)* – the current ref string, e.g., `refs/heads/main`.

Recommended token scopes:  
- Public repos only: `public_repo`  
- Private repos: `repo`  
- GitHub Enterprise: equivalent content permissions

## Configuration

Default format is **JSON**. YAML (`.yaml`/`.yml`) is supported as well.

### JSON schema (per entry)

```json
{
  "gitops-image-replacer": [
    {
      "repository": "org/repo",
      "branch": "main",
      "file": "path/to/values.yaml",
      "when": "^refs/heads/(main|release/.*)$",
      "except": "^refs/heads/feature/"
    }
  ]
}
```

**Fields**

- `repository` (string): `ORG/REPO` path on GitHub/GHE
- `branch` (string): target branch
- `file` (string): target file path in the repository
- `when` (string, optional): regex that must match `GIT_REF` when `--ci` is enabled
- `except` (string, optional): regex that must **not** match `GIT_REF` when `--ci` is enabled

> The tool uses `re.match` (anchored at the string start). Use `^...$` in your patterns if you require a full match.

### Examples

**JSON (default)**

```json
{
  "gitops-image-replacer": [
    {
      "repository": "acme/online-shop",
      "branch": "main",
      "file": "deploy/values.yaml",
      "when": "^refs/heads/(main|release/.*)$"
    },
    {
      "repository": "acme/payments",
      "branch": "develop",
      "file": "charts/payments/values.yaml",
      "except": "^refs/heads/legacy/"
    }
  ]
}
```

**YAML (alternative)**

```yaml
gitops-image-replacer:
  - repository: acme/online-shop
    branch: main
    file: deploy/values.yaml
    when: '^refs/heads/(main|release/.*)$'
  - repository: acme/payments
    branch: develop
    file: charts/payments/values.yaml
    except: '^refs/heads/legacy/'
```

## Supported Image Formats

The tool validates and matches container images using a comprehensive regex pattern. All components are optional except the image name.

### Regex Pattern

```regex
(?P<repository>[\w.\-_]+((?::\d+|)(?=/[a-zA-Z0-9._-]+/[a-zA-Z0-9._-]+))|)(?:/|)(?P<image>[a-zA-Z0-9.\-_]+(?:/[a-zA-Z0-9.\-_]+|))(:(?P<tag>[\w.\-_]{1,127})|)?(@(?P<digest>sha256:[a-f0-9]{64}))?
```

### Pattern Components

- **Repository/Registry** (optional): `docker.io`, `gcr.io`, `registry.example.com:5000`
  - Supports hostnames with optional ports
  - Alphanumeric, dots, dashes, underscores
- **Image Name** (required): `library/nginx`, `myorg/myapp`, `MyApp`
  - Case-sensitive (supports both uppercase and lowercase)
  - Can include organization/namespace
- **Tag** (optional): `:latest`, `:v1.2.3`, `:20241125-abc123`
  - Max 127 characters
  - Alphanumeric, dots, dashes, underscores
- **Digest** (optional): `@sha256:4bcff63911fcb4448bd4fdacec207030997caf25e9bea4045fa6c8c44de311d1`
  - SHA256 hash (64 hex characters, lowercase)

### Valid Examples

```bash
# Simple image name
nginx

# With registry and tag
docker.io/library/nginx:1.25

# With digest only
gcr.io/myproject/myapp@sha256:4bcff63911fcb4448bd4fdacec207030997caf25e9bea4045fa6c8c44de311d1

# Tag and digest (immutable reference)
registry.example.com:5000/org/app:v2.0.0@sha256:abc...

# Case-sensitive names
ghcr.io/MyOrg/MyApp:latest
```

## How it works

1. **Validation**: Checks CLI arguments, environment variables, and configuration file
2. **Precheck Phase**: Validates access to all target repositories/files (caches responses)
3. **Replace Phase**: Downloads files (reuses cached data), performs safe regex replacement
4. **Commit Phase**: If `--apply` is set and changes detected, commits via GitHub Contents API
5. **Exit Codes**: Returns `0` on success, non-zero on failures

### Performance Optimization

The tool caches file contents from the precheck phase, eliminating duplicate API calls during the replace phase. This reduces GitHub API usage by approximately 50% in typical scenarios.

## Exit Codes

- `0` success (no changes or committed changes)
- `1` validation or API error

## Best Practices

### Testing and Validation
- **Always start with dry-run**: Test your configuration without `--apply` first
- **Use verbose mode carefully**: `--verbose` prints file contents - avoid in CI logs with sensitive data
- **Validate regex patterns**: Use `^...$` anchors in `when`/`except` for full string matches

### Configuration Management
- **Keep config versioned**: Store `gitops-image-replacer.json` in your repository
- **Use meaningful names**: Standard naming makes the file's purpose obvious
- **Document patterns**: Comment your regex patterns (especially in CI mode)

### Security
- **Limit token scope**: Use minimal permissions (contents: write)
- **Rotate tokens**: Regularly update GitHub tokens
- **Review changes**: Use dry-run before production deployments

### Performance
- **Minimize targets**: Only configure files that need updates
- **Use CI patterns wisely**: `when`/`except` patterns reduce unnecessary runs
- **Leverage caching**: The tool automatically caches API responses

## Troubleshooting

### Common Issues

**401 Unauthorized**
- Verify `GITHUB_TOKEN` is set correctly
- Check token has `repo` or `public_repo` scope
- For GitHub Enterprise, confirm token has access to the organization

**404 Not Found**
- Verify `repository`, `branch`, and `file` paths in config
- Check branch name spelling (case-sensitive)
- Ensure file exists at the specified path

**No changes detected**
- Confirm target file contains the exact image name (case-sensitive)
- Tags and digests are optional in the search pattern
- Use `--verbose` to see file contents and verify the image reference format

**Rate limiting (429/403)**
- Built-in retries handle temporary rate limits
- Reduce execution frequency in CI/CD pipelines
- Consider using GitHub App tokens for higher limits

### Debug Mode

Run with `--verbose` to see:
- Full API URLs being called
- Complete file contents before replacement
- Desired file contents after replacement

**Warning**: Verbose mode may expose sensitive data in logs.

## Use Cases

### Automated Deployment Pipeline

Update production manifests when a new image is built:

```bash
# In your CI/CD pipeline after building image
gitops-image-replacer --apply docker.io/myorg/myapp:${CI_COMMIT_SHA}
```

### Multi-Environment Updates

Use CI mode to update different environments based on branch:

```json
{
  "gitops-image-replacer": [
    {
      "repository": "myorg/gitops-production",
      "branch": "main",
      "file": "apps/myapp/deployment.yaml",
      "when": "^refs/heads/main$"
    },
    {
      "repository": "myorg/gitops-staging",
      "branch": "main",
      "file": "apps/myapp/deployment.yaml",
      "when": "^refs/heads/(main|release/.*)$"
    }
  ]
}
```

### Digest Pinning

Update images with immutable digest references:

```bash
gitops-image-replacer --apply \
  registry.example.com/myapp:v2.0.0@sha256:4bcff63911fcb4448bd4fdacec207030997caf25e9bea4045fa6c8c44de311d1
```

## Contributing

Contributions are welcome! Please ensure:
- Code follows existing style and patterns
- Changes are tested with both dry-run and apply modes
- Documentation is updated for new features

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
