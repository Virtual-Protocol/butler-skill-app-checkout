# Changelog

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
- Safety rules the bundled draft did not have:
  - **Never issue a card for an in-app order.** `checkpoint` already prices the
    order against the owner's purchase budget and meters it in the same ledger
    as `acp card issue`, so paying the same order with a butler card would draw
    that budget down twice for one meal.
  - One checkpoint covers exactly the order it priced — a surge fee, a tip or an
    added item is a different order and needs a fresh one.
  - `--kind confirm` is named for irreversible non-purchases, which always ask
    because the caps cannot size them.
  - A total in a currency bevo-server cannot price always asks; the butler never
    converts one itself.
- Keywords chosen for the hub scorer, which only recommends on a distinctive
  name or keyword hit: the app names (`grab`, `grabfood`, `foodpanda`, `shopee`,
  `gojek`), the errands (`lunch`, `coffee`, `groceries`, `delivery`) and the
  recovery path from a blocked website. `buy`, `sell` and `trade` are generic
  tokens there and never qualify a skill.
