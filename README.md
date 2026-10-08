# technical-writer — factual technical documentation

An agent skill that writes objective technical documents. Procedures use the imperative. Design documents, tech notes, and architecture descriptions use a formal objective voice. The skill does not add advice.

**One directory, one skill.** `SKILL.md` is what an agent loads. This README is for people. Publishing this directory does not require a second copy of the instructions.

The writing rules are in [`SKILL.md`](SKILL.md). See [`LICENSE.txt`](LICENSE.txt) for the package notice.

## Install from GitHub

Replace `<owner>/<repo>` with the repository that contains this skill. When this directory is the repository root, the command below is enough. Node.js is required for `npx`.

```bash
npx skills add <owner>/<repo> --skill technical-writer
```

List the skill before installing:

```bash
npx skills add <owner>/<repo> --list
```

Use the skill once, without installing it:

```bash
npx skills use <owner>/<repo> --skill technical-writer
```

### Cursor

Install into the current project. The skills CLI writes Cursor project skills to `.agents/skills/technical-writer/`.

```bash
npx skills add <owner>/<repo> --skill technical-writer -a cursor -y
```

Install for every project on this machine. The CLI writes that copy to `~/.cursor/skills/technical-writer/`.

```bash
npx skills add <owner>/<repo> --skill technical-writer -g -a cursor -y
```

A manual copy works too. Keep the directory name `technical-writer`. Cursor reads both of these locations:

- Project: `.cursor/skills/technical-writer/` or `.agents/skills/technical-writer/`
- User: `~/.cursor/skills/technical-writer/`

### Other agents

Pass `-a` with the agent name from the [skills CLI agent list](https://github.com/vercel-labs/skills), or omit `-a` and pick an agent in the prompt.

```bash
npx skills add <owner>/<repo> --skill technical-writer -a claude-code -y
```

A private repository uses the same command. Git, the GitHub CLI, or SSH must already be allowed to clone it.

## Use

Ask for the document in normal language. The agent loads this skill when the request is to write, rewrite, or review a technical document, design document, tech note, architecture description, procedure, method of procedure (MoP), runbook, or instruction.

Name the skill when you want it on a task the description might not match:

```text
Use the technical-writer skill. Write the installation procedure from these notes.
```

The delivered document is Markdown unless you ask for another format. The skill states what the source supports. It asks when a fact is missing. It does not add recommendations.

For a rewrite of an existing file, the agent copies the original to `/tmp` first, edits the file in place, and checks the result with `git diff --no-index`. The reply gives the diff command, so you can review every change.

## Publish

Publish this directory as the repository root, or as `skills/technical-writer/` in a repository that holds several skills. `SKILL.md` must stay in place. The `name` field must stay `technical-writer`.

`README.md` can stay. It is not a `SKILL.md`, so it does not replace the skill and agents do not need it to write.

## Layout

```text
SKILL.md     # agent instructions; higher rules win on a conflict
README.md    # install and use, for people
LICENSE.txt  # package notice
```
