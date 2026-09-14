# Vertex Package Manager (VPM)

**VPM** is a modern, high-performance package manager built on proven APT architecture with enhanced features for dependency resolution, parallel downloads, and multi-repository support.

## Features

- **Fast Dependency Resolution**: SAT-based solver3 algorithm for optimal package resolution
- **Parallel Downloads**: Concurrent multi-source package fetching with automatic mirror fallback
- **Multi-Protocol Support**: HTTP, HTTPS, FTP, and local repository sources
- **Smart Caching**: Efficient in-memory package index with minimal disk footprint
- **Transaction History**: Full undo/redo support for package operations
- **External Solver Integration**: EDSP protocol support for pluggable dependency solvers
- **Configuration Management**: Flexible conf.d-based configuration system
- **Interactive CLI**: Both high-level (vpm) and low-level (vpm-get) interfaces

## Quick Start

### Building from Source

```bash
# Clone the repository
git clone https://github.com/Adamsdevelops000/VPM.git
cd VPM

# Configure with CMake
cmake . -DCMAKE_BUILD_TYPE=Release

# Build with parallel jobs
make -j$(nproc)

# Or use Ninja for faster builds
cmake -G Ninja .
ninja
```

### Running Tests

```bash
# Integration tests
./test/integration/run-tests -q

# Unit tests
make test
```

### Basic Usage

```bash
# Update package lists
vpm update

# Install a package
vpm install package-name

# Upgrade all packages
vpm upgrade

# Full system upgrade with dependency changes
vpm full-upgrade

# Remove a package
vpm remove package-name

# Search for packages
vpm search keyword

# Show package details
vpm show package-name
```

## Architecture

### Core Components

- **vpm-pkg**: Core package management library
  - Package caching and indexing
  - Dependency resolution with solver3
  - Repository source management
  - Acquire system for downloads

- **cmdline**: Command-line interfaces
  - `vpm`: High-level interactive interface
  - `vpm-get`: Low-level tool interface
  - `vpm-cache`: Package query tool
  - `vpm-mark`: Package state management

- **methods**: Protocol handlers
  - HTTP/HTTPS downloads
  - FTP support
  - Local repository access
  - Compression method handlers

- **ftparchive**: Repository tools
  - Index file generation
  - Package list creation
  - Release file management

### Directory Structure

```
vpm-pkg/              Core library
  ├── acquire/        Download scheduling
  ├── contrib/        Utility functions
  ├── deb/            Package format handlers
  ├── edsp/           External solver protocol
  ├── solver3/        SAT-based dependency resolver
  └── ...

cmdline/              CLI tools
  ├── vpm.cc          Modern interface
  ├── vpm-get.cc      Legacy compatibility
  └── ...

methods/              Protocol handlers

ftparchive/           Repository tools

test/                 Test suite
  ├── integration/    End-to-end tests
  └── libapt/         Unit tests

doc/                  Documentation & protocols
```

## Configuration

VPM is configured via files in `/etc/vpm/vpm.conf.d/` with options like:

```bash
# Enable debug output
Debug::pkgAcquire "true";

# Set default solver
VPM::Solver "internal";

# Configure sources
Dir::Etc::sourcelist "/etc/vpm/sources.list";
```

## Repository Sources

Add package sources to `/etc/vpm/sources.list`:

```
deb [arch=amd64] https://packages.example.com/debian focal main universe
deb-src https://packages.example.com/debian focal main
```

## Development

VPM follows these conventions:

- **Language**: C++23
- **Build System**: CMake 3.13+
- **Code Style**: 3-space indentation, 8-space tabs
- **Testing**: gtest for unit tests, shell scripts for integration tests
- **Dependencies**: OpenSSL, zlib, bzip2, LZMA, LZ4, Zstd, curl

### Contributing

We welcome contributions! Please:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes
4. Push to the branch
5. Open a Pull Request

See [CONTRIBUTING.md](CONTRIBUTING.md) for detailed guidelines.

## Documentation

- [Architecture Guide](doc/architecture.md)
- [EDSP Protocol](doc/external-dependency-solver-protocol.md)
- [Configuration Reference](doc/configuration.md)
- [Build Instructions](doc/building.md)

## License

VPM is licensed under the GNU General Public License v2.0 or later. See [COPYING](COPYING) for details.

## Credits

VPM is inspired by and built upon the proven architecture of APT (Advanced Package Tool). We acknowledge the Debian APT team's decades of work in package management.

## Support

- **Issues**: [GitHub Issues](https://github.com/Adamsdevelops000/VPM/issues)
- **Discussions**: [GitHub Discussions](https://github.com/Adamsdevelops000/VPM/discussions)
- **Documentation**: [Full Docs](doc/)

---

**Vertex Package Manager** - Elevate your package management.
