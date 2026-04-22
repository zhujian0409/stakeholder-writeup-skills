# stakeholder-writeup

**Languages:** English | [中文](README.zh-CN.md)

A [Claude Code](https://docs.claude.com/en/docs/agents-and-tools/claude-code/overview) skill by [@zhujian0409](https://github.com/zhujian0409) that turns a multi-turn technical investigation into a stakeholder-facing Markdown report — verified facts only, audience-tuned, explicit invocation.

## What it does

After spending 20–50 messages with Claude investigating a production issue or a feature, asking "write this up for my boss" usually gets you something that mixes verified findings with guesses, dumps SQL where an exec wanted a conclusion, and papers over impact with vague "mitigated" language.

`stakeholder-writeup` forces a different shape:

- **Step 0 — audience menu.** You pick 1–4 (Tech lead / Non-technical exec / PM · Business / External customer). The skeleton, jargon density, and closer template all change based on your pick.
- **Fact verification.** Every numeric claim in the draft must trace to a command you actually ran or a statement you literally made. No "~25% improvement" when nobody measured it.
- **Honest impact.** Unknowns stay unknown ("impact scope: not measured — need N+1 hours of log analysis to confirm") instead of being padded into false confidence.
- **12-item pre-send checklist.** Runs before the final draft lands in your hands. Fails loudly if anything is unsourced.

Output language auto-detects: English primary, Chinese when the conversation is primarily Chinese.

## When to use

Only when **explicitly invoked** — the skill hard-ignores casual phrases like "write a report" / "写个 md". Trigger it via one of:

- Slash command: `/stakeholder-writeup`
- Explicit name: "use the stakeholder-writeup skill on the debug session we just finished"
- Chinese: "用 stakeholder-writeup 写一份"

`disable-model-invocation: true` is set in the skill frontmatter, so Claude will not auto-trigger this skill on its own initiative.

## Installation

Clone this repo and copy the skill directory into your Claude Code user-level skills folder:

```bash
mkdir -p ~/.claude/skills
git clone https://github.com/zhujian0409/stakeholder-writeup-skills.git
cp -r stakeholder-writeup-skills/stakeholder-writeup ~/.claude/skills/
```

Or keep the repo and symlink (so `git pull` updates the skill in place):

```bash
mkdir -p ~/.claude/skills
git clone https://github.com/zhujian0409/stakeholder-writeup-skills.git
ln -s "$(pwd)/stakeholder-writeup-skills/stakeholder-writeup" ~/.claude/skills/stakeholder-writeup
```

Claude Code auto-registers anything placed under `~/.claude/skills/`. Your next session will see `/stakeholder-writeup` as a slash command.

## Design notes

Two ideas run through the skill:

1. **Evidence, not vibes.** Every numeric claim must trace to a real command output or an explicit user statement. No estimates masquerading as facts. The pre-send checklist fails if anything is unsourced.
2. **Audience-aware output.** A CEO doesn't want your SQL appendix. A tech lead doesn't want analogies. The skill figures out who's reading and tailors length, jargon density, and closer format accordingly.

See [SKILL.md](stakeholder-writeup/SKILL.md) for the full 7-step workflow, skeleton map, audience variant table, and the real-world pitfalls the skill refuses to fall into.

## License

MIT — see [LICENSE](LICENSE).
