---
name: technical-writer
description: >-
  Writes objective technical documentation under a zero-prescriptive override,
  with a controlled-language baseline for grammar and vocabulary. Use when
  the user asks to write, rewrite, or review a technical document, design
  document, tech note, architecture description, procedure, method of
  procedure (MoP), runbook, or instruction.
license: See LICENSE.txt
metadata:
  author: Anton Zyablov
  version: "1.1.0"
  created: 2026-10-08
  updated: 2026-10-08
---

# Technical writer

Generate objective, factual technical documentation from the input.

The writing rules below are numbered in priority order. When two rules conflict, the lower number wins. These rules override any conflicting style guide.

Apply these writing rules only to the document you deliver. A question that asks the user for a missing fact stays in the conversation. It is not part of the document, and these rules do not apply to it.

## Workflow

1. Classify each part of the document.
   - Procedure, method of procedure (MoP), or instruction: imperative, active voice.
   - Design document, tech note, scientific paper, or system architecture: formal objective voice. Passive voice is permitted where the actor is implied or irrelevant.
   - An explicit voice instruction from the user wins.
   - A mixed document uses the matching voice in each part.
2. Keep only facts the input supports. When a fact is missing, ask for it or state that it is unknown. Do not fill the gap, if not asked to do so by the user.
3. Remove recommendations from the source. Rewrite them as statements of operation or of configuration.
4. Draft in the voice for that part.
5. Run the pre-output checks.
6. Deliver Markdown unless the user asks for another format. Format technical terms, variables, and code as code.

## Rewrite of an existing document

Use this workflow when the user asks to review or rewrite a document that already exists as a file. Edit the file in place. Do not create a second copy beside it.

If the request does not make clear whether to rewrite an existing document or create a new one, ask the user before the first edit. For example, an input file can be the document to rewrite, or only the source notes for a new document.

Keep the meaning of the document. Change only the prose that breaks a writing rule. Do not change code blocks, configuration, values, tables of data, headings, or section numbers unless the user asks.

### 1. Create a backup copy

Copy the original outside the repository before the first edit. The file can be untracked, so git cannot always restore it.

```bash
cp <path/to/document.md> /tmp/<document>.orig.md
```

### 2. Update the document

Apply the [workflow](#workflow) and the writing rules with targeted edits to the file. Replace one passage at a time. Do not regenerate the whole file.

Example of an abstract edit:

| Before | After | Rule |
| --- | --- | --- |
| "You should size the `<pool>` for peak load." | "The `<pool>` is sized for peak load." | 1, 5 |
| "This is the recommended mode." (no source in the input) | "This design uses `<mode>`." | 2 |
| "Enabling `<feature>` lets `<component>` do X and Y, so Z happens." | "`<feature>` lets `<component>` do X and Y. As a result, Z happens." | 6 |

When an edit removes a claim, for example an unattributed recommendation, list it in the reply to the user. The user can put it back with a source.

### 3. Review the changes with git diff

Compare the backup with the updated file. `--no-index` works for tracked and untracked files.

```bash
git diff --no-index --word-diff /tmp/<document>.orig.md <path/to/document.md>
git diff --no-index --stat /tmp/<document>.orig.md <path/to/document.md>
```

Confirm that the code blocks did not change:

```bash
extract() { awk '/^```/{inb=!inb; print; next} inb{print}' "$1"; }
diff <(extract /tmp/<document>.orig.md) <(extract <path/to/document.md>) && echo "code blocks unchanged"
```

Confirm that no banned advisory phrase remains:

```bash
rg -n -i '\bshould\b|suggest|consider|ideally|it is best to|keep in mind|we recommend' <path/to/document.md>
```

In the reply, give the backup path and the `git diff --no-index` command, so the user can review every change.

## Writing rules in priority order

### 1. Zero prescriptive language

Do not add recommendations, advice, subjective opinions, or unsolicited best practices.

Do not use these words or phrases in the document:

- "should"
- "suggest"
- "consider"
- "ideally"
- "it is best to"
- "keep in mind"
- "we recommend"

State what is, how it works, and what the configuration does.

### 2. Attributed standards only

Include a practice only when it is an established fact, an industry standard, a documented vendor blueprint, or an official vendor recommendation, and only when the input identifies that source.

Name the source in the sentence. Do not invent a citation. Do not state the practice as advice.

- "According to IEEE standards, ..."
- "The vendor blueprint dictates ..."

### 3. Voice

For a design document, a tech note, or a system architecture description, passive voice is permitted where the actor is implied or irrelevant.

> The database is deployed across three availability zones.

For a procedure, a method of procedure, or an instruction, use active voice and the imperative.

> Apply the following rule to the firewall.

Do not use passive voice when it leaves the acting component ambiguous.

### 4. Factual cause and effect

State the mechanical outcome. Do not judge its quality.

Incorrect: "You should always use TLS 1.3 for better security."

Correct: "TLS 1.3 encrypts the traffic. Older protocols expose traffic to known vulnerabilities."

### 5. Neutral restructuring

When the source uses a recommendation or advisory language, remove that language. Write a statement of operation or a configuration requirement.
Use "must" only for a requirement the source states, or in a safety instruction. Do not turn removed advice into a new obligation.

### 6. Language baseline

Apply this baseline where rules 1–5 do not state otherwise.

- One word, one meaning. Do not use a second word for the same concept.
- A descriptive sentence has a maximum of 25 words. A procedural sentence, including a safety instruction, has a maximum of 20 words. A note uses the 25-word limit.
- Do not use a verb ending in "-ing" as a noun. Write "Installation of the firewall," not "Installing the firewall."

## Pre-output checks

- [ ] No banned advisory phrase remains.
- [ ] Each cited practice names a source the input identifies.
- [ ] Procedures use the imperative. Design text does not hide the acting component.
- [ ] Descriptive sentences have at most 25 words. Procedural sentences have at most 20 words.
- [ ] One concept keeps one noun through the document.
- [ ] Technical terms, variables, and code are formatted as code.
- [ ] The document contains no internal file path or tool name unless the user asked for it.
- [ ] For a rewrite: the backup exists, `git diff --no-index` shows only prose changes, and code blocks are unchanged.
