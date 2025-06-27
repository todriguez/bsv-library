# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is the BSV Library documentation repository, which contains comprehensive documentation for the BSV blockchain ecosystem. It is part of a larger monorepo (`bsv-academy-monorepo1`) and integrates content from multiple sources:

- **BSV Skills Center**: Core BSV documentation
- **BSV Academy**: Educational content and courses
- **Onboarding Content**: Structured learning pathways

The documentation is structured as a GitBook project, focused on providing technical documentation, guides, and learning resources for BSV blockchain developers and users.

## Development Commands

Since this is primarily a documentation repository using GitBook, there are no traditional build/test commands. The main workflows are:

### Content Management
- Documentation files are in Markdown format
- Primary navigation is defined in `SUMMARY.md`
- Content is organized in logical directories matching the navigation structure

### Monorepo Context
- This package is located at `packages/onboarding/bsv-library/` within the monorepo
- The parent monorepo uses pnpm workspaces
- To run commands from the monorepo root: `pnpm --filter <package-name> <command>`

## Architecture and Structure

### Content Organization

1. **BSV Skills Center** (`bsv-skills-center/`)
   - Protocol documentation
   - Network policies
   - Node operations
   - Privacy and security

2. **Academy Content** (`academy/` and `bsv-academy/`)
   - BSV Theory
   - Protocol and Design
   - Enterprise guides
   - Technical primitives (Hash Functions, Merkle Trees, Digital Signatures)

3. **Onboarding Content** (`onboarding/`)
   - Learning pathways (Technical, Business, Academic, Enterprise)
   - Getting started guides
   - Resources and tools
   - Assessment frameworks

4. **Guides** (`guides/`)
   - SDK documentation (TypeScript, Go, Python)
   - Business use cases
   - Local development setup

5. **Network Topology** (`network-topology/`)
   - Node configurations (SV Node, Teranode)
   - SPV Wallet documentation
   - Overlay services

### Key Files

- `SUMMARY.md`: Main navigation structure for GitBook
- `README.md`: Welcome page and entry point
- Individual pathway READMEs provide structured learning paths

### Integration Points

The repository integrates content from:
- Original BSV Skills Center documentation
- BSV Academy educational materials
- Onboarding repository content
- sCrypt documentation

## Important Considerations

1. **Documentation Focus**: This is a documentation-only repository. No application code or build processes.

2. **GitBook Structure**: Content follows GitBook conventions:
   - Hierarchical navigation in SUMMARY.md
   - Markdown files with specific linking patterns
   - Supports nested content structures

3. **Content Migration**: The onboarding content has been migrated from a separate repository and integrated into the unified structure.

4. **Navigation**: Use relative links between documents and maintain the structure defined in SUMMARY.md.

5. **Pathway-Based Learning**: Content is organized around learning pathways (Technical, Business, Academic, Enterprise) to guide different user types.