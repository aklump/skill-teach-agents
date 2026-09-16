# Claude Skill: Teach Agents

This skill helps preserve knowledge for future agents and sessions, reducing ramp-up time for agents working on your application.

## Install as a Private Skill

```bash
cd ~
mkdir -p ~/.claude/skills/
cd ~/.claude/skills/
git clone git@github.com:aklump/skill-teach-agents.git
rm -rf teach-agents/.git teach-agents/.claude-plugin

# Start a new Claude session.
claude

# Test that the skill appears by typing "/teach-ag"; it should appear in the list.
```

## Usage

Before closing a session, type `/teach-agents`, and this skill will capture useful context for future sessions.
