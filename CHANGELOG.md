# Changelog

## 2.1.1

- **The card is asked for at the payment screen, never before renting the
  phone** (owner, 2026-09-23). Cash, else the method already on the account;
  with neither, or when the owner asked to pay by card, the butler says the
  total, asks for the card and ends its turn — the phone waits about five
  minutes. On the reply it loads this skill again (a new turn starts at the
  default step limit) and carries on from `screen`, or restarts and rebuilds
  if the phone was released.

## 2.1.0

- **Your owner can pay with their own card** (owner's call, 2026-09-23: "the
  skill should not block user from entering card for purchase"). 2.0.0
  refused every card and ended the errand when an app took neither cash nor a
  saved method. Now the butler asks for the card in the chat before renting
  the phone and types it only into that app's card form. It never hands the
  card to `do`, never repeats it ("card ending 1234"), unticks "save card",
  pays only after the checkpoint is approved, and leaves the bank's
  confirmation to the owner.
- **An app missing from the cloud phone's app library installs from its
  official store in Chrome.** `install` now answers "not in the cloud-phone
  app library" (it used to read as a lost phone, and the server orphaned the
  phone). ZUS comes from Huawei AppGallery's own download; never a mirror.
- **Permission prompts are tapped, not pre-granted.** The provider refuses
  location grants (422, measured 2026-09-23), so `open` no longer claims to
  grant location; `grant` is for notifications.
- The daily phone time is an allowance `status` reports, not a fixed 60
  minutes; `device_pool_busy` (every phone in use) is "try a bit later".

## 2.0.0

**Breaking: rewritten for the Mastra butler (`virtuals-agent`), as the generic
playbook every phone-app skill builds on.** 1.0.2 was written for the retired
OpenClaw runtime (bevo-docker) and cannot run there.

- **Generic, not Grab-first.** The Grab Food walk-through moved to its own
  skill, `butler-grabfood` 1.0.0, which declares this one in
  `requires.skills` (owner's design, 2026-09-23: Grab is a super-app and food
  is one of its sections). This skill keeps what every app shares — the
  phone's clock, making sure the app opened, signing in, reading the screen,
  and every money rule — and its app step reads "if a skill for this app is
  loaded, follow its steps; otherwise work the app from `screen`, and with
  `do` where there are no labels". Food and Grab-food keywords moved there
  too; this one keeps the phone-app words, the apps with no skill of their own
  (ZUS, foodpanda, Shopee, Lazada, Gojek, Deliveroo, GrabCar, Grab Mart,
  rides) and the sign-in words.
- **The phone starts at the delivery address.** `start --address "<street
  address>"` sets the phone's GPS there, because apps rank shops by distance;
  the same address is still typed into the app. It is the owner's street
  address in words — a `request_location` answer never reaches a command. A
  start that prints "still starting" is run again as it was.
- **No labels → `do`.** Measured on a real rented phone (2026-09-23): ZUS
  Coffee exposes no accessibility labels, so `screen` prints nothing and
  tap-by-label cannot work there. Instead of guessing coordinates from a
  `shot`, the skill hands navigation to the phone provider's own vision agent
  — `app-checkout do "<reach X, read Y>" --schema '…'`, continued with
  `do --task <id>` in the same turn. `do` never orders, pays, signs in or
  types a code; the butler reads the total through it and taps Place order
  itself, once — by label, or at the point `do` read for it.
- **Permissions without a dialog.** `open` grants the app location before it
  opens, and `grant <package> location|notifications` covers a prompt nobody
  can read (Android's own location dialog is unlabelled too).
- **Never a duty.** Each order is one errand, run now, with its own approval.
- **Every in-app order asks the owner** — bevo-server stopped auto-approving on
  2026-09-21, so the budget pre-read and the `auto_approved` branch are dead.
  The checkpoint is filed without `--wait`, then claimed with
  `--approval-id <id> --wait 90`, repeated in the same turn until approved or
  declined (each claim keeps the phone alive), and filed by about minute 15 of
  the 25-minute rental. An approval never moves to a new phone; a Place order
  that did not go through is never retried on the spent approval.
- **Sign-in on the butler's own number:** `bevo-sms number`, the matching
  country in the app's picker (+1 → United States), `bevo-sms otp --since`
  the moment "Send code" was tapped, then `app-checkout type` the code.
- **Looking is an errand**, stopped before the basket.
- **The description is a quoted string.** 1.0.2's unquoted description held
  `phone: SMS`, which is not valid YAML, so gray-matter (Mastra's parser)
  rejected the frontmatter and Mastra dropped the skill without an error.
  Frontmatter is now `name`, `description` (≤ 200 chars), `version` and a
  one-line `metadata` carrying `butler` alone: `moneyMoving`, `keywords`,
  `requires.bins` = `app-checkout`, `bevo-sms`, `bevo-notify`, and
  `maxSteps: 150` (the steps a turn that loads it may take). The `openclaw`
  block, `tier`, `modes`, `routes` and `params` are gone.
- **No params and no `bevo-hub`.** `APP_CHECKOUT_*` and `bevo-hub set/show` are
  removed: the country and the address go on `start` for each errand, and the
  sign-in country follows the number the butler signs in with.
  `bevo-read card-budget` is gone too. The old `app-checkout phone` /
  `otp --type` are gone.
- `do` and `grant` are written in prose rather than in a shell block until the
  hub validator's `app-checkout` subcommand table lists them.
- Sections follow the new skill standard: `## Procedure` replaces
  `## Customize` and `## One-off procedure`. Body 9,146 → 11,939 chars
  (12,000 is the cap).

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
