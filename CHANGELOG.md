# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2025-11-25

### Added
- Initial release of gitops-image-replacer
- Automated container image updates across multiple GitHub repositories
- Support for Docker image tags, digests, and combined references
- Dry-run mode for safe validation before applying changes
- CI mode with regex pattern matching for `GIT_REF` validation
- Configuration via JSON and YAML files
- GitHub API integration with retry logic and error handling
- Response caching to reduce API calls by ~50%
- Comprehensive regex pattern for validating container image formats
- Case-sensitive image name support (uppercase and lowercase)
- Verbose mode for debugging with detailed logging
- MIT License

### Features
- **Multi-repository support**: Update images across any number of repositories and files
- **Flexible image formats**: Handles `name:tag`, `name@digest`, and `name:tag@digest`
- **Pattern matching**: Optional `when`/`except` regex patterns for conditional updates
- **Safe replacements**: Regex escaping prevents unintended replacements
- **Robust HTTP**: Automatic retries, timeouts, and proper error handling
- **Performance optimized**: API response caching eliminates duplicate requests

### Documentation
- Comprehensive README with usage examples
- Detailed regex pattern documentation
- Best practices for security and performance
- Troubleshooting guide
- Real-world use cases for CI/CD pipelines

[1.0.0]: https://github.com/slauger/gitops-image-replacer/releases/tag/v1.0.0
