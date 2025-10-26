# Build Guide - Open Integration Engine

This document describes how to build and test the Open Integration Engine project, an open source implementation of Mirth Connect.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [Project Structure](#project-structure)
- [Build Commands](#build-commands)
- [Testing](#testing)
- [Module-Specific Builds](#module-specific-builds)
- [Build Outputs](#build-outputs)
- [CI/CD Pipeline](#cicd-pipeline)
- [Troubleshooting](#troubleshooting)

## Prerequisites

### Required Software

1. **Java Development Kit (JDK) 8 with JavaFX**
   - Recommended: Zulu 8.0.462.fx-zulu
   - Must include JavaFX (fx package)

2. **Apache Ant 1.10.14**
   - Required for build orchestration

3. **SDKMAN (Recommended)**
   - Simplifies Java and Ant version management

### Setup Instructions

#### Using SDKMAN (Recommended)

```bash
# Install SDKMAN if not already installed
curl -s "https://get.sdkman.io" | bash
source "$HOME/.sdkman/bin/sdkman-init.sh"

# Install correct Java and Ant versions from project root
cd /path/to/engine
sdk env install

# Verify installation
java -version   # Should show 8.0.462.fx-zulu
ant -version    # Should show 1.10.14
```

#### Manual Installation

If not using SDKMAN, ensure Java 8 with JavaFX and Ant 1.10.14 are installed and available in your PATH.

### System Requirements

- Minimum 512MB RAM for basic compilation
- 2GB RAM recommended for full build with tests
- 2-3GB disk space for build artifacts

## Quick Start

```bash
# Navigate to server directory (contains master build orchestration)
cd server

# Full build without code signing (recommended for development)
ant -f mirth-build.xml -DdisableSigning=true

# Run all tests
ant -f mirth-build.xml test-run

# Create distribution package
ant -f mirth-build.xml dist
```

## Project Structure

The project consists of 8 modules built in a specific dependency order:

```
engine/
├── donkey/              # Core message routing engine (foundation)
├── server/              # Main server implementation
│   └── mirth-build.xml # Master build orchestrator
├── generator/          # HL7 vocabulary model generator
├── client/             # Desktop client GUI application
├── command/            # Command-line interface (CLI)
├── manager/            # Application manager/launcher
├── webadmin/           # Web-based administration interface
└── simplesender/       # Sample/utility application
```

### Module Dependencies

```
donkey (standalone)
  ↓
server (depends on donkey)
  ├→ client
  ├→ command
  └→ manager
```

### Build Scripts

All modules use Apache Ant with XML build files:

| Module | Build Script Location |
|--------|----------------------|
| **Master Build** | `server/mirth-build.xml` |
| Donkey | `donkey/build.xml` |
| Server | `server/build.xml` |
| Generator | `generator/build.xml` |
| Client | `client/ant-build.xml` |
| Command | `command/build.xml` |
| Manager | `manager/ant-build.xml` |
| WebAdmin | `webadmin/build.xml` |

## Build Commands

### Full Build

The master build orchestrates all modules in the correct dependency order:

```bash
cd server

# Production build with JAR signing
ant -f mirth-build.xml

# Development build without signing (faster)
ant -f mirth-build.xml -DdisableSigning=true

# Build with custom version
ant -f mirth-build.xml -Dversion=4.5.3
```

### Common Build Targets

```bash
# Clean all build artifacts
ant -f mirth-build.xml clean

# Compile all modules
ant -f mirth-build.xml compile

# Create setup directories
ant -f mirth-build.xml create-setup

# Package extensions
ant -f mirth-build.xml create-extension-zips

# Generate Javadocs
cd server
ant -f build.xml create-javadocs
# Output: docs/javadocs/user-api/
```

### Development Builds

```bash
cd server

# Run server in development mode
ant -f build.xml dev-run

# Run server using launcher
ant -f build.xml dev-launcher

# Create Derby embedded database
ant -f build.xml create-derby-db
```

### Code Quality

```bash
# Add license headers to source files
cd server
ant -f mirth-build.xml append-license

# Run dependency vulnerability check
cd ..
ant -f dependency-check.xml dependency-check
# Report: dependency-check-report.html
```

## Testing

### Running Tests

The project uses JUnit for testing with JaCoCo for code coverage.

```bash
cd server

# Run all module tests
ant -f mirth-build.xml test-run

# Skip tests during build
ant -f mirth-build.xml -DdisableTests=true
```

### Module-Specific Tests

```bash
# Run tests for a specific module
cd <module-directory>
ant -f build.xml test-run
```

### Test Locations

| Module | Test Location |
|--------|---------------|
| Server | `server/test/` |
| Donkey | `donkey/testlib/` |
| Command | `command/test/` |
| Client | `client/test/` |

### Test Reports

Test results are generated in each module's directory:

- `junit-reports/` - JUnit XML test results
- `code-coverage-reports/jacoco.exec` - JaCoCo coverage data

## Module-Specific Builds

For faster iteration during development, you can build individual modules:

### Donkey (Core Library)

```bash
cd donkey
ant -f build.xml clean
ant -f build.xml build
```

### Server

```bash
cd server
ant -f build.xml clean
ant -f build.xml compile
ant -f build.xml create-setup
```

### Client (Desktop GUI)

```bash
cd client
ant -f ant-build.xml clean
ant -f ant-build.xml build
```

### Command-Line Interface

```bash
cd command
ant -f build.xml clean
ant -f build.xml build
```

### Manager

```bash
cd manager
ant -f ant-build.xml clean
ant -f ant-build.xml build
```

### WebAdmin

```bash
cd webadmin
ant -f build.xml clean
ant -f build.xml dist
```

## Build Outputs

### Generated Directories

After running the master build, the following directories are created in the `server/` directory:

```
server/
├── classes/                    # Compiled Java classes
├── test_classes/               # Compiled test classes
├── setup/                      # Deployment-ready directory
│   ├── server-lib/            # Server JARs
│   ├── client-lib/            # Client JARs
│   ├── manager-lib/           # Manager libraries
│   ├── cli-lib/               # CLI libraries
│   ├── extensions/            # Connectors and plugins
│   ├── conf/                  # Configuration files
│   ├── public_html/           # Web UI files
│   └── logs/                  # Log directory
├── dist/                       # Distribution packages
│   └── extensions/            # Packaged extension ZIPs
├── build/                      # Build intermediate files
├── junit-reports/             # JUnit XML test results
└── code-coverage-reports/     # JaCoCo coverage data
```

### Key JAR Files

- `mirth-server.jar` - Server core
- `mirth-client-core.jar` - Shared client/server APIs
- `mirth-client.jar` - Desktop GUI
- `mirth-cli-launcher.jar` - Command-line tool
- `mirth-manager-launcher.jar` - Application manager
- `mirth-server-launcher.jar` - Server launcher

### Extensions

The build creates 11 connectors and 9+ data type plugins:

**Connectors:**
- HTTP, JDBC, JMS, TCP, File, SMTP, DICOM, WebService, JavaScript, Document, Virtual Machine

**Data Type Plugins:**
- HL7v2, HL7v3, XML, JSON, FHIR, EDI, NCPDP, DICOM, Delimited, Raw

## CI/CD Pipeline

The project includes a GitHub Actions workflow (`.github/workflows/build.yaml`) that:

1. Triggers on push/PR to main branch
2. Sets up JDK 8 with JavaFX (Zulu distribution)
3. Builds with signing on main branch, without signing on other branches
4. Packages distribution as `openintegrationengine.tar.gz`
5. Uploads artifact to GitHub

### Manual CI Build

To replicate the CI build locally:

```bash
cd server

# For main branch (with signing)
ant -f mirth-build.xml

# For feature branches (without signing)
ant -f mirth-build.xml -DdisableSigning=true

# Package distribution
tar -czf openintegrationengine.tar.gz setup/
```

## Troubleshooting

### Common Issues

**Java Version Mismatch**
```bash
# Verify Java version includes JavaFX
java -version
# Should show version with "fx" in the name (e.g., 8.0.462.fx-zulu)
```

**Ant Not Found**
```bash
# Install via SDKMAN
sdk install ant 1.10.14
```

**Build Fails with Memory Error**
```bash
# Increase Ant memory (set before running ant)
export ANT_OPTS="-Xms128m -Xmx2048m"
```

**Windows Build Issues**
```bash
# Use provided batch file instead of ant directly
build.bat
```

**Incremental Build Issues**
```bash
# Clean and rebuild from scratch
cd server
ant -f mirth-build.xml clean
ant -f mirth-build.xml -DdisableSigning=true
```

### Build Order Dependencies

The master build (`server/mirth-build.xml`) handles module dependencies automatically. If building modules individually, follow this order:

1. Donkey
2. Server
3. Client, Command, Manager (can be parallel)
4. WebAdmin

### Getting Help

- Check the [README.md](README.md) for project overview
- Review individual module `build.xml` files for module-specific targets
- Examine `server/mirth-build.properties` for version and configuration settings

## Additional Notes

- **No Root Build File:** The master build orchestration is in `server/mirth-build.xml`, not at the project root
- **JAR Signing:** Production builds sign JARs using keystore configuration in `keystore.properties`
- **Version Management:** Central version (4.5.2) is defined in `server/mirth-build.properties`
- **License Headers:** Source code uses MPL 2.0 license headers
- **Parallel Processing:** Manifest modification and JAR signing use multi-threading (4 threads by default)
