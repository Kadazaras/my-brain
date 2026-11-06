# my-brain

A shared "second brain" for Claude Code. Install it once, and every Claude Code session gets:

- **Team context**: shared instructions (like an org-wide `CLAUDE.md`), added at the start of every session.
- **Skills**: reusable playbooks such as `/weekly-review`.

When someone pushes a change to this repo, everyone who installed it gets the change.

## What's in this folder

```
my-brain/
├── .claude-plugin/
│   └── marketplace.json          # The "store": lists the plugins in this repo and where to fetch them
└── plugins/my-brain/             # The plugin itself
    ├── .claude-plugin/plugin.json  # Plugin name and description. No "version" on purpose (see below)
    ├── team-context.md           # Shared instructions. Edit this to change what Claude knows
    ├── hooks/hooks.json          # Adds team-context.md to every session at start
    └── skills/
        └── weekly-review/SKILL.md  # A skill. Add more folders here for more skills
```

**Why not a `CLAUDE.md`?** Claude Code ignores a `CLAUDE.md` inside a plugin. Instead, a `SessionStart` hook prints `team-context.md`, and Claude adds that output to the session. The hook runs when a session starts, after `/clear`, and after the conversation is compacted.

## Install (once per person)

You need read access to the GitHub repo, with your SSH key set up.

```bash
claude plugin marketplace add git@github.com:Kadazaras/my-brain.git
claude plugin install my-brain@my-brain
```

Start a new `claude` session to check it:
- Ask *"what is the brain marker?"*. It should answer with the marker set in `team-context.md`.
- Run `/plugin`. `my-brain` should be listed as enabled.
- The skills show up as commands, e.g. `/weekly-review` (or `/my-brain:weekly-review`).

## Changing the brain

1. Edit `plugins/my-brain/team-context.md`, or add or edit a skill in `plugins/my-brain/skills/<name>/SKILL.md`.
2. Commit and push to `main`.

Every commit counts as a new version, because `plugin.json` has no `"version"` field.
If you ever add `"version": "1.0.0"`, pushes are **ignored** until you bump that number.

## How updates reach people

An update has **three steps**. All three must happen before the change shows up:

| Step | What it does | Command |
|------|--------------|---------|
| 1. Refresh the marketplace | Downloads the latest catalog from GitHub | `claude plugin marketplace update my-brain` |
| 2. Update the plugin | Installs the new version of the plugin | `claude plugin update my-brain@my-brain` |
| 3. Restart Claude Code | Loads the new version | Exit and run `claude` again |

Step 1 alone does **not** update the plugin. Step 2 is needed too.

A running session keeps the old version and the old team context. `/reload-plugins` reloads the skills, but the team context only arrives at session start. If you reload instead of restarting, also run `/clear`.

### Option A: update by hand (default)

Run the three steps above whenever you want the latest version.

### Option B: auto-update (recommended)

With auto-update on, Claude Code runs steps 1 and 2 for you in the background. **You only restart.**

**Turn it on**, in either of these ways:

- **From inside Claude Code:** run `/plugin`, open **Marketplaces**, select `my-brain`, and choose **Enable auto-update**.
- **In settings:** add this to `~/.claude/settings.json`, keeping anything already there:

  ```json
  {
    "extraKnownMarketplaces": {
      "my-brain": {
        "source": { "source": "git", "url": "git@github.com:Kadazaras/my-brain.git" },
        "autoUpdate": true
      }
    }
  }
  ```

Auto-update is **off by default** for custom marketplaces like this one, so you have to turn it on.

**What happens after that:**

1. Someone pushes a change to `main`.
2. Next time you use Claude Code, within about 10 minutes of your first message, it checks GitHub and downloads the new version in the background. The delay is random, so the whole team doesn't fetch at the same moment.
3. **Your current session does not change.** Start a new session, or run `/reload-plugins` then `/clear`, to get the update.

**What you need to do:** nothing besides restarting Claude Code now and then. Make sure git can reach GitHub **without asking for anything**: background updates can't prompt for a password or SSH passphrase. Load your key into `ssh-agent`, or use a key without a passphrase. You can check with:

```bash
ssh -T git@github.com    # should greet you without prompting
```

**Check which version you have:**

```bash
claude plugin list       # "Version" shows the commit it is on
git ls-remote git@github.com:Kadazaras/my-brain.git main   # latest commit on GitHub
```

The first part of the version should match the latest commit hash.

## Rolling out to a whole organization

On a Claude Team or Enterprise plan, an org owner can paste the snippet below into **Organization settings → Claude Code → Managed settings**. IT can also deploy it as `managed-settings.json` on each machine.
Every member then gets the marketplace and the plugin with auto-update on, and can't turn it off:

```json
{
  "extraKnownMarketplaces": {
    "my-brain": {
      "source": { "source": "git", "url": "git@github.com:Kadazaras/my-brain.git" },
      "autoUpdate": true
    }
  },
  "enabledPlugins": { "my-brain@my-brain": true }
}
```

## Uninstall

```bash
claude plugin uninstall my-brain@my-brain
claude plugin marketplace remove my-brain
```
