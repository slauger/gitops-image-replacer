# CHANGELOG

## v0.1.0 (2025-11-25)

### Feature

* feat: prepare for PyPI release v1.0.0

- Restructure project for PyPI packaging
- Add pyproject.toml with package configuration
- Move script to src/gitops_image_replacer package
- Add GitHub Actions workflow for automated releases
- Add CHANGELOG.md for version tracking
- Update README with pip installation instructions
- Add MANIFEST.in for package files
- Add .gitignore for Python projects

Changes:
- API response caching (~50% fewer API calls)
- Support case-sensitive image names
- Debug output only in verbose mode
- Refactored pattern constants
- Correct URL encoding (quote instead of quote_plus)
- Comprehensive documentation updates ([`baef5eb`](https://github.com/slauger/gitops-image-replacer/commit/baef5ebee8b544eabe096155329d043cb425d90a))

* feat: refactoring (caching, multiple issues) ([`7b79ad3`](https://github.com/slauger/gitops-image-replacer/commit/7b79ad3580839c121e72c4ed20cbbbf1083a4bd7))

### Fix

* fix: use working release pipeline from netscaler-certbot-hook

- Replace ncipollo/create-release with direct python-semantic-release
- Use Python 3.9 consistently across all jobs
- Match proven workflow structure from netscaler-certbot-hook repo ([`db1aa2a`](https://github.com/slauger/gitops-image-replacer/commit/db1aa2a6493306cf05cfcff15dc7694489ad58ce))

* fix: use Python Semantic Release workflow

- Replace custom release action with python-semantic-release
- Add semantic_release configuration to pyproject.toml
- Use proven workflow from netscaler-certbot-hook project
- Automatic versioning based on conventional commits ([`3eb4c09`](https://github.com/slauger/gitops-image-replacer/commit/3eb4c090c91ac35d1b49a79a325d84a11aef5c1b))

### Unknown

* Merge pull request #1 from slauger/develop

release on pypi.org ([`6185548`](https://github.com/slauger/gitops-image-replacer/commit/61855484e1c0f6d1c1ace23a1998932977135886))

* inital commit ([`73fbe68`](https://github.com/slauger/gitops-image-replacer/commit/73fbe68b242df1eb2d7778a225926d180a26c69b))
