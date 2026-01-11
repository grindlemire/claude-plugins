---
name: mage
description: Run mage build commands for Go projects. Use when the user wants to run mage targets for building, testing, deploying, or other project automation tasks.
allowed-tools: Bash, Glob, Read
---

# Mage Build Commands

This skill helps run mage commands for Go projects. Mage is a make-like build tool using Go.

## First: Discover Available Targets

Before running mage commands, check what targets are available in the project:

```bash
# List all available mage targets
mage -l
```

If the project has a `magefiles/` directory, read the Go files there to understand what each target does:

```bash
ls magefiles/
```

Read the relevant `cmd*.go` files to understand the target's purpose and any arguments it accepts.

## Running Mage Commands

### Basic Usage

```bash
mage <target>              # Run a target
mage <target> <args>       # Run with arguments
mage -v <target>           # Verbose output
mage -h                    # Help
```

### Common Patterns

Most mage projects follow similar conventions:

```bash
mage build      # Build the project
mage run        # Run the development server
mage test       # Run tests
mage install    # Install dependencies
mage tidy       # Clean up (often runs go mod tidy)
mage deploy     # Deploy the application
mage clean      # Clean build artifacts
```

## Workflow

1. **Discover targets**: Run `mage -l` to see available commands
2. **Understand targets**: Read files in `magefiles/` for details on what each target does
3. **Run the target**: Execute `mage <target>` with any required arguments

## Notes

- Mage targets are defined as exported Go functions in the `magefiles/` directory
- Function doc comments describe what each target does
- Some targets accept arguments (check the function signature)
- Targets can have dependencies on other targets
