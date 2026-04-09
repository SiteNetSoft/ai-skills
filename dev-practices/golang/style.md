# Formatting, Naming & Imports

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
