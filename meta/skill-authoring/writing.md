# Writing Content

How to write the prose and examples inside skill files so the AI gets
maximum value per token.

## Conciseness

The AI is already smart. Only add context it doesn't already have. Before
writing any sentence, ask:

- Does the AI really need this explanation?
- Can I assume it already knows this?
- Does this paragraph justify its token cost?

**Bad — verbose (≈150 tokens):**

> PDF (Portable Document Format) files are a common file format that contains
> text, images, and other content. To extract text from a PDF, you'll need to
> use a library. There are many libraries available for PDF processing, but
> pdfplumber is recommended because it's easy to use and handles most cases
> well. First, you'll need to install it using pip. Then you can use the code
> below...

**Good — concise (≈50 tokens):**

````markdown
## Extract PDF text

Use `pdfplumber`:

```python
import pdfplumber

with pdfplumber.open("file.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```
````

The concise version trusts the AI to know what a PDF is and how `pip install`
works. Delete anything a competent engineer would already know.

## Degrees of freedom

Match the specificity of your instructions to how fragile the task is.

**High freedom — text instructions.** Use when multiple approaches are valid
and context should drive the choice.

```markdown
## Code review process
1. Analyze structure and organization
2. Check for potential bugs and edge cases
3. Suggest readability improvements
4. Verify adherence to project conventions
```

**Medium freedom — templates with parameters.** Use when a preferred pattern
exists but variation is acceptable.

**Low freedom — exact commands.** Use when operations are fragile, sequence
matters, or consistency is critical.

```markdown
## Database migration
Run exactly this command — do not modify flags:

    python scripts/migrate.py --verify --backup
```

Think of the AI as a robot crossing terrain: a narrow bridge needs guardrails
(low freedom), an open field needs only a direction (high freedom).

## Consistent terminology

Pick one word for each concept and use it throughout. Mixing synonyms makes
the AI second-guess whether they refer to the same thing.

- **Good:** always "endpoint", always "field", always "extract"
- **Bad:** mixing "endpoint" / "URL" / "route" / "path" for the same concept

## Concrete examples beat abstract descriptions

For anything output-shape-dependent (commit messages, report formats, code
style), include input/output examples. Examples convey style faster than
prose ever will.

````markdown
## Commit message format

**Input:** Added JWT authentication
**Output:**
```
feat(auth): implement JWT-based authentication

Add login endpoint and token validation middleware
```

**Input:** Fixed date display bug in reports
**Output:**
```
fix(reports): correct date formatting in timezone conversion

Use UTC timestamps consistently across report generation
```
````

## Templates for structured output

When you need a specific output shape, provide a template. Match the
strictness level to your requirements.

**Strict** — use when the format is non-negotiable:

> ALWAYS use this exact template structure: [template]

**Flexible** — use when adaptation adds value:

> Here is a sensible default — adjust sections based on what you discover:
> [template]

## Workflows for multi-step tasks

For procedures with more than a few steps, provide a checklist the AI can
copy and check off as it progresses. Clear sequencing prevents the AI from
skipping validation steps.

````markdown
## PDF form filling workflow

```
- [ ] Step 1: Analyze the form (run analyze_form.py)
- [ ] Step 2: Create field mapping (edit fields.json)
- [ ] Step 3: Validate mapping (run validate_fields.py)
- [ ] Step 4: Fill the form (run fill_form.py)
- [ ] Step 5: Verify output (run verify_output.py)
```
````

## Feedback loops

For quality-critical tasks, build a validate → fix → retry loop into the
workflow itself. The "validator" can be a script or a reference document the
AI checks its output against.

```markdown
1. Draft the content
2. Check against STYLE_GUIDE.md
3. If issues found, revise and re-check
4. Only proceed when all checks pass
```

## Avoid time-sensitive information

Don't write things that become wrong with time. Migrate deprecated content
into a collapsible "legacy" section instead.

**Bad:**
> Before August 2025, use the old API. After August 2025, use the new API.

**Good:**
````markdown
## Current method

Use the v2 endpoint: `api.example.com/v2/messages`

## Legacy patterns

<details>
<summary>v1 API (deprecated 2025-08)</summary>

v1 used `api.example.com/v1/messages` and is no longer supported.
</details>
````
