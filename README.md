# jojopas-plugins

A [Claude Code](https://code.claude.com) plugin marketplace — reusable skills for planning, agent delegation, and session continuity.

## Install

In Claude Code, add the marketplace once:

```
/plugin marketplace add jojopas/claude-plugins
```

Then install any of the plugins:

```
/plugin install war-gaming-plans@jojopas-plugins
/plugin install agent-persona@jojopas-plugins
/plugin install session-baton@jojopas-plugins
```

Claude Code manages updates — run `/plugin` any time to check.

## Plugins

### war-gaming-plans
Attacks your own implementation plan **before** the codebase does. Two passes — assumption verification and scenario sweep — then folds every fix back into the plan document. Triggers when you ask to "war game," "poke holes in," or "stress test" a plan, and proactively after any plan whose execution touches production, migrations, external services, or money. Lives in this repo under [`plugins/war-gaming-plans`](plugins/war-gaming-plans).

### agent-persona
Brings a persistent, file-backed agent teammate into a session in **Become**, **Consult**, or **Delegate** mode — loading their identity, memory, rules, and skills from the agent's workspace, and writing a session digest back on exit. Works with OpenClaw out of the box, or any agent-workspace layout via `roster.json`. Source: [jojopas/agent-persona](https://github.com/jojopas/agent-persona).

### session-baton
Banks a session into a one-shot, self-deleting resume marker (a **baton**) so the next session resumes with zero ramp-up. Ships the `/renew` skill plus a `SessionStart` hook that auto-injects the marker on `/clear` — installing the plugin wires the hook for you. Source: [jojopas/session-baton](https://github.com/jojopas/session-baton).

## Manual install (no plugin system)

Each skill also works as a plain personal skill — copy its `SKILL.md` folder into `~/.claude/skills/`. See each source repo's README for the exact path.

## License

MIT
