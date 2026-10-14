# Superpowers ML

Machine-learning extension skills for [Superpowers](https://github.com/obra/superpowers).

Two skills, both Claude Code:

- **`brainstorming-ml`** — ML-specific content for the brainstorming process: prediction target, problem type, training-data scope, EDA plan, feature-engineering plan, pipeline design, metric and candidate models. Produces a design spec whose EDA, feature and pipeline sections are explicit proposals.
- **`running-eda-ml`** — turns that EDA plan into a marimo notebook and pairs with you on a live kernel. You run the cells; the agent reads what they produced and works the findings one at a time until the final feature list and preprocessing pipeline are locked in. Every settled decision is appended to a decisions file as it lands, so the spec rewrite at the end is drafted from that file rather than from a conversation that may have been compacted by then.

The ML chain is `brainstorming-ml` → `running-eda-ml` → `superpowers:writing-plans`, replacing core brainstorming's rule that writing-plans comes next.

## Requirements

This is an extension, not a standalone library. It assumes:

- **Superpowers core** — the skills defer to `superpowers:brainstorming` for the brainstorming process and hand off to `superpowers:writing-plans` at the end. Nothing here pins a version, replaces a core skill, or copies core content in. Both skills do quote core brainstorming in order to override it (its terminal rule and its review-gate wording); that text was checked present in 5.1.0 and 6.4.1. A future core rewording would make an override point at text that no longer exists, so it is worth re-grepping after a core upgrade.
- **[marimo-pair](https://github.com/marimo-team/marimo-pair)** — `running-eda-ml` requires it for the live kernel. There is no fallback path.
- **Claude Code ≥ 2.1.152**, for the `reloadSkills` field the compaction hook emits.

## Install

```sh
claude plugin marketplace add marimo-team/marimo-pair
claude plugin install marimo-pair@marimo-pair

claude plugin marketplace add eduardorestrepo89/superpowers-ml
claude plugin install superpowers-ml@superpowers-ml
```

Core Superpowers installs separately and is unaffected; the two plugins sit side by side.

## The compaction hook

`hooks/session-start` fires on `SessionStart` with `source: compact` and nothing else. Compaction replays an already-invoked skill as a truncated attachment and does not re-send the available-skills listing, so a gate that entered as an instruction comes back as background information about something the agent once did. The hook re-asserts from outside the transcript that a part-way-through skill must be re-read from disk, and asks Claude Code to re-scan the skill directories.

It is a mitigation, not a fix. The gates still decay inside the context window; this only keeps re-pointing at the file. Whether it holds late in a long session is untested.

## License

MIT, with both copyright lines in `LICENSE`.

Four files come from [obra/superpowers](https://github.com/obra/superpowers) and are retained under its copyright: `hooks/run-hook.cmd` (verbatim), `docs/windows/polyglot-hooks.md`, `.gitattributes` and `.gitignore`. `hooks/session-start` was written here, but its shape and its `printf`-instead-of-heredoc workaround come from upstream. Everything else — both skills, both manifests, this README — is new work. No Superpowers skill content is copied in.
