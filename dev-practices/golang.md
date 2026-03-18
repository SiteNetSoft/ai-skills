# Go Best Practices

Guidelines for writing idiomatic, clean, and maintainable Go code. Based on official Go
documentation and Google's Go Style Guide.

## Formatting

- Run `gofmt` (or `goimports`) on all code — no exceptions
- Use **tabs** for indentation (enforced by `gofmt`)
- No fixed line length limit — refactor long lines rather than wrapping arbitrarily
- Opening brace on the **same line** as the control statement (enforced by semicolon insertion)
- No parentheses around `if`, `for`, `switch` conditions

## Naming Conventions

### General Rules
- `MixedCaps` for exported names, `mixedCaps` for unexported — **never** underscores
- Short names for narrow scope, longer names for wider scope
- Names should not repeat context already provided by the package

### Packages
- Single lowercase word, no underscores or mixedCaps
- Short, concise — users type them frequently
- Avoid generic names: `util`, `helper`, `common`, `misc`, `api`, `types`
- Don't repeat the package name in exported identifiers: `bufio.Reader` not `bufio.BufReader`

### Functions and Methods
- Noun-like names for functions returning values: `JobName()` not `GetJobName()`
- Verb-like names for functions performing actions: `WriteDetail()`
- No `Get` prefix on getters: `Owner()` not `GetOwner()`, but `SetOwner()` for setters
- Omit types and receiver info from names: `Parse()` not `ParseYAMLConfig()`

### Variables
- Single-letter variables for loop indices: `i`, `j`, `k`
- Short names in limited scope: `c` over `lineCount`
- Descriptive names for globals and package-level variables

### Receiver Names
- 1-2 letter abbreviation of the type: `c` for `Client`, `s` for `Server`
- **Never** `me`, `this`, or `self`
- Be consistent across all methods of a type

### Interfaces
- One-method interfaces: method name + `-er` suffix: `Reader`, `Writer`, `Closer`
- Define interfaces in the **consumer** package, not the implementor
- Return concrete types; accept interfaces

### Initialisms
- Consistent casing for acronyms: `URL` or `url`, never `Url`
- `ID` not `Id`, `HTTP` not `Http`, `ServeHTTP` not `ServeHttp`

## Error Handling

- Always check error returns — never discard with `_`
- Error as the **last** return value
- Return early on error — keep the happy path at minimal indentation:
  ```go
  if err != nil {
      return err
  }
  // normal flow continues
  ```
- Error strings: **not** capitalized, **no** trailing punctuation:
  ```go
  fmt.Errorf("something bad")  // good
  fmt.Errorf("Something bad.") // bad
  ```
- Use `%w` in `fmt.Errorf` when callers need `errors.Is()`/`errors.As()`; use `%v` otherwise
- Use sentinel errors (`var ErrNotFound = errors.New(...)`) for simple cases
- Use custom error types when callers need structured information
- Don't panic for normal errors — panic only for truly unrecoverable states or API misuse
- Never let panics escape package boundaries; use `defer`/`recover` at public API edges
- Libraries return errors; only `main` or top-level handlers should decide to log or exit

## Comments and Documentation

- Doc comments on **all** exported names — full sentences starting with the identifier name:
  ```go
  // Request represents a request to run a command.
  type Request struct { ... }
  ```
- Package comment adjacent to `package` clause (no blank line)
- Comments explain **why**, not **what** — skip obvious restatements
- Line comments (`//`) are the norm; block comments (`/* */`) for package headers or disabling code

## Imports

- Group with blank lines: stdlib first, then external:
  ```go
  import (
      "fmt"
      "os"

      "github.com/foo/bar"
  )
  ```
- Use `goimports` to manage import ordering automatically
- Avoid renaming imports unless there's a name collision
- Blank imports (`import _ "pkg"`) only for side effects, only in `main` or tests
- Dot imports (`import . "pkg"`) only to break circular deps in tests

## Functions and Methods

- Accept `context.Context` as the **first** parameter — never put it in a struct
- Prefer synchronous functions — let callers add concurrency
- Use named return values only when they improve clarity in godoc
- Naked returns only in very short functions
- Use option structs for many configuration parameters:
  ```go
  type Options struct {
      Timeout  time.Duration
      Retries  int
  }
  ```
- Or functional options when most callers need no config

### Pointer vs Value Receivers
- Pointer: if the method mutates, has `sync.Mutex`, or the struct is large
- Value: for small immutable structs, basic types
- **Don't mix** receiver types on a single type

## Concurrency

- "Do not communicate by sharing memory; share memory by communicating"
- Use channels for synchronization and data passing
- Specify channel direction in function signatures: `func sum(values <-chan int)`
- Make goroutine lifetimes obvious — document when/why they exit
- Never call `t.Fatal()` from spawned goroutines in tests
- Use buffered channels as semaphores for rate limiting
- Prefer `select` for multiplexing channels

## Data Structures

- Prefer `var t []string` (nil slice) over `t := []string{}` unless JSON encoding requires `[]`
- Use `make` with capacity hints when final size is known
- Maps and slices are reference types — modifications visible to callers
- Don't copy structs containing `sync.Mutex` or pointer-backed fields that alias

## Testing

- Keep assertions in the test function — avoid assertion helper libraries
- Table-driven tests for repetitive cases
- Failure messages should show: input, got, want:
  ```go
  t.Errorf("Foo(%q) = %d; want %d", tt.in, got, tt.want)
  ```
- Call `t.Helper()` in setup/cleanup functions
- Use real transports with test-double backends, not hand-rolled clients
- Test packages named with `test` suffix: `creditcardtest`
- Use `t.Error()` + `continue` without subtests; `t.Fatal()` inside `t.Run()`

## Linting

Use [`golangci-lint`](https://golangci-lint.run/) as the standard linter runner. Recommended
configuration:

### Default Linters (always enabled)
- **errcheck** — unchecked error returns
- **govet** — suspicious constructs (`go vet`)
- **staticcheck** — comprehensive static analysis
- **unused** — unused constants, variables, functions, types
- **ineffassign** — assignments to variables that are never read

### Recommended Additional Linters
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

### Example `.golangci.yml`
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

## Key Principles

1. **Clarity over cleverness** — code is read far more than written
2. **Simplicity** — use the least mechanism needed
3. **Zero values should be useful** — design types so uninitialized values work
4. **Composition over inheritance** — embedding, not subclassing
5. **Small interfaces** — one or two methods is ideal
6. **Handle errors explicitly** — always
7. **Local consistency** — match surrounding code style when the guide is silent

## Sources

- [Effective Go](https://go.dev/doc/effective_go)
- [Go Code Review Comments](https://go.dev/wiki/CodeReviewComments)
- [Google Go Style Guide](https://google.github.io/styleguide/go/guide)
- [Google Go Best Practices](https://google.github.io/styleguide/go/best-practices)
- [golangci-lint](https://golangci-lint.run/)
