# Changelog

## 2.0.0

**Breaking: rewritten for the Mastra butler (`virtuals-agent`).** 1.0.2 was
written for the retired OpenClaw runtime (bevo-docker) and cannot run there.

- **The description is a quoted string.** 1.0.2's unquoted description held
  `phone: SMS`, which is not valid YAML, so gray-matter (Mastra's parser)
  rejected the frontmatter and Mastra dropped the skill without an error.
  Frontmatter is now `name`, `description` (≤ 200 chars), `version` and a
  one-line `metadata` carrying `butler` alone: `moneyMoving`, the keyword list
  unchanged, `requires.bins` = `app-checkout`, `bevo-sms`, `bevo-notify`. The
  `openclaw` block, `tier`, `modes`, `routes` and `params` are gone.
- **The container's `app-checkout` grammar:** `start --app grab|zus|<package>`
  with `--country`, `--lat`/`--lon` and `--purpose`; `status`, `screen`, `shot`,
  `tap`, `type`, `key`, `swipe`, `wait`, `install`, `open`, `checkpoint`, `end`.
  The old `app-checkout phone` / `otp --type` are gone.
- **No params and no `bevo-hub`.** `APP_CHECKOUT_*` and `bevo-hub set/show` are
  removed: the country goes on `start` for each errand, the delivery address
  is typed into the app's own address search (ask the owner once, or recall
  it — a `request_location` answer is never passed to a command), and the
  sign-in country follows the number the butler signs in with.
  `bevo-read card-budget` is gone too.
- **Looking is an errand.** "Show me the ZUS menu on GrabFood" runs the same
  steps and stops before the basket: read the menu, `end`, send the items with
  their prices.
- **Every in-app order asks the owner** — bevo-server stopped auto-approving on
  2026-09-21, so the budget pre-read and the `auto_approved` branch are dead.
  The checkpoint is filed without `--wait`, then claimed with
  `--approval-id <id> --wait 90`, repeated until approved or declined (each
  claim keeps the phone alive), and filed by about minute 15 of the 25-minute
  rental. An approval never moves to a new phone.
- **Grab first.** The procedure walks GrabFood → ZUS Coffee end to end, names
  the Grab and ZUS packages, and handles a missing app: `install`, then `open`
  every ~30 s up to 4 times, because `open` can report success for an app that
  is not there.
- **Sign-in on the butler's own number:** `bevo-sms number`, the matching
  country in the app's picker (+1 → United States), `bevo-sms otp --since`
  the moment "Send code" was tapped, then `app-checkout type` the code.
- Sections follow the new skill standard: `## Procedure` replaces
  `## Customize` and `## One-off procedure`. Body 9,146 → 9,950 chars; the
  whole file 11,292 → 10,873, since params and routes left the frontmatter.

## 1.0.2

Wording only: the standing-order note now says "walk through the errand once
by hand," since the butler's earlier practice-run mode is gone. No command
string changed.

## 1.0.1

**Trimmed to just enough context.** SKILL.md 14,234 -> 11,370 chars (-20.1%), README
2,820 -> 2,343 (-16.9%). Explanation only — every `app-checkout`, `bevo-read`,
`bevo-notify` and `bevo-hub` command string is byte-identical (verified by diff).

- Step prose that restated the command's own flags is gone; the flags are the rule.
- The `--wait` / `--since` behaviour is stated once instead of three times.

## 1.0.0

- The phone rail as a skill. `app-checkout` — a cloud Android phone brokered by
  bevo-server — shipped as a container primitive in bevo-docker#138; the playbook
  that went with it was cut from the image before merge so it could live here.
- Written from the shim's real contract (15 subcommands, no recipe runner) and
  the two GrabFood MY flows the PR rehearsed, as plain numbered steps rather than
  a step grammar the container would have to interpret.
- Restructured onto the SKILL_STANDARD sections and cut to a delta over
  AGENTS.md: the purchase budget, untrusted content and the duty rule are cited
  from § 13/§ 14/§ 5 instead of restated.
- The owner never hears about the phone. Every failure line is an errand the
  butler could not run, never a session, a rental or a switch they do not have.

Rules that came out of reading the server and the shim rather than the draft they
replaced — each one is a wrong order or a burnt approval if it is missing:

- **`checkpoint` is filed with `--wait 0`.** Its default is to block for 15
  minutes polling the approval, during which it sends the phone nothing — and
  six idle minutes releases the phone. On any approval tapped after that, the
  shim consumes the approval and reports "place it now" onto a device the server
  has already taken away. File, keep the phone alive with `screen`, then claim
  with `--approval-id <id> --wait 60`.
- **`otp` is passed `--since`.** Without it the first poll returns the newest
  code already in the inbox — on a second sign-in, the previous one — and
  `--type` types it in to be rejected. The number is shared with every other
  rail, so a stale code is always there.
- **Every rental is a fresh phone.** `profile_ref` is read but never written
  server-side and the pool's only tier is ephemeral, so nothing is signed in and
  the sign-in is part of every errand. (The container's own toolbox row still
  claims the opposite.)
- **The checkpoint amount is the final total** — after delivery, service fee and
  tip — not the item subtotal. Nothing downstream catches an understated one.
- **Never issue a card for an in-app order.** `checkpoint` already put it through
  the owner's purchase policy; a card puts the same order through it again.
- **`--country` sets the region and a capital-city GPS fix, not the delivery
  address.** `APP_CHECKOUT_LAT`/`APP_CHECKOUT_LON` carry the real one.
- `tap --text` is an unanchored case-insensitive regex, so `"Allow"` also matches
  "Don't allow"; `--nth` is 0-based; `type` goes to whatever has focus, which on
  a fresh screen is nothing; `screen` lists only what is currently visible.
- The pre-shopping warning names all three ways an order stops — Asks-first (the
  default, and unrelated to price), over the per-purchase cap, over the day's
  remainder — and never converts a local total against a USD cap.
- Keywords chosen for the hub scorer, which only recommends on a distinctive name
  or keyword hit: measured at 0 misses over 24 phone-app asks and 0 hijacks over
  25 trade/read/website controls, with the browser rail still ranking first on
  every website ask. `buy`, `sell` and `trade` are generic tokens there and never
  qualify a skill.
