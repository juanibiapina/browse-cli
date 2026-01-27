# Browse Skills

This directory contains skill files for AI coding agents.

## Pi Agent

To use the browse skill with [Pi coding agent](https://github.com/badlogic/pi-mono):

```bash
# Option 1: Symlink (auto-updates)
ln -s "$(pwd)/skills/browse" ~/.pi/agent/skills/browse

# Option 2: Copy
cp -r skills/browse ~/.pi/agent/skills/
```

The skill will be available when pi detects browser automation tasks.

## Other Agents

The `SKILL.md` file is a comprehensive reference that can be adapted for other AI coding agents or used as documentation for LLM prompts.
