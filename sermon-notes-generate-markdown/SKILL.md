---
name: sermon-notes-generate-markdown
description: Convert a pastor's sermon notes (.md, .doc, .docx, .eml) into a formatted final markdown file following the sermon notes template. Use when the user invokes /sermon-notes-generate-markdown.
disable-model-invocation: true
allowed-tools: Bash, Read, Write
---

# Sermon Notes → Markdown

Convert a pastor's sermon notes into a clean, formatted final markdown file.

## Step 1: Get the file path

If $ARGUMENTS is provided, use it as the file path. Otherwise, ask the user:

> Please provide the full path to the pastor's sermon notes file. Supported formats: `.md`, `.doc`, `.docx`, `.eml`

Validate:
- File must exist
- Extension must be `.md`, `.doc`, `.docx`, or `.eml`
- If invalid, tell the user and stop

## Step 2: Extract the content

**If `.md`**: Read the file directly.

**If `.doc` or `.docx`**: Run the following to extract text on macOS:

```bash
textutil -convert txt -stdout "<file_path>"
```

If `textutil` fails, inform the user and suggest installing `pandoc` via `brew install pandoc`, then use:
```bash
pandoc "<file_path>" -t plain
```

**If `.eml`**: Extract only the plain-text message body using Python's built-in `email` module (handles `quoted-printable` decoding automatically). Save the result as `<basename>.md` in the same directory, then use that `.md` file for all subsequent steps.

```bash
python3 - << 'EOF'
import email, sys

eml_path = "<file_path>"
md_path = "<same_directory>/<basename>.md"

with open(eml_path, 'rb') as f:
    msg = email.message_from_bytes(f.read())

body = None
for part in msg.walk():
    if part.get_content_type() == 'text/plain':
        body = part.get_payload(decode=True).decode('utf-8', errors='replace')
        break

if body is None:
    print("ERROR: No text/plain part found in EML", file=sys.stderr)
    sys.exit(1)

with open(md_path, 'w', encoding='utf-8') as f:
    f.write(body)

print(f"Extracted body to: {md_path}")
EOF
```

After extraction, read the resulting `.md` file and proceed with Step 3 using `<basename>.md` as the source.

## Step 3: Build the final file

Determine the output filename from the input file's base name (without extension), appended with `-final.md`. Example: `johns-sermon.docx` → `johns-sermon-final.md`. Place it in the same directory as the input file.

The output file must follow this exact structure:

```
# Overview

<numbered list of the pastor's main points starting at 1, with indented numbered sub-points as needed (separate set of numbers starting at 1). Omit all bible verse references from this section.>

# Outline

<all notes, formatted per the rules below>
```

## Step 4: Formatting rules for the `# Outline` section

Apply ALL of the following rules to the content:

### 4a. Main points → `#` heading
- Lines that are main points get a single `#` heading.
- It should be in the format `# <<bullet point number>>. <<main point>>`
- each subsequent main point increments bullet point number
- Insert one blank line after the heading line.

### 4b. Sub-points → `##` heading
- Lines that are sub-points get a `##` heading.
- It should be in the format `## <<bullet point number>>. <<main point>>`
- each subsequent sub main point increments bullet point number
- Insert one blank line after the heading line.

### 4c. Bible verses
Format every verse block as:

```
**VERSE NAME**
verse text here

```

- The verse reference goes on its own line in bold (e.g., `**John 3:16**`).
- The verse text goes on the next line, plain.
- Always follow with one blank line.
- if the verse is multiple lines, with blank lines in between, please remove those blank lines ie like poetry.

### 4d. Standalone bold numbers
If a number is bolded **by itself** — i.e., `**8**` with no surrounding words — remove the bold markers so it becomes plain `8`.
This does NOT apply when the number is part of a longer bold phrase, e.g., `**8 For God so loved**` stays bold.

### 4e. Replace underscores with asterisks
Replace every `_` character with `*` throughout the entire document.

### 4f. No leading whitespace on content lines
Every line that contains words or numbers must start at column 1. Strip any leading spaces or tabs.

## Step 5: Write the output file

Write the formatted content to `<<basename>>-final.md` in the same directory as the source file.

Confirm to the user: output file path and a brief summary (e.g., number of main points found).

## Step 6: display the output file
Run cat command on the `<<basename>>-final.md` in the claude terminal, so the user has an easy access point to copy and paste this to clipboard, so we can put this into the final sermon notes.

Tell the user: to copy the output to clipboard, press **CTRL-O** in the Claude terminal.