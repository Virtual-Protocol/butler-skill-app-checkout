# Changelog

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
