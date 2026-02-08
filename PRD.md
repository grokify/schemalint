# Product Requirements Document: schemalint

## Executive Summary

**schemalint** is a JSON Schema linter for static type compatibility. It validates JSON Schema files for patterns that cause problems when generating code for statically-typed languages like Go, Rust, and TypeScript.

## Problem Statement

### Current State

JSON Schema supports dynamic patterns that don't map cleanly to statically-typed languages:

| JSON Schema Pattern | Problem for Static Types |
|---------------------|-------------------------|
| `anyOf: [A, B]` without discriminator | No way to determine type at compile time |
| `additionalProperties: true` | Cannot define struct fields |
| `type: ["string", "number"]` | Mixed types require interface{}/any |
| Missing `type` field | Ambiguous type inference |

### Impact

- **Code generation fails**: Generators produce `interface{}` or `any` types
- **Type safety lost**: Runtime errors instead of compile-time errors
- **Poor developer experience**: No IDE autocomplete or type checking
- **Maintenance burden**: Manual type assertions everywhere

### Root Cause

JSON Schema authors often don't consider the constraints of statically-typed languages. Without validation, schemas pass linting but produce unusable generated code.

## Target Users

1. **Schema Authors**: Designing JSON Schemas for APIs and specifications
2. **SDK Authors**: Building typed clients for APIs defined by JSON Schema
3. **API Developers**: Generating server/client code from schemas
4. **CI/CD Pipelines**: Validating schemas before code generation

## Goals

### Primary Goals

1. **Detect Problematic Patterns**: Identify union types lacking discriminators
2. **Enforce Type Compatibility**: Require explicit types and disallow mixed types (scale profile)
3. **Multiple Output Formats**: Support text, JSON, and GitHub Actions annotations
4. **Configurable Profiles**: Default profile for warnings, scale profile for strict enforcement

### Secondary Goals

1. **Clear Suggestions**: Provide actionable fix suggestions for each issue
2. **CI Integration**: Exit codes for scripting and GitHub annotations
3. **Extensibility**: Support additional profiles for different target languages

### Non-Goals (v1)

1. Code generation (separate tool)
2. JSON Schema validation (use existing validators)
3. Runtime validation

## Functional Requirements

### FR-1: Lint Checks (Default Profile)

| ID | Requirement | Priority | Status |
|----|-------------|----------|--------|
| FR-1.1 | Detect unions without discriminator fields | High | ✅ v0.1.0 |
| FR-1.2 | Detect inconsistent discriminator field names | High | ✅ v0.1.0 |
| FR-1.3 | Detect missing const values in union variants | High | ✅ v0.1.0 |
| FR-1.4 | Warn on large unions (>10 variants) | Medium | ✅ v0.1.0 |
| FR-1.5 | Warn on deeply nested unions (>2 levels) | Medium | ✅ v0.1.0 |
| FR-1.6 | Warn on additionalProperties in union variants | Medium | ✅ v0.1.0 |

### FR-2: Lint Checks (Scale Profile)

| ID | Requirement | Priority | Status |
|----|-------------|----------|--------|
| FR-2.1 | Disallow anyOf/oneOf/allOf composition | High | ✅ v0.2.0 |
| FR-2.2 | Disallow additionalProperties: true | High | ✅ v0.2.0 |
| FR-2.3 | Require explicit type field | High | ✅ v0.2.0 |
| FR-2.4 | Disallow mixed type arrays | High | ✅ v0.2.0 |

### FR-3: Pattern Detection

| ID | Requirement | Priority | Status |
|----|-------------|----------|--------|
| FR-3.1 | Detect nullable patterns: `anyOf: [T, null]` | High | ✅ v0.1.0 |
| FR-3.2 | Detect reference patterns | High | ✅ v0.1.0 |
| FR-3.3 | Skip all-$ref unions (require resolution) | Medium | ✅ v0.1.0 |

### FR-4: CLI Interface

| ID | Requirement | Priority | Status |
|----|-------------|----------|--------|
| FR-4.1 | `schemalint lint` command | High | ✅ v0.1.0 |
| FR-4.2 | `--profile` flag for profile selection | High | ✅ v0.2.0 |
| FR-4.3 | `--output` flag for format selection | High | ✅ v0.1.0 |
| FR-4.4 | Exit codes (0=ok, 1=errors, 2=warnings) | High | ✅ v0.1.0 |
| FR-4.5 | `schemalint version` command | Medium | ✅ v0.1.0 |

## Non-Functional Requirements

### NFR-1: Performance

- Lint 10,000+ line schemas in under 1 second
- Handle schemas with 100+ type definitions

### NFR-2: Usability

- Clear, actionable error messages
- Suggestions for fixing each issue
- GitHub Actions integration

### NFR-3: Distribution

- Cross-platform binaries (linux, darwin, windows)
- Homebrew tap for easy installation
- Go install support

## Success Metrics

1. **Adoption**: Used in CI pipelines for major JSON Schema projects
2. **Issue Detection**: Catches 100% of union discriminator issues
3. **Developer Experience**: <5 minutes from install to first lint

## Milestones

| Version | Scope | Status |
|---------|-------|--------|
| v0.1.0 | Schema linter with union detection | ✅ Complete |
| v0.2.0 | Scale profile, Homebrew support | ✅ Complete |
| v0.3.0 | Configuration file, additional checks | 🔲 Planned |
| v1.0.0 | Production ready | 🔲 Planned |

## References

- [JSON Schema Draft 2020-12](https://json-schema.org/draft/2020-12/json-schema-core)
- [Go JSON Codegen Challenges](https://pkg.go.dev/encoding/json)
