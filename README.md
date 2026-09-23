# butler-app-checkout

Run an errand inside a phone app — ZUS, foodpanda, Shopee, a Grab ride,
anything that only exists as an app or whose website will not let a butler
through — on a cloud Android phone that bevo-server rents by the minute, with
the owner's approval before anything is paid.

This is the **generic** phone-app playbook, and every app skill builds on it:
the phone's clock, starting the phone at the owner's address, making sure the
app opened, signing in on the butler's own number, reading the screen, and
every money rule live here. A skill for one app declares this one in
`metadata.butler.requires.skills` and carries only that app's steps; this
skill's app step tells the butler to follow such a skill when it is loaded.

What it teaches the butler, in order: settle the errand with the owner before
renting a phone (what to get, and the delivery or pickup address — the phone's
GPS is set there with `start --address`); install an app the phone provider's
library lacks from Huawei AppGallery through Chrome (the phone has no
Google account for Google Play; never an APK mirror); sign up or sign in on its **own** number
(`bevo-sms number`, codes from `bevo-sms otp`); work unlabelled screens through
`do`; and pay the way the owner chooses — cash, a saved method, or the owner's
own card, asked for only at the payment screen and typed only into that app's
card form — after an approval the owner gives for every order.

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

No `duty.py`, and the butler never files a duty for an app errand: each order
runs once, in a conversation, with its own approval.

## What stays in the container

The `app-checkout` command is a **container primitive**, not part of this skill:
`virtuals-agent` ships it as `src/integrations/butler/bin/app-checkout.mjs`, a
forwarder to bevo-server's phone rental (`/butler-exec/device-session*` and
`/butler-exec/app-action*`). bevo-server rents the phone — the provider key and the
device id never reach the container, which holds only a numeric session id — and
files every in-app order as an owner approval. The same command sets the phone's
GPS from a street address (`start --address`, geocoded with OpenStreetMap),
grants an app notifications ahead of time (`grant`; the provider refuses a
location grant, so a location prompt is tapped), and hands navigation on a screen
with no accessibility labels to the phone provider's own vision agent (`do`),
which is told never to order, pay, sign in or type a code. `bevo-sms` (the
butler's own number and its sign-in codes) and `bevo-notify` are container
commands too. This repository holds only the how-to: an app-specific flow is
prose the butler reads, never a step grammar the container interprets.

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
