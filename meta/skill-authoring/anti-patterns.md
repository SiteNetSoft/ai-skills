# Anti-Patterns

Common mistakes when writing skill files. Each one either wastes context
budget, confuses the AI, or ages badly.

## Explaining what the AI already knows

> PDFs are a file format for documents. To work with them, you'll need a
> library...

Assume general knowledge. Only add what the AI wouldn't know from training —
project-specific conventions, undocumented gotchas, non-obvious decisions.

## Deeply nested references

`index.md` → `advanced.md` → `details.md` → actual information.

The AI may partially read files when following reference chains (e.g. only
the first 100 lines), so content buried two hops deep may be silently
missed. **All sub-files should be linked directly from `index.md`.**

## Offering too many options without a default

> You can use pdfplumber, or PyMuPDF, or pypdf, or pdf2image, or...

Pick one and recommend it. If alternatives matter, present them as escape
hatches for specific edge cases:

> Use `pdfplumber` for text extraction. For scanned PDFs requiring OCR, use
> `pdf2image` with `pytesseract` instead.

## Windows-style paths

`scripts\helper.py` breaks on Unix. Always use forward slashes — they work
on every platform.

## Time-sensitive language

> After August 2025, switch to the new API.
> We're currently migrating away from X.
> The team is planning to adopt Y next quarter.

These phrases rot. Use a "legacy patterns" section for deprecated content
and convert relative dates to absolute ones (`2025-08-01`, not "next
August").

## Inconsistent terminology

Calling the same concept an "endpoint", "URL", "route", and "path" in
different sections forces the AI to decide whether they're the same thing.
Pick one term per concept.

## Vague names

Skill names like `helper`, `utils`, `tools`, `documents`, or `data` tell the
AI nothing about when to use them. Prefer specific, action-oriented names:
`processing-pdfs`, `analyzing-spreadsheets`, `writing-commit-messages`.

## Vague descriptions

> description: Helps with documents
> description: Processes data
> description: Does stuff with files

A description that doesn't tell the AI **when** to use the skill is dead
weight. Include both what it does and what triggers its use:

> Extract text and tables from PDF files, fill forms, merge documents. Use
> when working with PDF files or when the user mentions PDFs, forms, or
> document extraction.

## First-person or second-person descriptions

> description: I can help you process Excel files
> description: You can use this to process Excel files

Descriptions are injected into the system prompt, where mixed point-of-view
causes discovery problems. **Always write in third person:**

> description: Processes Excel files and generates reports

## Punting errors back to the AI (for skills with scripts)

```python
def process_file(path):
    return open(path).read()  # Fails, AI figures it out
```

If the script can handle a failure mode, it should. Explicit error handling
with useful messages is cheaper than round-tripping through the AI.

## Magic numbers without justification

```python
TIMEOUT = 47  # why 47?
MAX_RETRIES = 5  # why 5?
```

If you don't know the right value, the AI won't either. Document the
reasoning so future edits can reason about tradeoffs:

```python
# HTTP requests typically complete within 30s; longer accounts for slow links
REQUEST_TIMEOUT = 30

# Three retries balances reliability vs latency; most transient failures
# resolve by the second attempt
MAX_RETRIES = 3
```

## Massive single files

When a skill grows past ~400 lines, the AI loads all of it even when only
10% is relevant. Split into a skill graph and let progressive disclosure do
the work.

## Duplicating content across sub-files

If the same paragraph appears in three sub-files, extract it into `index.md`
(if it's universally relevant) or a dedicated sub-file that others link to.
Duplication means the AI pays the token cost multiple times per session.
