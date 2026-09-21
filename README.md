# butler-app-checkout

Run an errand inside a phone app — GrabFood, Grab, foodpanda, anything that only
exists as an app or whose website blocks a browser — on a cloud Android phone that
bevo-server rents by the minute.

One Butler skill, published through the
[Butler Skill Hub](https://github.com/Virtual-Protocol/butler-skills): `skills.json`
there lists this skill by name and repo link, every hub build re-resolves the listed
ref to a commit, and Butler containers clone that commit — so a merge here reaches
Butlers on the next build.

- `SKILL.md` — the playbook (frontmatter + the fixed sections; see the hub's
  [SKILL_STANDARD.md](https://github.com/Virtual-Protocol/butler-skills/blob/main/SKILL_STANDARD.md))
- `CHANGELOG.md` — one line per version; every change bumps `version` in SKILL.md

No `duty.py`: this skill is `modes: ["one-off"]`. A standing order ("coffee every
morning") is a duty built with `bevo-automation` after walking through the errand once by hand.

## What stays in the container

The `app-checkout` command is a **container primitive**, not part of this skill:
`bevo-docker` owns `api/scripts/app-checkout.py` and installs the wrapper, and
`bevo-server` rents the phone (the Mobilerun key and the device id never reach the
container — the shim holds only a numeric session id). This repository holds only
the how-to: like `butler-web-checkout`, an app-specific flow is prose a butler
reads, never a step grammar the container interprets.

## Validate before merging

The hub publishes its validator standalone — no Butler account, container or
registry checkout needed:

```bash
curl -sSLO https://virtual-protocol.github.io/butler-skills/tools/validate.py
python3 validate.py --standalone . --maintainer
```

`--maintainer` is required: `butler-` is the maintainer-reserved namespace. Keep the
downloaded validator out of the commit — `.gitignore` already lists it.

`replay.py` is for skills with a `duty.py` and prints "nothing to replay" here, so CI
is a single step:

```yaml
- uses: Virtual-Protocol/butler-skills/.github/actions/validate@main
  with:
    maintainer: "true"
```

The validator only accepts `app-checkout` as a command from
[butler-skills#25](https://github.com/Virtual-Protocol/butler-skills/pull/25) onward —
that PR has to be on the hub's `main` before this repo's CI can pass.
