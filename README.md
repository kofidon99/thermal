# Thermal

A one-button glider. Hold to dive, let go to climb, and read the air — because
the day is short. Single HTML file, no build step, no dependencies, no external
network requests.

## Putting this online (about 5 minutes)

1. **github.com/new** → repository name `thermal`, **Public**, create.
2. **Add file → Upload files** → drag in `index.html`, `.nojekyll` and this
   `README.md` → **Commit changes**.
3. **Settings → Pages** → Source **Deploy from a branch**, Branch **main**,
   folder **/ (root)** → **Save**.
4. Wait a minute, then open `https://YOUR-USERNAME.github.io/thermal/`.

Tag each released version (`v1`, `v2`) so the exact code behind a Playables
submission stays pinned.

---

## Controls

| Action | Touch | Keyboard |
|---|---|---|
| Tuck and dive | Hold anywhere | Hold `Space`, `Enter`, `S` or `↓` |
| Spread and climb | Let go | Release |
| Pause | Pause button | `Esc` or `P` |
| Mute | Speaker button | `M` |

One button, two airspeeds. It is identical on every difficulty.

## The hook — you can see the air

Height and speed are the same thing in two pockets. Tucking moves it into
speed, spreading moves it back into height, and drag takes a cut both ways —
a much bigger cut the faster you go. That is the whole flight model, and it is
the real one.

What makes it a game is that **the air is drawn**. Warm chevrons rise where the
air rises, cool ones fall where it falls, and the brighter they are the
stronger it is. A row of cumulus sits on top of every street of lift, which is
exactly how you find one from a distance in a real glider. There is nothing
about the sky that the sky does not tell you.

Two instruments say the same thing in numbers:

- **AIR** on the left is a netto variometer: what the air itself is doing, not
  what you are doing to yourself. The tick on it is the break-even line — above
  it the air lifts you faster than the wings let you down.
- **The dashed line** ahead is your glide range in still air. Where it ends is
  where you would touch down. It goes red when it no longer reaches the next
  street, which is the moment to stop being clever.

The strip along the bottom shows the ground and the streets ahead. A phone sees
less sky than a desktop, so without it the strategy would be easier on a big
screen; the instrument gives both the same information.

## The day

**A flight is one soarable day — 150 seconds.** The sun comes up on the left,
crosses, and goes down on the right; when it lands, the day is over and your
distance stands. Lift dies away through the last quarter, so the end of a
flight is a scramble rather than a cut.

The clock is not decoration. It is the reason the button is worth pressing —
see *Measuring the fun* below.

## The flock

You fly with a flock of swifts. Touch the ground and one goes under you,
carries you back up to a working height, and leaves. Out of swifts, out of day.

**A swift joins you every time you top a street out at cloud base**, up to the
ceiling shown by the hollow pips. That is the only way to gain one, so you
cannot play safe into safety — the way to earn margin is to go up and use it.

## Difficulty

One measured number: **slack** — how far you can glide from cloud base compared
with how far it is to the next street of lift.

```
slack = cloud base × (best glide speed / sink at that speed) / gap
```

| | Slack | Street gap | Lift | Swifts (max) | Pays |
|---|---|---|---|---|---|
| **Cruise** | 2.6× | 1500 | 150 | 4 (6) | ×0.7 |
| **Drive** | 2.0× | 1980 | 165 | 3 (5) | ×1.0 |
| **Redline** | 1.7× | 2350 | 180 | 2 (4) | ×1.45 |

At 2.6 you can waste a whole glide and still arrive. At 1.7 you have to leave
every street near the top and fly the gap properly. Gaps widen and lift fades
as the day goes on, so the numbers above are where a flight *starts*.

**Nothing about the control changes between tiers** — same hold, same two
airspeeds, same response. All the difficulty is in the air. Cloud base, the
polar and the flight geometry are **fixed logical sizes**, not viewport
fractions, so a 9:20 phone and a 16:9 desktop fly identically.

Each tier keeps its own best score, best distance and most streets topped.

## Playables compliance notes

- **Initial load ~71 KB**, one file. Limit is 30 MB.
- **Zero external requests.** All art drawn procedurally on canvas, all audio
  synthesised with Web Audio. Verified in `test/run.js`.
- **No copyrighted assets** — no image or audio file in the bundle.
- **Scales to 1:1, 16:9 and 9:16.** Screenshots in `test/shots/`.
- **60 fps** at phone and desktop resolutions, measured under load.
- **`firstFrameReady()` then `gameReady()`**, in that order.
- **Pause and mute obeyed immediately.** The wings are released on the way into
  a pause, so a held finger can never dive you while you are away.
- **Progress saved through `saveData` / `loadData`**, localStorage as fallback.
- **No ads wired up yet.**

## Repo layout

```
index.html              the whole game
.nojekyll               serve files as-is
thermal-playables.zip   bundle for the developer portal
src/body.html           source of truth
build.js                wraps src/body.html into index.html
test/driver.js          the auto-pilot the other tests share
test/run.js             aspect ratios, external requests, input, pause, perf
test/sdk.js             integration against a mocked ytgame SDK
test/gameplay.js        flock ledger, air / speed-to-fly / ceiling differentials
test/probe.js           measures how far a day actually goes
test/bisect.js          disables one draw phase at a time and measures
test/shot.js            screenshot capture
```

`node build.js` rebuilds. `node test/gameplay.js 3` runs one section.

### Testing a flight model without debug hooks

The shipped build has no test affordances, so the tests fly it the way a person
does: `test/driver.js` reads one column of pixels down the variometer — the
same needle the player watches — and works the button from that. It has four
policies: never tuck, always tuck, tuck where the air is going down past the
crossover, and a learner that has the right instinct and is wrong a third of
the time.

The correctness backbone is an identity. A swift is spent by exactly one
landing and won back only at cloud base, and the flight ends when the last one
goes, so on a flight that ends that way:

```
landings  ===  the tier's starting swifts  +  swifts won
```

Each rule is then tested as a **differential** — the same build with one thing
changed, flown by the same auto-pilot:

| Claim | Differential | Result |
|---|---|---|
| the air holds you up | streets and ripple switched off | 435 m with air, 106 m without, cloud base never reached |
| cloud base is the reward | ceiling raised out of reach | 3 swifts won vs 0, same air |
| speed-to-fly is worth something | three policies, one short day | 33% further in the same day |

#### Two traps, both in the instruments

**The variometer used to show total energy** — the air minus your own sink —
which is what a real glider's main instrument shows and reads beautifully. It
is also quietly circular: diving makes the needle plunge, and a needle that
says "you are going down" while you are diving says "dive harder". The test
auto-pilot fell straight into it and held the button until it hit the ground.
A player would have too. The bar is netto now: it shows the air, which is also
what the sky is painted to show, so the instrument and the view agree.

**The auto-pilot used to desynchronise.** Input is ignored while a companion is
carrying you back up, so a driver that only dispatches on a change of mind
believes the button is still down, the game knows it is not, and the glider
spends the rest of the flight at best glide. Hold is re-asserted every few
frames now.

### Measuring the fun

`test/probe.js` answers a design question rather than a correctness one: **how
far does one day go, and does flying it well look different from flying it
badly?**

But the probe that mattered most here was an offline one. Before the day clock
existed, the score was distance and nothing else, which makes the objective
*distance per unit height* — and the answer to that is always "fly slowly". A
model of the flight physics, swept across polar shapes, top speeds, response
rates and look-ahead, put the best possible speed-to-fly policy **2 to 4 per
cent** ahead of never touching the button at all. A strategy worth 3% is not a
strategy; it is a rounding error with a button attached.

Adding a clock changes the objective to *distance per unit time*, which is what
speed-to-fly theory is actually about, and the same model put a pilot who reads
the air about 20% ahead of one who never tucks and several times ahead of one
who never lets go. Measured in the real build over one short day: **33%
further**. The day is not a framing device — it is what makes the button worth
pressing.

| | Distance | Speed | Streets topped |
|---|---|---|---|
| reads the air | 1,231 m | 99 km/h | 2 |
| never tucks | 923 m | 74 km/h | 3 |
| never lets go | 548 m | 134 km/h | 0 — down at 14 seconds |

## Performance

60 fps at every resolution on the first measurement, because the lessons from
the earlier games were already in place: the sky is baked once per layout at
real canvas resolution and blitted 1:1, the sun's halo is authored at exactly
the size it is drawn rather than stretched, every sprite is cached, and no
gradient is built per frame. The painted air is the one thing drawn per column
per frame, and it is flat fills and cached chevrons for that reason.

`test/bisect.js` — disable one draw phase at a time and measure — was written
before the first optimisation and went unused again.
