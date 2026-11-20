# Echoes of the Forgotten Kingdom — Version 2 (Unity)

First-person 3D fantasy exploration and puzzle game set in the world of Elvareth. No guns, no
shooting. Version 2 is a Unity rebuild of the earlier Three.js browser game.

## Read first

- `docs/WORLD_ARCHITECTURE.md` — the foundation. Read it before proposing or changing anything
  structural. Check its Status line: while it says "awaiting owner approval", write no game code.

## Rules

- Communicate with the owner in Vietnamese. Write code and technical documentation in English.
- Keep **World Architecture** (`Assets/_Core/`) and **World Content** (`Assets/Content/`)
  separate. Content depends on core; core never references specific content.
- New places are added as data + content. If adding a location seems to require editing `_Core`,
  stop and raise it — that is an architecture gap, not a content task.
- IDs are permanent: never rename, reorder or reuse one. Saves must stay loadable; ship a migration
  instead of dropping progress.
- Condition and effect types are append-only.
- Work incrementally, plan before large changes, and wait for approval on plans.
- Never claim a feature works, performs well or sounds right without having actually run it. The
  cloud environment cannot run the Unity Editor: label anything not run in the Editor as untested.
- Reference images are art direction only. Do not commit third-party images or embed them in the
  game.
