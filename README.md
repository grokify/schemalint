# SchemaLint

[![Build Status][build-status-svg]][build-status-url]
[![Lint Status][lint-status-svg]][lint-status-url]
[![Go Report Card][goreport-svg]][goreport-url]
[![Docs][docs-godoc-svg]][docs-godoc-url]
[![License][license-svg]][license-url]

JSON Schema linter for static type compatibility.

## Overview

**schemalint** validates JSON Schema files for compatibility with statically-typed languages like Go, Rust, TypeScript, and others. It catches patterns that cause problems in code generation before they become runtime issues.

## Installation

### Homebrew

```bash
brew install grokify/tap/schemalint
```

### Go Install

```bash
go install github.com/grokify/schemalint/cmd/schemalint@latest
```

## Usage

### Generate Schema from Go Types

Generate a JSON Schema from Go struct types:

```bash
schemalint generate github.com/myorg/myproject/types Config
schemalint generate -o schema.json github.com/myorg/myproject/types Config
```

This creates a temporary Go program that uses [invopop/jsonschema](https://github.com/invopop/jsonschema) to reflect on your type and generate the schema. The target package can be local (in GOPATH/src) or remote.

### Lint Schema

Check a JSON Schema for patterns that cause problems in code generation:

```bash
schemalint lint schema.json
```

### Profiles

Use `--profile` to select a linting profile:

```bash
schemalint lint schema.json                  # default profile
schemalint lint schema.json --profile scale  # strict scale profile
```

| Profile | Description |
|---------|-------------|
| `default` | Standard checks for discriminators, union size, nesting |
| `scale` | Strict mode that disallows composition keywords for clean static types |

### Output Formats

```bash
schemalint lint --output text schema.json   # Human-readable (default)
schemalint lint --output json schema.json   # Machine-readable JSON
schemalint lint --output github schema.json # GitHub Actions annotations
```

### Exit Codes

| Code | Meaning |
|------|---------|
| 0 | No issues found |
| 1 | Errors found (schema has problems) |
| 2 | Warnings found but no errors |

## Lint Checks

### Default Profile

#### Errors

| Code | Description |
|------|-------------|
| `union-no-discriminator` | Union (`anyOf`/`oneOf`) has no discriminator field |
| `inconsistent-discriminator` | Variants use different discriminator field names |
| `missing-const` | Union variant lacks `const` value for discriminator |
| `duplicate-const-value` | Multiple variants have the same discriminator value |
| `invalid-property-case` | Property name does not follow the configured case convention |

#### Warnings

| Code | Description |
|------|-------------|
| `large-union` | Union has more than 10 variants |
| `nested-union` | Union nested more than 2 levels deep |
| `additional-properties` | Union variant has `additionalProperties: true` |

### Scale Profile

The scale profile includes all default checks plus these additional errors:

| Code | Description |
|------|-------------|
| `composition-disallowed` | Disallow `anyOf`, `oneOf`, `allOf` |
| `additional-properties-disallowed` | Disallow `additionalProperties: true` |
| `missing-type` | Require explicit `type` field |
| `mixed-type-disallowed` | Disallow type arrays like `["string", "number"]` |

## Example

Given this schema with a union that lacks a discriminator:

```json
{
  "$defs": {
    "Response": {
      "anyOf": [
        {"type": "object", "properties": {"data": {"type": "string"}}},
        {"type": "object", "properties": {"error": {"type": "string"}}}
      ]
    }
  }
}
```

Running `schemalint lint` will report:

```
[error] $/$defs/Response/anyOf: anyOf union has no discriminator field
  suggestion: Add a const property (e.g., 'type' or 'kind') to each variant with a unique value

Summary: 1 error(s), 0 warning(s)
```

Fix by adding a discriminator:

```json
{
  "$defs": {
    "Response": {
      "anyOf": [
        {
          "type": "object",
          "properties": {
            "type": {"const": "success"},
            "data": {"type": "string"}
          }
        },
        {
          "type": "object",
          "properties": {
            "type": {"const": "error"},
            "error": {"type": "string"}
          }
        }
      ]
    }
  }
}
```

## Roadmap

See [TASKS.md](TASKS.md) for planned features including:

- Code generation for Go, Rust, TypeScript
- Full `$ref` resolution
- OpenAPI 3.1 support

## References

- [PRD.md](PRD.md) - Product requirements
- [TRD.md](TRD.md) - Technical requirements
- [JSON Schema Draft 2020-12](https://json-schema.org/draft/2020-12/json-schema-core)

## License

MIT License - see [LICENSE](LICENSE) for details.

 [build-status-svg]: https://github.com/grokify/schemalint/actions/workflows/ci.yaml/badge.svg?branch=main
 [build-status-url]: https://github.com/grokify/schemalint/actions/workflows/ci.yaml
 [lint-status-svg]: https://github.com/grokify/schemalint/actions/workflows/lint.yaml/badge.svg?branch=main
 [lint-status-url]: https://github.com/grokify/schemalint/actions/workflows/lint.yaml
 [goreport-svg]: https://goreportcard.com/badge/github.com/grokify/schemalint
 [goreport-url]: https://goreportcard.com/report/github.com/grokify/schemalint
 [docs-godoc-svg]: https://pkg.go.dev/badge/github.com/grokify/schemalint
 [docs-godoc-url]: https://pkg.go.dev/github.com/grokify/schemalint
 [license-svg]: https://img.shields.io/badge/license-MIT-blue.svg
 [license-url]: https://github.com/grokify/schemalint/blob/main/LICENSE
