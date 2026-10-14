# Superpowers ML — working in this repo

This is a standalone extension plugin for [Superpowers](https://github.com/obra/superpowers), holding two ML skills. It is not a fork and has no upstream remote: it began as one, but the history was rebuilt so that only the skill commits and the four retained upstream files remain (see the root commit and `README.md`). **Nothing here goes upstream.** Do not open pull requests against obra/superpowers from this repo, and do not apply upstream's contributor rules to changes made here.

## What lives here

- `skills/brainstorming-ml/` and `skills/running-eda-ml/` — the only content.
- `hooks/` — a SessionStart hook that fires on compaction only, plus the polyglot cmd/bash wrapper inherited from upstream (see `docs/windows/polyglot-hooks.md`).
- `.claude-plugin/` — plugin and marketplace manifests. Plugin name is `superpowers-ml`; it must not collide with core `superpowers`. The version is in both `plugin.json` and `marketplace.json`, so bump them together.

The chain is `brainstorming-ml` → `running-eda-ml` → `superpowers:writing-plans`. Both skills override core brainstorming's terminal rule ("writing-plans comes next"), so a change to how one skill hands off usually means a matching change in the other.

## Rules

**Skill references are namespaced.** `superpowers:brainstorming` and `superpowers:writing-plans` resolve to installed core and must stay spelled that way. The two skills here are `superpowers-ml:brainstorming-ml` and `superpowers-ml:running-eda-ml`. If the plugin is ever renamed, those self-references have to be renamed with it.

**Don't pin core.** The point of this plugin is that it works against whatever Superpowers version is installed. Don't copy core skill content in, don't reimplement a core skill, and don't depend on a core detail beyond the skill names above. The one deliberate exception: both skills quote core brainstorming verbatim so they can override it (its terminal rule, "the ONLY skill you invoke after brainstorming is writing-plans", and its review-gate wording, "before we start writing out the implementation plan"). Don't add more quotes like these. After a core upgrade, grep the installed core for these strings, because an override that points at reworded text no longer overrides anything.

**Skills are behaviour-shaping content, not prose.** Changing a gate is changing code. Use `superpowers:writing-skills` and test the change: a RED baseline where an agent without the skill violates the rule, then GREEN where an agent with it complies. A scenario that only passes with the skill present proves nothing on its own.

**Commits here carry no `Co-Authored-By` or `Claude-Session` trailers.** Commit and push only when asked, as separate explicit steps.

**Changes to `hooks/session-start` must be tested on both branches** — `source: compact` emits valid JSON, anything else emits nothing — and the file is pinned to LF in `.gitattributes`. Don't let a CRLF checkout into it. To check:

```sh
echo '{"source":"compact"}' | /usr/bin/bash hooks/session-start | python -m json.tool
echo '{"source":"startup"}' | /usr/bin/bash hooks/session-start | wc -c   # expect 0
git ls-files --eol hooks/session-start                                   # expect i/lf w/lf
```

On Windows, call Git Bash by path (`/usr/bin/bash` inside the Bash tool). A bare `bash` can resolve to the WSL launcher in `WindowsApps`, which fails with `execvpe(/bin/bash) failed` and makes the hook look broken when it isn't.

`hooks/run-hook.cmd` is kept verbatim from upstream. Don't edit it to fix something here; work around it in `hooks.json` or `session-start`.
