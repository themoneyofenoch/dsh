# DESIGN.md — Ammaniel's design language (locked 2026-10-18)

Read this before drawing a screen, a card, a flyer, an icon or a store page.
It is what he approved on Latindance after many rejected rounds. It works for
any app with the same brand.

## 1. The one rule he keeps saying

> **Black field, colour only on the lines and the buttons.**

The page is a near-black **#000000** (not grey, not violet, not a coloured wash).
White type. The five neon colours live on **borders, bars, dots and buttons** —
never as a full-page background tint.

Rejected and why:
- cream / white page → too plain, "not wow" for this brand
- pure flat black with no colour → dead
- coloured aurora wash over the whole page → reads as "dark purple background"
- warm grey `oklch(0.3 0.015 285)` page → reads violet; must be #000000

## 2. The palette — five stops, no others

```
mint   #2ee6a8
cyan   #22d3ee
yellow #ffd447
orange #ff9e3d
pink   #ff3d6e → #ff3d7f
violet #c22bff
```

The main run (used everywhere, 135°):

```css
--ld-neon: linear-gradient(135deg,
  #2ee6a8 0%, #22d3ee 16%, #ffd447 34%, #ff9e3d 48%,
  #ff3d6e 62%, #ff3d7f 76%, #c22bff 100%);
```

The three bars of a designed flyer / a button pair:

```css
.ld-bar-a { background: linear-gradient(90deg,#2ee6a8,#22d3ee 62%,#4b2bff); }
.ld-bar-b { background: linear-gradient(90deg,#ffd447,#ff9e3d 58%,#ff3d6e); }
.ld-bar-c { background: linear-gradient(90deg,#ff3d7f,#c22bff); }
```

Card fill `#0A0A0C`; the lifted card grey he asked for is **`#1A1A21`**, rows `#1E1E26`. The bar/nav surface is the softer **dark grey `#0B0B0D`** — he picked that grey himself ("the dark grey you have on the nav top border i like it", m01099), so bars and top bands use it, never pure black. Muted text `#A7A3B3`.

**The button pair he settled on (m01483, live):** blue → mint, dark ink letters, no veil:

```css
.ld-grad, .ld-mint { background-image: linear-gradient(to right, #2bc0ff, #a1ffce); color: #06303a; }
```

The old five-stop rainbow is **not** used on buttons any more — he called it ugly. It stays on flyer art, the nav top line and the icon.

Reference file with the exact stops:
`/Users/nakfaai/Developer/projects/facelink/assets/logo-sunset.svg`

## 3. Borders: none

On the card pages (My AI, Events, Hub, More) there are **no outlines**.
- chips = flat `rgba(255,255,255,0.10)` fill, no border
- cards = flat `#0A0A0C`, radius 18–24, soft shadow only
- buttons = **filled** gradient (`.ld-grad`, `.ld-bar-a/b/c`), never an outline
- if a line is wanted, it is a 2px gradient run under a title or the tab bar top

The only outline allowed is the 1px `rgba(255,255,255,0.06)` hairline for depth.

## 4. A card — the anatomy he approved

One night = one card:

1. flyer fills the frame, 4:5, `object-fit: cover`, rounded 18–20
2. **date chip** top-left (cyan, `WED, OCT 21` in caps)
3. **time chip** top-right (mint)
4. title, white, 15–17px bold, two lines max
5. venue · city, muted, 12px
6. style pills, amber text on `rgba(255,255,255,0.10)`
7. price, green `#2ee6a8`
8. one action row: **I'm going** (filled gradient) · **Details** (grey fill)

A big hero card = the same anatomy at ~86% of the phone width, near full screen.

Real flyer first: **if the organizer has a flyer, show it. If not, show the
designed neon art** (three gradient bars + big day number + month). Never show
a stock portrait or a random photo — that is what he called ugly.

## 5. Type

- headings: white, bold, tight (`-0.02em`), 22–28px on a phone
- labels/chips: 10–11px, uppercase, `0.08em` tracking
- body: 12–13px, muted `#A7A3B3`
- brand wordmark: `latindance.` lowercase with the dot

## 6. Layout — the first page (approved m00293, built)

Top to bottom, one screen plus a scroll:

1. brand row — LD icon 28px + `latindance.` + city pill `📍 Dallas · 60 mi ▾`
2. greeting — "Good evening" + one line: `186 nights within 60 miles · 1 tonight`
3. **one big night card**, almost filling the page
4. dots underneath, active dot a 20px amber pill
5. under it, rows he asked for by name: **Favorite · Today · Weekend · Near you**
6. same cards everywhere; horizontal tracks only where he asked to swipe

City = a **60 mile radius** around the city centre **plus the city name** in the
same request — coordinates alone pull unmeasured events from other countries
(Mexico showed up in Dallas this way).

## 6b. The centre button = My AI, and the plan

- the middle button of the tab bar is **My AI**; it carries a small **white "1"** bubble until a plan exists
- tapping it opens the **AI plan** popup: "When do you want to dance?" → "And where?" → the count, the first night and a 14-day calendar with dots
- the same popup also opens on its own **3 minutes** into the app when no plan is saved (`ldPlan` in localStorage)
- popup colours: blue → mint on the selected chip and the action; body `#111115`; never the five-stop rainbow
- no "Plan my week" pill in the page header — the plan lives on that middle button

## 6c. The tab bar

- sits **flush on the bottom edge** (bottom 0, left 0, right 0), rounded top corners only, `#0B0B0D` on dark pages
- page content keeps `pb-[calc(6.5rem+env(safe-area-inset-bottom))]` clear of it
- a 2px `var(--ld-neon)` line across its top edge with a blurred copy under it

## 7. Motion

- 2 second logo flash when the app opens, once per session
- cards rise 8px + fade in, 220ms ease-out, staggered 40ms
- no bounce, no spin, no long easing

## 7b. The event page (locked m01765, built 20261018-221721)

The flyer **is** the page:

1. the flyer starts on the **very top edge** — no header bar above it, no black band, no rounded top corners
2. only a round **left arrow** floats top-left over the flyer (black/45 blur, white arrow) — never "← Back to Events" text
3. **date chip** top-right, **heart** beside it, both on the flyer
4. **I'm going** sits over the bottom of the flyer (filled, full width, rounded pill)
5. the photo carries **no title, time, price or dance icons** — the flyer already says them; those repeat below the card
6. below the flyer: who's going, song request, find a partner, then the facts
7. the card keeps its 4:5 frame at full phone width (`aspect-[4/5]`, `object-fit: cover`), so no letterbox bars

## 8. Store and icon

- app icon may sit on **black**; app screens stay black for this brand
  (the cream/light rule applies to other brands' documents, not to Latindance)
- icon: big LD letters filling 90% of the tile, the five stops across them
- store screenshots: real screens, real flyers, phone width, no blurry photos

## 9. Words

- never a negative word — write the want ("Nights find you", not "Don't miss out")
- numbers first in any status line
- no jargon in the UI; a button says what it does (`I'm going`, `See all`)

## 10. Check before sending

- [ ] field is #000000, no violet tint
- [ ] colour only on lines/buttons/dots
- [ ] zero borders
- [ ] white text ≥ 4.5:1 on every fill
- [ ] flyer or designed art on every card — no stock portraits
- [ ] no negative words
- [ ] event page: flyer touches the top edge, arrow only, date + heart on the flyer
- [ ] tab bar flush with the bottom edge, content clear of it
- [ ] one screenshot sent to him for approval **before** building a new screen

## Where this lives in Latindance

- tokens + classes: `/Users/nakfaai/Developer/projects/latindance/src/app/globals.css`
  (`--ld-neon`, `.neon-line`, `.neon-ring`, `.ld-bar-a/b/c`, `.ld-grad`, `body.ld-black-field`)
- My AI page: `/Users/nakfaai/Developer/projects/latindance/src/components/latin-dance/first-page.tsx`
- Events page: `/Users/nakfaai/Developer/projects/latindance/src/components/latin-dance/neon-events-view.tsx`
- Hub: `/Users/nakfaai/Developer/projects/latindance/src/components/latin-dance/hub-view.tsx`
- More sheet + tab bar: `/Users/nakfaai/Developer/projects/latindance/src/components/latin-dance/mobile-nav.tsx`
- Event page: `/Users/nakfaai/Developer/projects/latindance/src/components/latin-dance/event-detail-view.tsx`
  + `/Users/nakfaai/Developer/projects/latindance/src/components/latin-dance/event-flyer-card.tsx` (`BigFlyerCard full`)
- AI plan popup: `/Users/nakfaai/Developer/projects/latindance/src/components/latin-dance/ai-plan-sheet.tsx`
- flyer choice: `/Users/nakfaai/Developer/projects/latindance/src/lib/event-image.ts`
  (`realFlyerUrl`, `isStockPhoto`, `biggerImage`)

## 6d. Real flyer on the card (built 20261018-234712)
When an event has an organiser flyer, the poster is drawn edge to edge with only the date and time chips on it; the title, venue, styles and price sit in a block under the photo. The gradient scrim and the white text on the photo belong to the designed day poster only, so a real flyer is never covered or muddied.
The calendar holds real nights only: rows invented by the seed data are blocked, and a night without its own flyer borrows one from the same series, venue or organiser.
Near-me keeps the whole metro: events with no coordinates are kept when their city is in the metro of the chosen city.

## 9b. Signed-in pages use the same field (built 20261018-232815)
Feed, Direct Messages, Create/Edit event, Profile, Dashboard, People, Groups, Leaderboard, Pinboard, Song request, Find partner all sit inside `.ld-neon-scope` with `body.ld-black-field`: black field, `#1A1A21` cards, `#C9C4D5` muted text, no light panels.
The primary token inside the scope is `--primary:#2bc0ff` with `--primary-foreground:#06303a`, so every default button is the blue→mint pair. Pink `#ff3d7f`, amber `#f59e0b` and orange `#f97316` gradient headings are replaced by `linear-gradient(90deg,#2bc0ff,#a1ffce)` (`.ld-neon-scope .gradient-text`). Profile cover: `from-[#07202e] via-[#20143a] to-[#050409]` with a cyan/violet wash. The avatar ring wrapper keeps `rounded-full` so no square shows behind it.
