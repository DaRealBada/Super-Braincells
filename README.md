# Braincell

A slot-machine for study questions. Pull the cord, land on a question, research it for a set time, then say your answer out loud against a clock.

Live: https://super-brains-darealbadas-projects.vercel.app

## What it does

- **Three modules** — HCI, Cloud Computing, ERP, or Random across all three.
- **Two reels** — *Questions* (full research questions) or *Concepts* (short concept titles).
- **Physical trigger** — drag the brain pendant down; the elastic cord stretches, snaps back, and the snap fires the spin. A Spin button does the same thing.
- **Weighted reel** — motion-ticked deceleration that lands on a random row and never repeats the row it started on.
- **Two-phase clock** — a research countdown, then a speaking countdown, with pause, done, restart and reset.
- **Settings** — research length (5–30 min), speaking length (1–3 min), spin length, sound, ghost previews, and auto-start after a spin.

## Files

| File | Role |
| --- | --- |
| `index.html` | The whole site — markup, styles, logic, favicon. No build step, no dependencies. |

Fonts (Cormorant Garamond, Lora) and the brain glyph load from CDNs; everything else is inline.

## How it works

```mermaid
stateDiagram-v2
    [*] --> Idle

    Idle --> Spinning : pull cord released past threshold<br/>or Spin pressed
    Spinning --> Idle : reel settles on a row<br/>(question landed)

    Idle --> Researching : Start research
    Spinning --> Researching : auto-start enabled

    Researching --> Researching : Pause / Resume
    Researching --> Finished : Done<br/>(clock forced to 0:00)
    Researching --> Speaking : countdown reaches 0:00
    Finished --> Speaking : Speak now

    Speaking --> Speaking : Pause / Resume<br/>Restart
    Speaking --> Idle : countdown reaches 0:00

    Researching --> Idle : Reset
    Speaking --> Idle : Reset
    Finished --> Idle : Reset

    note right of Idle
        Cord, module picker and
        mode switch are live only here
    end note
```

### The spin

```mermaid
flowchart TD
    A[pointerdown on cord or brain] --> B{phase is idle?}
    B -- no --> C[dull thud, ignore]
    B -- yes --> D[track drag: stretch d, sway angle a]
    D --> E[cord thins and lengthens<br/>reel nudges down with it]
    E --> F[pointerup]
    F --> G{d >= 42px?}
    G -- no --> H[spring back, no spin]
    G -- yes --> I[thwack, spring back]
    I --> J[spin fires as cord passes rest]
    J --> K[pick random target row != start row]
    K --> L[3-5 full revolutions + delta<br/>quartic ease-out over ~4.6s]
    L --> M[snap to row grid, chime, lock in question]
```

Every exit path — short pull, blocked spin, failed spin — restores the reel to the row grid, so the reel can't be left mid-row.

## Editing the question bank

All content lives in the `DATA` object at the top of the `<script>` block:

```js
DATA = {
  "Module name": {
    research: [ "question", … ],   // main question set
    wider:    [ "question", … ],   // appended to research in Questions mode
    concepts: [ ["Concept title"], … ]  // Concepts mode
  }
}
```

Add a module by adding a key, then adding its name to the `CATS` array (keep `"Random"` last).

## Deploying

Static single file. Upload `index.html` to any host, or point Vercel/Netlify at this repo with no build command and the repo root as the output directory.
