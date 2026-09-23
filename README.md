# butler-app-checkout

Run an errand inside a phone app — GrabFood, Grab, foodpanda, anything that only
exists as an app or whose website will not let a butler through — on a cloud
Android phone that bevo-server rents by the minute, with the owner's approval
before anything is paid.

One Butler skill, published by the
[Butler skill hub](https://github.com/Virtual-Protocol/butler-skills): the hub's
`skills.json` lists this skill by name, repo and ref, and every hub build
re-resolves that ref to a commit and publishes the files. The butler
(`virtuals-agent`) does not ship it: it installs the skill on demand with
`hub_install` when an errand calls for it, then loads it with the `skill` tool. A
merge here reaches butlers on the next hub build.

- `SKILL.md` — the playbook: frontmatter plus the standard sections (see the hub's
  [SKILL_STANDARD.md](https://github.com/Virtual-Protocol/butler-skills/blob/main/SKILL_STANDARD.md))
- `CHANGELOG.md` — one entry per version; every change bumps `version` in SKILL.md

No `duty.py`: a phone-app errand runs once, in a conversation. A standing order
("coffee every morning") is a duty, set up after walking the errand once.

## What stays in the container

The `app-checkout` command is a **container primitive**, not part of this skill:
`virtuals-agent` ships it as `src/integrations/butler/bin/app-checkout.mjs`, a
forwarder to bevo-server's phone rental (`/butler-exec/device-session*` and
`/butler-exec/app-action*`). bevo-server rents the phone — the provider key and the
device id never reach the container, which holds only a numeric session id — and
files every in-app order as an owner approval. `bevo-sms` (the butler's own number
and its sign-in codes) and `bevo-notify` are container commands too. This
repository holds only the how-to: an app-specific flow is prose the butler reads,
never a step grammar the container interprets.

## Validate before merging

The hub publishes its validator standalone — no Butler account, container or
registry checkout needed:

```bash
curl -sSLO https://virtual-protocol.github.io/butler-skills/tools/validate.py && python3 validate.py --standalone . --maintainer
```

`--maintainer` is required: `butler-` is the maintainer-reserved namespace. Keep the
downloaded validator out of the commit — `.gitignore` already lists it. CI runs the
same check through the hub's composite action (`.github/workflows/validate.yml`):

```yaml
- uses: Virtual-Protocol/butler-skills/.github/actions/validate@main
  with:
    maintainer: "true"
```
