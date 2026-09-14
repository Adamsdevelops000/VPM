# Contributing to VPM (Vertex Package Manager)

Thank you for your interest in contributing to VPM! This document provides guidelines for contributing to our project.

## Getting Started

1. **Fork the repository** on GitHub
2. **Clone your fork** locally:
   ```bash
   git clone https://github.com/YOUR_USERNAME/VPM.git
   cd VPM
   ```
3. **Create a feature branch**:
   ```bash
   git checkout -b feature/your-feature-name
   ```

## Development Setup

```bash
# Install dependencies (Debian/Ubuntu)
sudo apt-get install build-essential cmake git perl libssl-dev zlib1g-dev \
  libbz2-dev liblzma-dev liblz4-dev libzstd-dev

# Build VPM
cmake . -DCMAKE_BUILD_TYPE=Debug
make -j$(nproc)

# Run tests
make test
./test/integration/run-tests -q
```

## Code Style

VPM follows these conventions:

- **Indentation**: 3 spaces with 8-space tabs
- **Line Length**: 100 characters (soft limit)
- **Naming**: `camelCase` for variables, `PascalCase` for classes
- **Comments**: Doxygen-style for public APIs

### Vim Configuration

```vim
setlocal shiftwidth=3 noexpandtab tabstop=8
```

### Clang-Format

Run `./git-clang-format.sh` before committing to ensure consistent formatting.

## Commit Messages

Write clear, descriptive commit messages:

```
Subsystem: Brief description (50 chars max)

More detailed explanation of changes (wrap at 72 chars).
Explain the "why" not just the "what".

Fixes #123
```

## Pull Request Process

1. **Update documentation** if needed
2. **Add tests** for new features or bug fixes
3. **Run the test suite** locally:
   ```bash
   make test && ./test/integration/run-tests -q
   ```
4. **Push to your fork** and **create a Pull Request**
5. **Respond to feedback** and update as needed
6. A maintainer will review and merge when ready

## Reporting Bugs

When reporting bugs, include:

- **VPM version**: `vpm --version`
- **Operating system** and version
- **Steps to reproduce** the issue
- **Expected vs actual** behavior
- **Error messages** or logs (use code blocks)
- **System information**: `uname -a`, `dpkg -l | grep vpm`

## Suggesting Features

- Check existing issues first to avoid duplicates
- Explain the use case and why it's useful
- Provide examples if possible
- Be open to feedback and discussion

## Areas for Contribution

### High Priority
- Performance optimizations
- Memory usage improvements
- Test coverage expansion
- Documentation improvements

### Medium Priority
- Bug fixes
- Error message improvements
- Localization/translations
- Feature implementations from discussions

### Low Priority
- Code style cleanups
- Comment improvements
- Example scripts

## License

By contributing to VPM, you agree that your contributions will be licensed under the same GPL 2.0 license as the project.

## Questions?

Feel free to:
- Open an issue with a question tag
- Start a discussion on GitHub
- Contact the maintainers

We appreciate all contributions!
