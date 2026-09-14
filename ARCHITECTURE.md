# VPM (Vertex Package Manager) Architecture

## System Overview

VPM is a modular, layered package management system inspired by APT with modern enhancements:

```
┌─────────────────────────────────────────────────┐
│  User Interface Layer                           │
│  ┌──────────────┬──────────────┬──────────────┐ │
│  │ vpm (modern) │ vpm-get (compat) │ vpm-cache │ │
│  └──────────────┴──────────────┴──────────────┘ │
└────────────────┬────────────────────────────────┘
                 │
┌────────────────▼────────────────────────────────┐
│  Business Logic Layer                           │
│  ┌──────────────┬──────────────┬──────────────┐ │
│  │ Install Mgr  │ Cache Builder │ Dependency  │ │
│  │              │              │ Resolver    │ │
│  └──────────────┴──────────────┴──────────────┘ │
└────────────────┬────────────────────────────────┘
                 │
┌────────────────▼────────────────────────────────┐
│  Core Library Layer (vpm-pkg)                   │
│  ┌──────────────────────────────────────────┐  │
│  │ Package Cache │ Solver3 │ Source Lists   │  │
│  │ Acquire System │Index Files │ Policies   │  │
│  └──────────────────────────────────────────┘  │
└────────────────┬────────────────────────────────┘
                 │
┌────────────────▼────────────────────────────────┐
│  Download/Protocol Layer (methods)              │
│  ┌──────────────┬──────────────┬──────────────┐ │
│  │ HTTP/HTTPS   │ FTP          │ File         │ │
│  │ Compression  │ Copy         │ Verify       │ │
│  └──────────────┴──────────────┴──────────────┘ │
└────────────────┬────────────────────────────────┘
                 │
┌────────────────▼────────────────────────────────┐
│  System Integration                             │
│  ├─ Package Database (/var/lib/vpm)            │
│  ├─ Repository Lists (/etc/vpm/sources.list)  │
│  ├─ Configuration (/etc/vpm/vpm.conf.d)        │
│  └─ dpkg Integration                           │
└─────────────────────────────────────────────────┘
```

## Core Components

### 1. vpm-pkg Library

**Purpose**: Provides all package management functionality.

**Key Modules**:

#### Cache Management (`pkgcache.*`, `pkgcachegen.*`)
- In-memory representation of available and installed packages
- Version, dependency, and conflict information
- Efficient lookup and filtering mechanisms
- Memory-mapped for large repositories

#### Dependency Resolution (`solver3.*`, `depcache.*`)
- SAT-based constraint solver for optimal dependency resolution
- Handles version constraints, conflicts, and provides/replaces
- Tracks package installation state
- Identifies broken dependencies and conflicts

#### Acquire System (`acquire.*`, `acquire-worker.*`, `acquire-item.*`)
- Parallel download scheduling
- Multi-source URI handling with fallback
- Progress reporting and resumable downloads
- Method spawning and inter-process communication

#### Repository Management (`sourcelist.*`, `indexfile.*`)
- Parse and manage package sources
- Load package lists from repositories
- Handle release files and signatures
- Pin policy management

#### Package Records (`pkgrecords.*`, `pkgsystem.*`)
- Query detailed package information
- Handle Debian package metadata
- Provide package data to tools

### 2. Command-Line Interfaces

#### vpm (Modern Interface)
**File**: `cmdline/vpm.cc`

High-level user-friendly interface:
```bash
vpm install package           # Install
vpm remove package            # Remove
vpm upgrade                    # Upgrade packages
vpm full-upgrade              # Full system upgrade
vpm search term               # Search packages
vpm show package              # Show details
vpm list                       # List packages
vpm update                     # Refresh package lists
```

#### vpm-get (Compatibility Interface)
**File**: `cmdline/vpm-get.cc`

Low-level tool interface for scripting.

#### vpm-cache (Query Tool)
**File**: `cmdline/vpm-cache.cc`

Query package cache without modifying state.

#### vpm-mark (State Management)
**File**: `cmdline/vpm-mark.cc`

Manage package installation state (hold, auto).

### 3. Download Methods

**Directory**: `methods/`

Protocol-specific handlers spawned as separate processes:

- **http/https**: HTTP(S) downloads with pipelining
- **ftp**: FTP protocol support
- **file**: Local file access
- **copy**: In-filesystem copying
- **gzip**: Gzip decompression
- **bzip2**: Bzip2 decompression
- **lzma**: LZMA decompression
- **xz**: XZ compression support

### 4. Repository Tools

**Directory**: `ftparchive/`

- **vpm-ftparchive**: Generate package indices for repositories
- **vpm-sortpkgs**: Normalize package list files
- Support for creating release files and signatures

## Data Flow

### Package Installation Flow

```
1. User: vpm install package
   ↓
2. CLI Parser: Parse command line
   ↓
3. Acquire: Download package lists from sources
   ↓
4. Cache Builder: Parse package indices into memory
   ↓
5. Dependency Resolver: Compute installation set
   ↓
6. Download Manager: Fetch required packages
   ↓
7. Installation: Call dpkg to unpack/configure
   ↓
8. State Tracking: Update /var/lib/vpm
```

### Cache Generation Flow

```
1. Read /etc/vpm/sources.list
   ↓
2. Spawn HTTP/FTP methods for each source
   ↓
3. Download Packages, Sources, Release files
   ↓
4. Parse and merge into unified package cache
   ↓
5. Store cache in /var/cache/vpm/pkgcache.bin
   ↓
6. Build dependency graph
```

### Dependency Resolution Flow

```
1. User requests: install=[], remove=[]
   ↓
2. Solver3 constraint setup
   ↓
3. Build SAT formula from dependencies
   ↓
4. Solve constraints
   ↓
5. Return solution: [to_install], [to_remove]
```

## Configuration System

VPM reads configuration from multiple files in order:

```
1. /etc/vpm/vpm.conf
2. /etc/vpm/vpm.conf.d/*.conf
3. Environment variables (APT_CONFIG)
4. Command-line options
```

## State Management

### /var/lib/vpm/
```
├── lists/                 # Downloaded package indices
│   ├── partial/          # Incomplete downloads
│   └── [source]/         # Per-source package lists
├── extended_states       # Manual/auto install markers
├── mirrors/              # Mirror information
└── periodic/             # Periodic task state
```

### /var/cache/vpm/
```
├── pkgcache.bin         # Compiled package cache
├── srcpkgcache.bin      # Compiled source package cache
└── archives/            # Downloaded .deb files
    └── partial/         # Incomplete downloads
```

## Dependency Solver (solver3)

VPM uses a modern SAT-based solver:

- **Input**: Package constraints (installed, conflicts, depends)
- **Algorithm**: DPLL/CDCL SAT solver
- **Output**: Solution (packages to install/remove) or conflict report
- **Optimization**: Minimizes removals, prefers upgrades

### Solver Phases

1. **Setup Phase**: Translate dependencies to SAT clauses
2. **Solving Phase**: Compute satisfying assignment
3. **Validation Phase**: Verify solution correctness
4. **Reporting Phase**: Output changes to user

## Error Handling

VPM provides detailed error reporting:

- **Dependency Conflicts**: Lists conflicting packages
- **Broken Packages**: Identifies unmet dependencies
- **Lock Failures**: Reports when resources are unavailable
- **Network Errors**: Implements retry logic and mirror fallback

## Thread Safety

- **Acquire system**: Thread-safe for multi-method downloads
- **Cache**: Read-only after initialization
- **State tracking**: Atomic writes for consistency

## Performance Characteristics

- **Cache load**: O(1) package lookups
- **Dependency resolution**: O(n²) SAT solving
- **Download scheduling**: O(1) per item enqueue
- **Memory usage**: Proportional to repository size (~100MB typical)

## Extension Points

1. **Custom Methods**: Add protocol handlers in `/usr/lib/vpm/methods/`
2. **External Solvers**: Implement EDSP protocol
3. **Configuration Hooks**: Use `DPkg::Pre/Post-Invoke`
4. **Status Callbacks**: Hook into acquire progress reporting

## Security

- **Repository Authentication**: GPG signature verification
- **Package Verification**: MD5/SHA256 hash checking
- **Privilege Escalation**: Runs with minimal privileges
- **Sandbox Methods**: External methods run with restricted permissions
