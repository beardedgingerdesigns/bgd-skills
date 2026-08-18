# sropus

Opt-in Opus 5 launcher that collars its verbosity with an appended system prompt.

**Not a skill.** Tracked here for versioning/portability only — no symlink into `~/.claude/skills/`. The live file is `~/.claude/sr-opus-5.md`; this copy is the backup of record.

## What it is

`sr-opus-5.md` is a communication contract appended to Opus 5's system prompt. It enforces terse, high-signal, reference-pointed responses with no sycophancy. Five sections: positive/negative patterns, reference points (`D1`/`R1`/`F1`…), hard operational boundaries, aliases (`scr`/`eli`/`foc`/`ref`), do/don't examples.

## How it's wired

Two files, opt-in per launch. Plain `claude` is untouched.

```zsh
# ~/.zshrc
sropus() { claude --model claude-opus-5 --append-system-prompt "$(cat ~/.claude/sr-opus-5.md)" "$@"; }
```

`--append-system-prompt` takes a string, so the file is expanded inline via `$(cat ...)` — the CC build has no `--append-system-prompt-file` variant. Edit `~/.claude/sr-opus-5.md` to tune; takes effect next launch.

## Carve-out

The prompt deliberately omits any "never add a co-author to a commit message" rule — that contradicts the global `Co-Authored-By` rule. Global CLAUDE.md governs attribution.

## Scope

Opt-in per launch only. Not applied to plain `claude`, spawned agents, or as an always-on default (deferred).

## Keeping the backup current

```bash
cp ~/.claude/sr-opus-5.md ~/repos/bgd-skills/sropus/sr-opus-5.md
```

Full rationale: claude-os `decisions/log.md`, entry 2026-08-17.
