# Linting

Use [`golangci-lint`](https://golangci-lint.run/) as the standard linter runner.

## Default Linters (always enabled)
- **errcheck** — unchecked error returns
- **govet** — suspicious constructs (`go vet`)
- **staticcheck** — comprehensive static analysis
- **unused** — unused constants, variables, functions, types
- **ineffassign** — assignments to variables that are never read

## Recommended Additional Linters
- **gosec** — security vulnerabilities
- **gocritic** — bugs, performance, and style
- **revive** — configurable replacement for `golint`
- **misspell** — spelling mistakes in comments and strings
- **unconvert** — unnecessary type conversions
- **unparam** — unused function parameters
- **gocyclo** — cyclomatic complexity
- **contextcheck** — proper context propagation
- **nolintlint** — well-formed `nolint` directives
- **modernize** — Go code modernization patterns

## Example `.golangci.yml`

```yaml
linters:
  enable:
    - errcheck
    - govet
    - staticcheck
    - unused
    - ineffassign
    - gosec
    - gocritic
    - revive
    - misspell
    - unconvert
    - unparam
    - gocyclo
    - contextcheck
    - nolintlint

linters-settings:
  gocyclo:
    min-complexity: 15
  revive:
    rules:
      - name: exported
      - name: var-naming
      - name: blank-imports
      - name: context-as-argument
      - name: error-return
      - name: error-strings
      - name: error-naming
      - name: increment-decrement
      - name: range
      - name: receiver-naming
      - name: indent-error-flow
      - name: superfluous-else
      - name: unreachable-code
```
