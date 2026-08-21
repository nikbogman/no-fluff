# no-fluff

A Claude skill that keeps documentation, code comments, and docstrings short
and useful.

It doesn't try to document everything. It tries to say what matters in as few
words as possible.

## What it does

- Removes comments that just repeat what the code already says.
- Shortens long-winded writing, and turns it into tables, lists, or commands where that's clearer.
- When the same thing is explained in three places, keeps one and links to it.
- Points out broken links, commands that no longer work, and settings that were renamed.
- Adds a docstring only when something public has none and its name doesn't explain it.

It prefers deleting text to rewriting it. If there's nothing to cut, it says so
and leaves your files alone.

## What it leaves alone

Some comments look like noise but are actually doing a job. Delete one and the
build breaks. These are skipped:

| Type | Examples |
|---|---|
| Tool directives | `# noqa`, `# type: ignore`, `// eslint-disable-next-line`, `//go:embed`, `# fmt: off`, shebangs |
| Doctests and doc examples | Python doctests, Rust doc examples — these run as tests |
| Generated files | Sphinx/TypeDoc output, OpenAPI stubs, CHANGELOGs a tool maintains |
| Vendored dependencies | `node_modules/`, `vendor/`, `third_party/`, anything gitignored |
| License and policy files | `LICENSE`, `SECURITY.md`, `CODE_OF_CONDUCT.md`, issue and PR templates |

It also checks `git blame` before removing a comment that looks pointless but
is strangely specific. Those usually got added after something broke.

## Install

```bash
git clone https://github.com/<you>/no-fluff.git
cp -r no-fluff ~/.claude/skills/
```

For one project only, put it in `.claude/skills/no-fluff/` instead. There's
also a packaged `no-fluff.skill` on the releases page.

## Usage

```
/nofluff
/nofluff src/auth/
"clean up the comments in payments.py"
"this README is bloated"
```

| What you ask for | What it does |
|---|---|
| Specific files | Edits them |
| Nothing specific | Asks which files you mean |
| The whole repo | Shows you what it plans to change first |

For anything bigger than a file or two, commit your work first. The diff is
your only way back.

## Making it automatic

The skill runs when you ask for it. If you also want Claude writing this way
during normal work, add this to your `AGENT.md` or `CLAUDE.md`:

```markdown
## Documentation

Write documentation, comments, and docstrings fluff-free: maximum useful
information, minimum necessary text. Comments explain *why*, not *what* —
if the code already says it, don't comment it. Don't add docs for coverage's
sake. See the `no-fluff` skill for the full rules and use it for cleanup passes.
```

Claude always reads those files, so this works more reliably than waiting for
the skill to trigger. It's also less work overall — not writing five lines
beats deleting four later. Keep it short; don't paste in the whole rule set.

## Things to watch out for

Roughly most to least likely:

**Too many rules in AGENT.md.** The more rules that file has, the less
carefully Claude follows any one of them. If yours already has fifteen, adding
a sixteenth weakens the rest. Use the short version above.

**It can go too far.** "Don't add docs for coverage's sake" sometimes turns
into skipping docs you actually wanted. If you keep having to ask for
docstrings, soften that line.

**Deleted text is gone.** Start from a clean git state, read the diff, and put
documentation changes in their own commit so undoing them is easy.

**Big runs make big diffs.** Try one folder before the whole project.

**It guesses where things belong.** To avoid repeating itself, it moves
information into whichever file seems to own that topic. If your project has no
clear structure, it might pick the wrong one. Check what moved.

**No way to protect a single comment.** Apart from the list above, there's no
"leave this alone" marker yet, so unusual legal wording could get rewritten.

## What's in the box

```
no-fluff/
├── SKILL.md               the main rules
├── README.md              this file
└── references/
    ├── comments.md        code comments and docstrings
    └── docs.md            Markdown files and READMEs
```

Claude reads `SKILL.md` whenever the skill runs, and only opens the reference
files when they're relevant.

## How this was built

Claude wrote most of this skill. A human set the goals and reviewed each round
of changes, but the wording is mostly the model's. Nothing here is novel — it's
ordinary advice about documentation, written down so an agent will follow it.

It's been tested on one deliberately tricky test file: tool comments, a
doctest, a comment worth keeping, and some real padding. It hasn't been tested
across lots of real projects. Read the diffs on your first few runs.

Bug reports welcome, especially about comments it deleted that it shouldn't have.
