# gitops-image-replacer

A small CLI tool that replaces container image references (tag and/or digest) in files across one or more GitHub repositories and optionally commits the changes.

## Features

- Dry-run and apply mode
- Optional CI mode validating `GIT_REF` against regex patterns
- Supports tags *and* digests (e.g., `registry.io/ns/app:1.2.3@sha256:...`)
- Per-repository/file configuration (JSON and YAML are supported)
- Safe regex replacement with escaping
- Robust HTTP calls with retries and timeouts
- Clean logging with optional `--verbose`

## Requirements

- Python 3.8+
- A GitHub/GitHub Enterprise token with content read/write access

## Installation

```bash
# Run directly
python3 gitops-image-replacer.py --help

# Or mark as executable
chmod +x gitops-image-replacer.py
./gitops-image-replacer.py --help
```

## Quick Start

1. Create a configuration file (default: `gitops-image-replacer.json`).  
2. Run a dry-run:
   ```bash
   ./gitops-image-replacer.py docker.io/example/app:2.0.0

   ./gitops-image-replacer.py docker.io/example/app:2.0.0@sha256:4bcff63911fcb4448bd4fdacec207030997caf25e9bea4045fa6c8c44de311d1

   ./gitops-image-replacer.py docker.io/example/app@sha256:4bcff63911fcb4448bd4fdacec207030997caf25e9bea4045fa6c8c44de311d1

   ./gitops-image-replacer.py --config gitops-image-replacer.json docker.io/example/app:2.0.0
   ```
3. Apply changes (commit to target repos):
   ```bash
   ./gitops-image-replacer.py --apply docker.io/example/app:2.0.0

   ./gitops-image-replacer.py --config gitops-image-replacer.json --apply docker.io/example/app:2.0.0
   ```

## CLI

```text
usage: gitops-image-replacer.py [-h] [--config <file>] [--apply] [--ci]
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

## How it works

1. Validate CLI/env and configuration file.
2. Precheck each target (file exists? branch accessible? permissions ok?).  
3. Download the file, replace the image reference (`<name>[:<tag>][@<digest>]`) using a safe regex.  
4. If changes are detected and `--apply` is set, commit via GitHub Contents API (`PUT /repos/{owner}/{repo}/contents/{path}`).  
5. Exit code `0` on success, non-zero on failures.

## Exit Codes

- `0` success (no changes or committed changes)
- `1` validation or API error

## Best Practices

- Start with a **dry-run**.
- Use `--verbose` only in trusted logs (it can print file contents).
- Add `^...$` anchors in `when/except` if you need full matches.
- Keep config close to where it’s used; naming the file `gitops-image-replacer.json` makes ownership obvious.

## Troubleshooting

- **401 Unauthorized**: Check token scopes and repository access.
- **404 Not Found**: Verify `repository`, `branch`, and `file` path.
- **No changes detected**: Ensure the target file actually contains the image name you pass; tags/digests are optional in the match.
- **Rate limiting (429/403)**: Retries are built-in; consider reducing frequency or increasing concurrency limits.
