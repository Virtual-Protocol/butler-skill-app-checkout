# app-checkout — flags and limits

- `start --app`: `grab`, `zus`, or an Android package — Grab is
  `com.grabtaxi.passenger`, ZUS `com.coffee.love_coffee`.
- `key`: `back`, `home` or `enter`.
- `swipe`: a direction (`up` shows more of a list), or
  `<x1> <y1> <x2> <y2> [--ms N]`. Swipe from mid-screen, not the bottom edge:
  apps float buttons there.
- `wait --text "<regex>" [--timeout S]`: at most 100 s.
- `tap --text "<regex>" [--nth N]`: case-insensitive, matched anywhere in the
  label — anchor it (`"^Allow$"`, or it also hits "Don't allow") and escape
  `? . ( ) +` (`"call you\?"`). `--nth` is 0-based.
- `do "<instruction>" [--schema '<json object>'] [--steps N] [--timeout S]`:
  `--steps` at most 60, `--timeout` at most 100 s. Say where to go and what to
  bring back, e.g.
  `app-checkout do "open the menu, find Iced Latte, read its sizes and prices" --schema '{"type":"object","properties":{"sizes":{"type":"array","items":{"type":"string"}}}}'`.
  "still working on it (task <id>)": `app-checkout do --task <id>` until done;
  `do --task <id> --cancel` stops it.
