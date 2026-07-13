# skills

Harness-agnostic agent skills. One canonical copy per skill lives here; every
agent harness (Claude Code, Codex, Grok, Gemini, …) picks them up by
**symlink**, so there is never a second version to keep in sync.

Each skill is written in capability terms (spawn a subagent, message a live
agent, pick a model tier) rather than one harness's tool names, so the same
`SKILL.md` works wherever it's linked. See each skill's own README for what it
does.

## Skills

| Skill | What it does |
|-------|--------------|
| [`team-lead`](team-lead/) | Orchestrate a team of subagents into a verified, adversarially-checked deliverable. |
| [`discuss`](discuss/) | Sounding-board / discussion mode — direct back-and-forth or a multi-agent panel. |
| [`stress-test-plan`](stress-test-plan/) | Pre-build validation — 4 CIA-Red-Team personas run in sequence against a plan/idea. |

## Install (clone once, symlink per harness)

Clone the repo somewhere stable:

```sh
git clone https://github.com/bbk1ng/skills.git ~/skills
```

Then symlink each skill into whatever harness skill dir you use. Harness skill
dirs are conventionally `~/.<harness>/skills/`:

| Harness | Skills dir |
|---------|-----------|
| Claude Code | `~/.claude/skills/` |
| Codex | `~/.codex/skills/` |
| Grok | `~/.grok/skills/` |
| Gemini | `~/.gemini/skills/` |

Link one skill into one harness:

```sh
ln -s ~/skills/team-lead ~/.claude/skills/team-lead
```

Link **all** skills into **all** harnesses in one shot:

```sh
for harness in claude codex grok gemini; do
  dir="$HOME/.$harness/skills"
  [ -d "$dir" ] || continue
  for skill in ~/skills/*/; do
    name=$(basename "$skill")
    ln -sfn "$skill" "$dir/$name"   # -f replaces a stale copy, -n won't descend into an existing link
  done
done
```

`ln -sfn` is safe to re-run: it replaces a stale link in place. If a harness
already holds a **real directory** for a skill (an old un-linked copy), remove
that dir first — otherwise `ln` creates the link *inside* it:

```sh
rm -rf ~/.codex/skills/team-lead && ln -sfn ~/skills/team-lead ~/.codex/skills/team-lead
```

## Update

```sh
cd ~/skills && git pull
```

Every harness sees the update instantly — they all point at this clone.

## Editing a skill

Edit `~/skills/<name>/SKILL.md` here, commit, push. Do **not** edit the copy a
harness shows you — that's a symlink back to this repo. Frontmatter
`description:` values containing `:` or `"` must be quoted, or YAML parsing
breaks.
