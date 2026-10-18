# Marvel vs. Capcom: Clash of Super Heroes — features and roadmap

Where this port stands against a finished one, and what each missing piece needs. The reference is the
toolkit's own complete example, [vampire-savior-recomp](https://github.com/tmcleroy/vampire-savior-recomp)
(`games/vampire_savior/src/`), and the toolkit's documents `docs/FEATURES.md` and `docs/FIGHT.md`.

Everything below is an **optional feature**: with every feature off, the port must run exactly as the
board does. Nothing from the game (no ROM, no RAM dump, no ripped art/text) may ever be committed — each
thing found is recorded in `PROGRESS.md` as *how* it was found (a RAM diff, a write watch, a trace),
never as the data itself.

## What works today

From the toolkit, with no game work:

- The **launcher**: chooses the ROM set, checks every file by CRC-32, sets the controls, Play.
- The **in-game menu** (Esc, `F1`, a stick's click, or Start+Select): save states (9 slots), Reset,
  Insert coin, Player 2 joins, Quit game.
- **Stop, step, slow** (P, O, `-`/`=`).
- **Run-ahead** and the **Display** options (scaling, picture shape, window size, full screen).
- Every key and button **remappable**, for each player.

The game's own memory is **not yet mapped**: this first build is the translation of the board's program
(68000 and Z80) with `make explore`, and no addresses in the game's RAM have been found yet. Everything
in the next sections is still to do.

## What a finished port adds (this is the work)

### 0. Foundation — know the game's memory

`games/mvsc/src/` will name, from addresses found on the running board:

- the **credits** word (so Start can work with no coin, and an attract watcher is possible);
- the two **player records**: health (and what the bars are drawn from), the place across the stage,
  the facing, the super (meter) gauge, and **the state that says what a player is doing** (standing,
  walking, crouching, in the air, attacking, reeling), which the frame data, the dummy and the hit boxes
  want;
- the **round's clock**;
- the **attack boxes** themselves, for the hit-box display.

How the toolkit finds these (all in `docs/FEATURES.md`):

- `--dump-ram F:FILE` + `tools/ramfind.py DUMP=VALUE ...` — addresses that held a known value across
  several dumps.
- `--trace LOW-HIGH:FILE` with a run that does known things at known frames.
- `--watch LOW[-HIGH]` — every write to an address and the instruction that made it, read against
  `work/mvsc/program.lst`.
- `--poke ADDR=BYTE,...` to try a guess; `--menu FRAME:KEYS` to press the menu's own rows;
  `--shot-every N:PFX` to see a run.

### 1. The Features page (the game's own switches)

| Feature | Kind | Needs |
| --- | --- | --- |
| `free_play` | on/off | the credit word |
| `no_damage`, `no_damage_2` | on/off | each player's health and the round's full value |
| `infinite_time` | on/off | the round's clock |
| `one_hit` | on/off | the opponent's health |
| `full_gauge` | on/off | the value behind each player's super meter |
| Board/operator settings (difficulty, round time, free play) | choices | reading the EEPROM's operator settings |
| "The game's own test menu" | command | a routine address to jump to |

### 2. Training mode (the headline feature)

| Piece | What it adds | Needs |
| --- | --- | --- |
| Input display | what the stick holds, and for how long | the facing (from the two players' places across the stage) |
| Frame data | startup, active, recovery, and the advantage after a hit | the state that says a move is out, per player |
| Combo readout | hits, damage, and what of it comes back | the damage rule (a rewrite) |
| The dummy (player 2) | stand, crouch, jump, guard, record and replay | the game's idea of a round and of guarding |
| Hit boxes | the players' and projectiles' boxes, drawn on the picture | the box tables and the camera's place |

The toolkit's `fight/` parts (`fight/fight.h`) give all of these once the game's own state is known.

### 3. Alternative modes

Each is a command with a page of its own: an **attract watcher** (the credit word held at zero), and an
**endless** game (the round/match state). A two-machine link is not possible (the board's link hardware
is a separate build).

### 4. Widescreen

A switch on the launcher's first page, off in every preset (it can change how the game plays). Needs
rewrites of the camera and the background scrollers and of the objects that must be kept while they are
in the wide picture. The toolkit's `widescreen.cpp` in Vampire Savior is the example.

### 5. Readable rewrites (correctness, and what the above builds on)

Rewrite the game's key routines by hand as named C++ (`docs/READABLE.md`): the players' movement and
collision, damage, throws, and the round's advance. This is what makes a clean `one_hit`, the training
mode and the widescreen possible, and it is the part most worth doing carefully.

## Recommended order

1. **The foundation** — the credits, the clock and the player records (health, position, facing, state,
   gauge); everything else builds on it.
2. **The Features page** — free play and the cheats, small and self-contained.
3. **Training mode** — the input display first, then the frame data, the combo readout, the dummy and the
   hit boxes, on the toolkit's `fight/` parts.
4. **Readable rewrites** — needed by 3 and 5, and correctness.
5. **Widescreen.**
6. **Alternative modes.**

## Before uploading: a checklist

- [x] `README.md` states the exact ROM set (`mvsc`, Europe 980123) and that **only that revision** works,
      with the Capcom disclaimer.
- [x] `rom/`, `build/`, `work/` are ignored and **absent** from the repository; no game data, keys or dumps
      anywhere (`.github/workflows/check.yml` fails a push that tracks one); no screenshots.
- [x] `PROGRESS.md` records how each finding was made (a RAM diff, a watch, a trace), not the data.
- [ ] `make check`, `make check-toolkit`, `make check-common` all pass (the CI runs the last two).
- [x] `LICENSE` (GPL-3.0-only) and `THIRD_PARTY_NOTICES.md` (the cipher is MAME's, BSD-3-Clause); the
      release archives carry them and the licences of what they ship.
- [ ] Repo description set (see README's title line / the About field).
- [ ] First commit made and pushed; only source, docs and tooling are tracked (no ROM, key or derived
      data — `rom/`, `work/` and `build/` are ignored).
- [ ] The repository builds from a fresh clone with the player's own ROM set (`make`, `make explore`,
      `make launcher`).
