# Roblox Popup Minigame System

A modular, proximity-triggered minigame framework for Roblox. Minigames are loaded automatically from a folder, shuffled into a FIFO queue, and chained together via a single `onFinish` callback — adding a new minigame requires no changes to the manager. However, it is unfinished, and was made to test my hand at minigames.

---

## Features

- **Auto-discovery** — any `ModuleScript` dropped into `GenMinigames` is loaded and queued automatically
- **Shuffled FIFO queue** — minigames play in a randomized order; each runs exactly once per activation
- **Callback chaining** — each minigame calls `onFinish()` on success, which triggers the next one with no polling
- **Retry on fail** — every minigame re-launches itself on failure, preserving the original `onFinish` continuation
- **Shared GUI template** — all minigames clone a single `MINIGAME_GUI` from `ReplicatedStorage` and hide unused frames
- **Proximity prompt trigger** — the queue starts when a player interacts with a `ProximityPrompt` in the workspace

---

## Project Structure

```
ReplicatedStorage/
├── MINIGAME_GUI              -- Shared GUI template (cloned by each minigame)
│   └── Main/
│       ├── SimonFrame        -- Simon Says UI
│       ├── TimedHitsFrame    -- Timed Hits UI
│       ├── MathFrame         -- Math Minigame UI
│       └── TypingFrame       -- Typing Test UI
└── GenMinigames/             -- Drop ModuleScripts here to register them
    ├── SimonSays
    ├── TimedHits
    ├── MathMinigame
    └── TypingTest

Workspace/
└── GeneratorMinigameTest/
    └── ProximityPrompt       -- Triggers the minigame queue on interaction
```

---

## Minigame Contract

Every minigame module must export a table with a single function:

```lua
local MyMinigame = {}
MyMinigame.Name = "My Minigame" -- optional, used for debugging

function MyMinigame.StartMinigame(rs, screenGui, onFinish)
    -- rs        : ReplicatedStorage reference
    -- screenGui : the GUI template (not used directly; each minigame clones MINIGAME_GUI itself)
    -- onFinish  : call this with no arguments on a successful completion
end

return MyMinigame
```

On failure, the convention is to re-call `MyMinigame.StartMinigame(rs, screenGui, onFinish)` directly, which retries without advancing the queue.

---

## Included Minigames

### Simon Says

A logic deduction game. Each round displays a sentence like:

> *"Simone says Red is not dangerous"*

The player must determine whether clicking the mentioned button is safe or a trap. The truth is derived from two independent flags XOR'd together:

| Commander trustworthy | Adjective `isSafeType` | Button is safe? |
|---|---|---|
| `true` | `true` | ✅ Yes — click the mentioned button |
| `true` | `false` | ❌ No — click any other button |
| `false` | `true` | ❌ No — "Simone" is lying |
| `false` | `false` | ✅ Yes — a liar saying something is unsafe means it's safe |

Fake commanders include typos, number substitutions, and negations (`Simone`, `SSimon`, `S1mon`, `5imon`, `Simon doesn't say`, etc.) to catch players reading too fast. Buttons, commanders, and adjectives are all shuffled at the start of each game. Runs for 3 rounds by default.

| Config | Default |
|---|---|
| `maxRounds` | 3 |

---

### Timed Hits

A rhythm game. Five boxes bounce vertically on screen in a staggered wave. The player must press **Space** to "hit" each box in order, left to right, while it overlaps the central midline. A successful hit freezes the box and turns it green. Missing a box or pressing early resets the minigame.

Overlap is checked using raw `AbsolutePosition` and `AbsoluteSize` comparisons — no raycasting or `GuiObject:GetRelativeOffset` involved.

| Config | Default |
|---|---|
| Tween duration per box | 0.4s |
| Tween stagger delay | 0.1s between each box |
| Total boxes | 5 |

---

### Math Minigame

A two-choice arithmetic quiz. Generates a random expression using `+`, `-`, or `*` with operands between 1–50 (subtraction always puts the larger number first to prevent negatives). One button shows the correct answer; the other shows a wrong answer offset by a random value between −10 and +10, guaranteed to be non-negative and not equal to the correct answer. Clicking the wrong button triggers an immediate retry.

| Supported operations |
|---|
| Addition (`+`) |
| Subtraction (`−`) |
| Multiplication (`×`) |

---

### Typing Test

Displays a random pangram-style sentence and asks the player to type it exactly. A live RichText feedback display renders each character green (correct), red (wrong), or grey (not yet typed) as the player types. The minigame completes the moment the input string matches the target exactly — no submit button needed.

Focus is captured automatically on start via `playerInput:CaptureFocus()`.

Sample sentences include standard typing-practice pangrams:
- *"The quick brown fox jumps over the lazy dog."*
- *"Sphinx of black quartz, judge my vow."*
- *"Waltz, bad nymph, for quick jigs vex."*

---

## Adding a New Minigame

1. Create a `ModuleScript` inside `ReplicatedStorage.GenMinigames`.
2. Follow the module contract above — export a table with `StartMinigame(rs, screenGui, onFinish)`.
3. Inside the function, clone `rs:WaitForChild("MINIGAME_GUI")`, hide the frames your minigame doesn't use, and call `onFinish()` on success.
4. On failure, call `MyMinigame.StartMinigame(rs, screenGui, onFinish)` to retry without advancing the queue.

That's it, the manager picks it up automatically on the next activation.

---

## Known Limitations & TODOs

- On retry, each minigame calls `StartMinigame` recursively rather than through the manager — deep fail loops could accumulate stack frames
- The `screengui` parameter passed into `StartMinigame` is shadowed by a new local clone inside every minigame; the original parameter is never used and can be removed from the contract
- `TimedHits` stores its `InputBegan` connection as a single value rather than a table, so `cleanup` calls `connections:Disconnect()` directly — this will break if the pattern is ever extended to multiple connections
- The queue is not refilled or reshuffled after it empties — a second `ProximityPrompt` trigger after all minigames complete does nothing
- `SimonSays` includes `"will kill you"` with `isSafeType = true`, which is intentionally a reverse bluff — this may be worth a comment in the data table for future maintainers
- No server-side validation — minigame completion is handled entirely client-side

---

## License

MIT — use freely in your own Roblox projects.
