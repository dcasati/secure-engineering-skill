# secure-engineering-skill

An agent skill that applies a disciplined, secure engineering mindset to design,
implementation, and review: simplicity, correctness, secure defaults, least
privilege, enforced trust boundaries, and auditable behavior.

## Attribution

This skill is heavily inspired by the OpenBSD Project's engineering culture.
**It is unofficial and not affiliated with or endorsed by the OpenBSD Project.**
"OpenBSD" is used only to credit the inspiration.

## Contents

- [`secure-engineering/SKILL.md`](secure-engineering/SKILL.md): the skill.

## Usage

Copy the `secure-engineering/` folder into your agent's skills directory (for
example `.claude/skills/` or `.github/skills/`). The agent loads it when a
request involves secure-by-default design, trust boundaries, least privilege,
privilege separation, attack-surface reduction, or Bash scripting.

## License

[ISC](LICENSE).
