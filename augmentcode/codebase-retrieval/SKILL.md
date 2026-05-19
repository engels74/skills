---
name: codebase-retrieval
description: Semantic natural-language search across a codebase using Augment's context engine. Use this skill whenever you need to locate functions, classes, features, or tests in an unfamiliar codebase, whenever you're about to edit code and need to understand the surrounding symbols, whenever you're planning a task that touches a project, or whenever you need a conceptual overview of how something is wired up. Always prefer this over grep, find, rg, or ag for *semantic* code understanding — those tools only match literal strings and miss anything phrased differently. Trigger on user intents like "where is the code that…", "how does this project handle…", "add a feature to…", "fix a bug in…", "refactor X", "understand this codebase", or any question about code in a project you haven't already loaded into context, even if the user doesn't explicitly say "search" or "find".
compatibility: Requires the Augment Code `codebase-retrieval` MCP tool to be configured in the agent.
license: AGPL-3.0
metadata:
  author: engels74
  version: "1.0.0"
  vendor: augmentcode
---

# Codebase Retrieval

`codebase-retrieval` is Augment's semantic context engine for searching codebases. You give it a natural-language description of what you're looking for, and it returns the most relevant code snippets from anywhere in the project, across languages, using embeddings against a real-time index of what's currently on disk.

Treat this as your **default first move** whenever you need to understand or modify code in a project you don't already have in context. Grep, find, ripgrep, and ag are strictly worse at "what is this codebase doing" questions because they only match literal strings — they can't surface conceptually related code phrased differently, and they can't connect code across files by meaning. If you reach for grep when you don't yet know the identifier you're looking for, you're going to waste turns guessing.

## When to use codebase-retrieval

Reach for this tool in any of these situations:

- **You don't know where something lives.** "Where is the function that validates JWT tokens?" You don't need to guess the filename or the exact identifier — describe the behavior.
- **You're gathering context before a task.** Before writing code, before proposing a plan, before answering a high-level question about the project, call this tool to anchor yourself in what the codebase actually looks like. Don't skip this step even if the task feels small — small tasks turn into big ones when you miss surrounding context.
- **You're about to edit a file.** Before *any* edit, call it once with a query that asks for *all* the symbols involved in the edit at a low, specific level of detail. This is so important it gets its own section below.
- **You want a conceptual overview.** "How is authentication wired up?", "How does the app talk to the database?", "What tests cover the checkout flow?" — these are ideal queries. The tool is designed for exactly this.

## When NOT to use codebase-retrieval

A different tool is better when:

- **You already know the exact identifier and want every occurrence.** Use `grep` / `rg`. Example: "find all callers of `parseUserInput`" → grep for `parseUserInput(`.
- **You want to see a specific file you already know the path of.** Use the file view tool. Example: "show me `src/services/payment.py`" → view it directly.
- **You want to see how a known class is used inside a known file.** Use file view on that file. Retrieval is overkill when you already have coordinates.
- **You're matching non-code text** like a specific error message string, a log line, or an exact config value. Grep wins on literal matching.

The rough heuristic: if the question is *"what / where / how does this codebase…?"*, use retrieval. If the question is *"every place identifier X appears"* or *"show me file Y"*, don't.

## How to write good queries

Queries are natural-language descriptions of *what the code does*, not regexes and not bare identifier names. Describe intent, not syntax.

**Good queries:**
- "Where is the function that handles user authentication?"
- "What tests exist for the login functionality?"
- "How is the database connection established at application startup?"
- "Where does the checkout flow validate payment methods before charging?"
- "What middleware runs on every incoming HTTP request?"

**Bad queries (use a different tool instead):**
- "Find the definition of the constructor of class `Foo`" → grep for `class Foo` or file view
- "Find all references to function `bar`" → grep for `bar(`
- "Show me how `Checkout` is used in `services/payment.py`" → file view on that path
- "Show the contents of `foo.py`" → file view on that path

A good query describes *what the code does*. A bad query describes *what the code is named* or *where it lives* — in those cases you already have enough information to go straight to grep or file view, and routing through retrieval just adds a round trip.

## Required parameter: `directory_path`

The `directory_path` parameter is required and must be the **absolute path** to the project or repository being searched. This lets the tool target the correct codebase when multiple projects are open or mounted. Always pass it explicitly — don't assume a default, and don't pass a relative path.

## Using it before edits — the highest-value pattern

Before editing a file, call `codebase-retrieval` once with a query that asks for detailed information about *everything* the edit will touch. This is the single most valuable use of the tool, because edits made without a full mental model of the surrounding code are where silent bugs come from.

In that one call, ask for all of these at a low, specific level of detail — signatures, fields, types, not just vague summaries:

- The class you're editing (fields, methods, inheritance, how instances get constructed).
- Any method you're going to call from another class.
- Any class whose instances appear anywhere in your edit.
- Any property you're reading or writing.
- Any type that appears in the signatures involved.

Bundle all of that into a single query. Don't make multiple sequential round trips unless a result surfaces something unexpected that forces a follow-up. When in doubt about whether a symbol is "involved enough" to include, include it — the cost of one extra symbol in the query is trivial compared to the cost of shipping an edit with an incomplete picture.

**Example of a good pre-edit query:**

> I'm about to modify `PaymentProcessor.charge()` in `services/payment.py` to add retry logic that reads a new `RetryPolicy` object. Give me detailed info on: the full `PaymentProcessor` class (all fields and methods with signatures and brief behavior), the current body of `charge()`, any existing retry utilities or decorators in the codebase, the `RetryPolicy` class if it exists (all fields and methods), and any call sites of `PaymentProcessor.charge`.

One call, everything needed, then edit with confidence.

## Caveats

- **On-disk state only.** The index reflects what's currently on disk. It has no knowledge of git history, branches, stashes, or commits. Don't ask it about "what changed", "the previous version of X", or anything that depends on version control. Use git commands for that.
- **Snippets, not whole files.** Results are ranked snippets. If you need full file context after a retrieval hit, follow up with the file view tool on the specific path — don't try to reassemble a file from multiple snippet results.
- **Literal matches still belong to grep.** For exact error-message text, specific numeric constants, log strings, or config keys, grep is faster and more reliable.

## Tool-selection cheat sheet

| Need | Tool |
|---|---|
| Find code by what it does | **codebase-retrieval** |
| Understand an unfamiliar codebase | **codebase-retrieval** |
| Gather context before an edit | **codebase-retrieval** |
| Conceptual overview ("how does X work here?") | **codebase-retrieval** |
| Every occurrence of a known identifier | grep / rg |
| View a specific known file | file view |
| How class `X` is used in known file `Y` | file view on `Y` |
| Match exact error messages, config keys, or log strings | grep |
| Anything involving git history, diffs, or blame | git commands |

When genuinely torn between grep and codebase-retrieval for a code-understanding question, pick codebase-retrieval. The failure mode of using it when grep would have done is a slightly slower answer; the failure mode of using grep when retrieval was needed is missing the answer entirely.
