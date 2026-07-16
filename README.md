# war-gaming-plans

A [Claude Code](https://code.claude.com) skill that attacks your own implementation plan **before** the codebase does.

Two passes — assumption verification and scenario sweep — then folds every fix back into the plan document. A war game that ends in a review memo is a failed war game: the executor reads the plan, not the memo.

Use it when a plan or spec is written but not yet executed, or any time you'd say "poke holes in this," "stress-test this," or "what could go wrong."

## Install (recommended — plugin marketplace)

In Claude Code:

```
/plugin marketplace add jojopas/war-gaming-plans
/plugin install war-gaming-plans@jojopas-plugins
```

That's it. Claude Code manages updates — run `/plugin` any time to check.

## Install (manual — no plugin)

Copy the skill folder into your personal skills directory:

```bash
git clone https://github.com/jojopas/war-gaming-plans
cp -R war-gaming-plans/plugins/war-gaming-plans/skills/war-gaming-plans ~/.claude/skills/
```

Or, to scope it to a single project so everyone who clones that repo gets it:

```bash
mkdir -p <your-repo>/.claude/skills
cp -R war-gaming-plans/plugins/war-gaming-plans/skills/war-gaming-plans <your-repo>/.claude/skills/
```

## How it triggers

Once installed, Claude invokes it automatically when you ask to "war game," "poke holes in," or "stress test" a plan — and proactively after writing any plan whose execution touches production, migrations, external services, or money.

## License

MIT
