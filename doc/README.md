# VPM Documentation

Welcome to the Vertex Package Manager documentation!

## Quick Links

- [Getting Started](getting-started.md) - Installation and basic usage
- [Architecture](../ARCHITECTURE.md) - System design and components
- [Configuration Reference](configuration.md) - Config files and options
- [User Guide](user-guide.md) - Command-line usage
- [Developer Guide](developer-guide.md) - Building and contributing
- [EDSP Protocol](external-dependency-solver-protocol.md) - External solver integration

## Key Concepts

### Package Sources

VPM retrieves packages from configured sources (repositories). Sources are defined in `/etc/vpm/sources.list`:

```
deb [arch=amd64] https://packages.example.com/debian focal main
deb-src https://packages.example.com/debian focal main
```

### Package State

Packages can be in several states:
- **Not installed**: Not on the system
- **Installed**: Currently installed
- **Unpacked**: Files extracted but not configured
- **Half-configured**: Configuration incomplete
- **On hold**: Installation/removal blocked by user

### Dependency Resolution

VPM uses a SAT solver to determine which packages must be installed to satisfy dependencies.

### Package Priority

Packages are prioritized by:
1. User request (install/remove)
2. Package pin value
3. Version number
4. Dependency relationships

## File Structure

```
doc/
├── README.md                    # This file
├── architecture.md              # System architecture
├── configuration.md             # Configuration reference
├── getting-started.md           # Quick start guide
├── user-guide.md                # Usage documentation
├── developer-guide.md           # Development guide
├── external-dependency-solver-protocol.md  # EDSP spec
└── protocols/                   # Protocol specifications
    ├── edsp.md                  # EDSP v0.5
    └── eipp.md                  # EIPP installation planner
```

## For Different Users

### End Users
Start with [Getting Started](getting-started.md) and [User Guide](user-guide.md).

### System Administrators
Read [Configuration Reference](configuration.md) for setting up repositories and policies.

### Developers
See [Developer Guide](developer-guide.md) and [Architecture](../ARCHITECTURE.md).

### Distribution Maintainers
Read about [Repository Tools](../ftparchive/) for building package indices.

## Contributing to Docs

Documentation improvements are welcome! Please:
1. Follow the existing style
2. Use clear, concise language
3. Include examples where helpful
4. Keep cross-references up to date

See [CONTRIBUTING.md](../CONTRIBUTING.md) for details.
