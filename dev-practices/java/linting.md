# Linting

Use a combination of [Checkstyle](https://checkstyle.sourceforge.io/), [SpotBugs](https://spotbugs.github.io/), and [Error Prone](https://errorprone.info/) for comprehensive static analysis.

## Checkstyle — Style Enforcement

Checkstyle enforces coding conventions at the syntax level (formatting, naming, import order).

### Recommended checks

```xml
<!-- checkstyle.xml -->
<module name="Checker">
  <module name="TreeWalker">
    <!-- Naming -->
    <module name="TypeName"/>
    <module name="MethodName"/>
    <module name="ConstantName"/>
    <module name="LocalVariableName"/>
    <module name="ParameterName"/>

    <!-- Imports -->
    <module name="AvoidStarImport"/>
    <module name="UnusedImports"/>
    <module name="ImportOrder">
      <property name="groups" value="java,javax,org,com"/>
      <property name="separated" value="true"/>
    </module>

    <!-- Whitespace & formatting -->
    <module name="WhitespaceAround"/>
    <module name="EmptyLineSeparator"/>
    <module name="LeftCurly"/>
    <module name="RightCurly"/>
    <module name="NeedBraces"/>

    <!-- Code quality -->
    <module name="MethodLength">
      <property name="max" value="40"/>
    </module>
    <module name="ParameterNumber">
      <property name="max" value="5"/>
    </module>
    <module name="CyclomaticComplexity">
      <property name="max" value="10"/>
    </module>
    <module name="MagicNumber"/>
    <module name="EqualsHashCode"/>
    <module name="FinalLocalVariable"/>

    <!-- Javadoc -->
    <module name="JavadocMethod">
      <property name="accessModifiers" value="public"/>
    </module>
  </module>

  <!-- File-level checks -->
  <module name="FileLength">
    <property name="max" value="500"/>
  </module>
  <module name="NewlineAtEndOfFile"/>
</module>
```

### Maven integration
```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-checkstyle-plugin</artifactId>
  <version>3.3.1</version>
  <configuration>
    <configLocation>checkstyle.xml</configLocation>
    <failsOnError>true</failsOnError>
  </configuration>
  <executions>
    <execution>
      <phase>validate</phase>
      <goals><goal>check</goal></goals>
    </execution>
  </executions>
</plugin>
```

---

## SpotBugs — Bug Pattern Detection

SpotBugs finds likely bugs: null dereferences, resource leaks, incorrect synchronization, bad API use.

### Maven integration
```xml
<plugin>
  <groupId>com.github.spotbugs</groupId>
  <artifactId>spotbugs-maven-plugin</artifactId>
  <version>4.8.3.0</version>
  <configuration>
    <effort>Max</effort>
    <threshold>Medium</threshold>
    <failOnError>true</failOnError>
  </configuration>
  <executions>
    <execution>
      <phase>verify</phase>
      <goals><goal>check</goal></goals>
    </execution>
  </executions>
</plugin>
```

### Key bug categories to enforce
| Category | Description |
|----------|-------------|
| `NP` | Null pointer dereferences |
| `RCN` | Redundant null checks |
| `REC` | Catch of Exception or Throwable |
| `OS` | Unclosed streams / resources |
| `IS` | Incorrect synchronization |
| `BC` | Questionable casts |

---

## Error Prone — Compile-Time Bug Detection

Error Prone integrates with `javac` and catches bugs as you compile — before tests run.

### Maven integration
```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-compiler-plugin</artifactId>
  <version>3.13.0</version>
  <configuration>
    <compilerArgs>
      <arg>-XDcompilePolicy=simple</arg>
      <arg>-Xplugin:ErrorProne</arg>
    </compilerArgs>
    <annotationProcessorPaths>
      <path>
        <groupId>com.google.errorprone</groupId>
        <artifactId>error_prone_core</artifactId>
        <version>2.26.1</version>
      </path>
    </annotationProcessorPaths>
  </configuration>
</plugin>
```

### High-value checks enabled by default
- `NullAway` — null safety (with `@Nullable` annotation)
- `MissingOverride` — missing `@Override`
- `EqualsIncompatibleType` — `equals()` called with incompatible types
- `CollectionIncompatibleType` — wrong type in `contains()` / `remove()`
- `FutureReturnValueIgnored` — ignored `Future` return values

---

## Summary: When Each Tool Catches Issues

| Tool | Phase | Catches |
|------|-------|---------|
| Error Prone | Compile | Logic bugs, API misuse |
| Checkstyle | Compile/CI | Style, naming, structure |
| SpotBugs | Verify/CI | Runtime bug patterns, leaks |
