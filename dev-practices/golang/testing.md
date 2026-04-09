# Testing

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
