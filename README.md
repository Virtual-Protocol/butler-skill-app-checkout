# butler-app-checkout

Run an errand inside a phone app — GrabFood, Grab, foodpanda, anything that only
exists as an app or whose website blocks a browser — on a cloud Android phone that
bevo-server rents by the minute. Sign in with the butler's own number, order, and
clear every "Place order" through the owner's purchase policy first.

This repository is one Butler skill. It is published through the
[Butler Skill Hub](https://github.com/Virtual-Protocol/butler-skills): `skills.json`
there lists this skill by name and repo link, and every hub build re-resolves the
listed ref to a commit, so a merge here reaches Butlers on the next build. Butler
containers clone the resolved commit.

- `SKILL.md` — the playbook (frontmatter + the fixed sections; see the hub's
  [SKILL_STANDARD.md](https://github.com/Virtual-Protocol/butler-skills/blob/main/SKILL_STANDARD.md))
- `CHANGELOG.md` — one line per version; every change bumps `version` in SKILL.md

There is no `duty.py`: this skill is `modes: ["one-off"]`. A standing order
("coffee every morning") is a duty built with `bevo-automation` after rehearsing
the errand once — the skill says so under `## Limits` rather than shipping a
schedule of its own.

## What stays in the container

The `app-checkout` command itself is a **container primitive**, not part of this
skill: `bevo-docker` installs the wrapper and owns `api/scripts/app-checkout.py`,
and `bevo-server` rents the phone (the Mobilerun key and the device id never reach
the container — the shim holds only a numeric session id). This repository holds
only the how-to.

The same split as `butler-web-checkout`, and for the same reason: an app-specific
flow is prose a butler reads, never a step grammar the container has to interpret.
The two GrabFood recipes that shipped in the first draft of bevo-docker#138 were
removed before merge for exactly that, and their content is the numbered steps in
`SKILL.md`.

## Validate before merging

No Butler account, container or registry checkout needed — the hub publishes its
validator as a standalone file:

```bash
curl -sSLO https://virtual-protocol.github.io/butler-skills/tools/validate.py
python3 validate.py --standalone . --maintainer
```

`--maintainer` is required: `butler-` is the maintainer-reserved namespace. Keep
the downloaded validator out of the commit — `.gitignore` already lists it.

`replay.py` is for skills with a `duty.py` and prints "nothing to replay" here, so
CI is a single step:

```yaml
- uses: Virtual-Protocol/butler-skills/.github/actions/validate@main
  with:
    maintainer: "true"
```

The validator only accepts `app-checkout` as a command from
[butler-skills#25](https://github.com/Virtual-Protocol/butler-skills/pull/25)
onward — that PR has to be on the hub's `main` before this repo's CI can pass.
