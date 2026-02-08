# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Heimdall is an AWS security scanner that discovers IAM privilege escalation paths. It detects 50+ IAM privilege escalation patterns, 85+ attack chain patterns with MITRE ATT&CK mapping, and analyzes 10 AWS services for cross-service escalation.

## Development Commands

```bash
# Install in development mode
pip install -e .

# Install with dev dependencies
pip install -e ".[dev]"

# Run linting
ruff check heimdall/

# Run tests
pytest

# Run specific test file
pytest tests/test_example.py

# Run tests with coverage
pytest --cov=heimdall
```

## CLI Usage

```bash
# Main commands
heimdall dashboard              # One-command security overview
heimdall iam scan               # Scan AWS account, build trust graph
heimdall iam detect-privesc     # Find privilege escalation paths
heimdall iam attack-chain       # Analyze multi-step attack chains
heimdall iam cross-service      # Cross-service escalation (10 services)
heimdall iam tui                # Interactive terminal UI

# Terraform scanning
heimdall terraform scan plan.json              # Scan Terraform plan
heimdall terraform scan plan.json --fail-on critical  # CI/CD gate
```

## Architecture

### Core Modules

- **heimdall/cli.py** - Main CLI entry point using Click framework. All commands flow through here.
- **heimdall/commands/** - Individual command implementations (attack_chain, cross_service, diff, privesc, etc.)
- **heimdall/iam/** - IAM scanning and analysis:
  - `scanner.py` - Core IAM scanning logic
  - `privesc_patterns.py` - 50+ privilege escalation pattern definitions
  - `policy_resolver.py` - IAM policy resolution
  - `scp_resolver.py` - Service Control Policy handling
- **heimdall/graph/** - Graph-based attack path analysis:
  - `builder.py` - Builds NetworkX trust graph from IAM data
  - `analyzer.py` - Path analysis algorithms
  - `graph_intelligence.py` - Advanced graph analytics (critical paths, betweenness centrality, SCP simulation)
- **heimdall/attack_chain/** - Multi-step attack chain analysis:
  - `builder.py` - Constructs attack chains from findings
  - `schema.py` - Chain/step data models with MITRE ATT&CK mapping
- **heimdall/cross_service/** - Cross-service privilege escalation:
  - `scanner.py` - Orchestrates multi-service analysis
  - `analyzers/` - Per-service analyzers (S3, Lambda, EC2, KMS, Secrets Manager, RDS, STS, SNS, SQS, DynamoDB)
- **heimdall/terraform/** - Terraform plan security analysis:
  - `parser.py` - Parses terraform plan JSON
  - `analyzer.py` - Detects attack paths in planned changes
  - `patterns.py` - 45+ Terraform-specific attack patterns

### Key Data Flow

1. `IAMScanner` collects AWS IAM data (roles, users, policies, trust relationships)
2. `GraphBuilder` creates a NetworkX directed graph of trust relationships
3. `PathAnalyzer` finds paths between principals
4. `PrivescDetector` matches findings against `PRIVESC_PATTERNS`
5. `AttackChainBuilder` constructs multi-step chains with blast radius scoring
6. Output via Rich console, SARIF, CSV, or JSON

### Pattern System

Privilege escalation patterns are defined in `heimdall/iam/privesc_patterns.py` as `PrivescPattern` dataclasses with:
- Required IAM actions
- Severity level (CRITICAL/HIGH/MEDIUM/LOW)
- Method category (POLICY_MANIPULATION, PASSROLE_ABUSE, CREDENTIAL_ACCESS, etc.)
- Explanations and remediation guidance

Terraform patterns in `heimdall/terraform/patterns.py` use a similar `PatternDef` structure.

### UI Framework

- **Rich** - Terminal formatting, tables, progress bars
- **Textual** - TUI framework for interactive mode (`heimdall iam tui`)
- **Click** - CLI framework with command groups

## Code Style

- Line length: 100 characters
- Target Python version: 3.9+
- Linting: ruff
- Formatting: black
- Norse mythology theme in comments and variable names (Bifröst, Odin, etc.)
