---
name: technical-writer
description: >-
  Writes objective technical documentation under a zero-prescriptive override,
  with a controlled-language baseline for grammar and vocabulary. Use when
  the user asks to write, rewrite, or review a technical document, design
  document, tech note, architecture description, procedure, method of
  procedure (MoP), runbook, instruction, or how-to.
license: See LICENSE.txt
metadata:
  author: Anton Zyablov
  version: "1.2.0"
  created: 2026-10-08
  updated: 2026-10-13
---

# Technical writer

Generate objective, factual technical documentation from the input.

The writing rules below are numbered in priority order. When two rules conflict, the lower number wins. These rules override any conflicting style guide. Exception: formatting conventions of the project, for example in `CLAUDE.md` or `AGENTS.md`, win on formatting. An example is a ban on code formatting in customer documents.

**Backed claims.** A comparative or quality word, for example "scalable", "efficient", or "faster", states a fact when a number or a proven fact from the input backs it in the same sentence or the next one. Such a claim is allowed under every rule below. Example: "The new design converges faster. Failover time drops from 800 ms to 50 ms." Without that backing, remove the word or ask for the fact.

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
6. Deliver Markdown unless the user asks for another format. Format technical terms, variables, and code as code, unless a project convention forbids it.

## Rewrite of an existing document

Use this workflow when the user asks to review or rewrite a document that already exists as a file. Edit the file in place. Do not create a second copy beside it.

If the request does not make clear whether to rewrite an existing document or create a new one, ask the user before the first edit. For example, an input file can be the document to rewrite, or only the source notes for a new document.

Keep the meaning of the document. Change only the prose that breaks a writing rule. Do not change code blocks, configuration, values, tables of data, headings, or section numbers unless the user asks.

When the user says a passage is unclear, make the rewrite shorter than the original. Fix that passage only. Do not add new sections around it.

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
| "Configure the error counter threshold crossing alarm generation interval." | "Configure the interval between alarms when the error counter crosses its threshold." | 6 |
| "The design is highly scalable." (no fact in the input) | "The design supports `<N>` sessions per node." or ask for the fact | Backed claims, 4 |

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

Confirm that no quality judgement, filler, or idiom remains. The pattern list is `banned-terms.txt` in the directory of this skill. The command removes comment lines and matches whole words only:

```bash
rg -n -i -w -f <(grep -v '^#' <skill-dir>/banned-terms.txt) <path/to/document.md>
```

Review each match. A word backed by a number or a proven fact in the same or the next sentence stays, see *Backed claims*. A product name or an established industry term stays.

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

Name the exact part of the source: the RFC section, or the vendor guide and its release. Example: "RFC 8476, Section 3". Do not cite working drafts or documents that can change or disappear, if not asked to do so by the user.

The sentence states the fact itself. The reference supports the fact. It does not replace it.

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

Words that judge or inflate need proof. Use one only when a fact from the input in the same sentence or the next one supports it. Without that fact, remove the word. The groups:

- Quality judgements: "robust", "seamless", "powerful", "flexible", "elegant".
- Vague intensifiers: "very", "extremely", "highly", "significantly".
- Marketing: "best-in-class", "cutting-edge", "next-generation", "innovative". These claim a ranking, so only a named comparative source proves them.
- Minimisers: "simply", "easily", "just", "obviously". Do not use them in a procedure, because they hide a step.

The same applies to a word outside `banned-terms.txt` that fits one of these groups.

Example: "The failover is fast. It completes in 50 ms." The second sentence proves the first.

### 5. Neutral restructuring

When the source uses a recommendation or advisory language, remove that language. Write a statement of operation or a configuration requirement.
Use "must" only for a requirement the source states, or in a safety instruction. Do not turn removed advice into a new obligation.

### 6. Language baseline

Apply this baseline where rules 1–5 do not state otherwise.

- One word, one meaning. Do not use a second word for the same concept.
- A descriptive sentence has a maximum of 25 words. A procedural sentence, including a safety instruction, has a maximum of 20 words. A note uses the 25-word limit.
- Do not use a verb ending in "-ing" as a noun. Write "Installation of the firewall," not "Installing the firewall."
- Use the standard term for a concept. When no standard term exists, define the new term at its first use.
- Expand each acronym at its first use.
- Do not stack more than three nouns, unless the stack is an established term such as "BGP route reflector". Break a longer stack with a verb or a preposition. Keep every part of the meaning.
- Do not use idioms, decorative metaphors, or filler, for example "under the hood", "single pane of glass", "basically", or "in order to". Established industry terms are allowed. An explanatory analogy is allowed under rule 7.
- When two sentences need a dash or a semicolon to join them, write two sentences.
- Do not put a parenthetical remark inside another parenthetical remark.
- Write a sentence only when it gives a fact, a step, a setting, or a constraint of this system. Remove generic method and repeated statements.

### 7. Narrative flow

Apply this rule to a design document, a tech note, a scientific paper, or a system architecture description. Do not apply it to a procedure, a method of procedure, or an instruction.

The model is the classic technical article: the reader follows a line of reasoning, not a list of statements.

- Open each section with the problem or the question that the section answers.
- Put the topic sentence first in each paragraph. The rest of the paragraph supports it.
- Order the content as context, problem, mechanism, consequence, and limits.
- Connect sentences and paragraphs with logical connectors: "as a result", "however", "for this reason", "in contrast", "therefore".
- Follow an abstract statement with a concrete example: a worked case, a packet path, or a number from the input.
- Use a subject and an active verb when the actor is known. Write "The router withdraws the route," not "Withdrawal of the route occurs."
- Vary the sentence length within the limits of rule 6. Put the key point in a short sentence.
- An analogy is allowed when it explains a mechanism. State it once, then give the exact technical statement. Do not extend it.
- Close a section with its consequence or its limit, not with a summary of the same section.

Example:

> Each aggregation router holds one transport route per cell site. As a result, its label table grows with the cell-site count, not with its own configuration. For example, 100 cell sites behind one router add 100 routes and 100 incoming labels. However, a filter toward the cell sites does not reduce this load, because the router still needs every route itself.

## Pre-output checks

- [ ] No banned advisory phrase remains.
- [ ] No unsupported judging word, filler, or idiom remains. Each match of `banned-terms.txt` is reviewed.
- [ ] Each judging or comparative word is backed by a number or a proven fact in the same or the next sentence.
- [ ] Design text opens each section with its problem, connects paragraphs with logical connectors, and gives a concrete example after an abstract statement.
- [ ] Each cited practice names a source the input identifies, with the section or the release.
- [ ] Procedures use the imperative. Design text does not hide the acting component.
- [ ] Descriptive sentences have at most 25 words. Procedural sentences have at most 20 words.
- [ ] One concept keeps one noun through the document.
- [ ] Each acronym is expanded at its first use. Each new term is defined at its first use.
- [ ] No noun stack remains. Each sentence gives a fact, a step, a setting, or a constraint.
- [ ] Technical terms, variables, and code are formatted as code, unless a project convention forbids it.
- [ ] The document contains no internal file path or tool name unless the user asked for it.
- [ ] For a rewrite: the backup exists, `git diff --no-index` shows only prose changes, and code blocks are unchanged.
- [ ] For a passage the user called unclear: the rewrite is shorter than the original.
