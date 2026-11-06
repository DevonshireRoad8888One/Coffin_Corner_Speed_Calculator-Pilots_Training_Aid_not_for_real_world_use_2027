# Coffin_Corner_Speed_Calculator-Pilots_Training_Aid_not_for_real_world_use_2027
Coffin_Corner_Speed_Calculator-Pilots_Training_Aid_not_for_real_world_use_2027 -  Coffin Corner — Icing &amp; Speed-Margin Calculator
# Coffin Corner — Icing & Speed-Margin Calculator

**A single-file, offline, dependency-free web tool for high-altitude speed-margin management in icing.**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-2.0-ffb020.svg)]()
[![Dependencies](https://img.shields.io/badge/dependencies-none-3ddc84.svg)]()
[![Self-test](https://img.shields.io/badge/self--test-24%2F24-3ddc84.svg)]()

> ### ⚠️ Safety notice
>
> **This is an educational study aid. It is NOT an approved document, NOT a performance tool, and NOT authorised for operational use.**
>
> * It does **not** replace the AFM/POH, the operator's manual, the FMS, or a qualified instructor.
> * It does **not** compute certified performance. Dynamic pressure is used as a *structural-load proxy*, not a certified limit.
> * The built-in Learjet 60 data set is **ILLUSTRATIVE** (derived from public commentary, not from certified documents).
> * **Never use it to brief, dispatch, or decide anything.**
>
> Built for aviation students, instructors, and safety-of-flight educators.

---

## Table of contents

- [What it is](#what-it-is)
- [Quick start](#quick-start)
- [The four tabs](#the-four-tabs)
- [The physics on one screen](#the-physics-on-one-screen)
- [Worked example: the top of the climb](#worked-example-the-top-of-the-climb)
- [The storm rule: 8 NM vs 20 NM](#the-storm-rule-8-nm-vs-20-nm)
- [The case: N722BD, 16 February 2025](#the-case-n722bd-16-february-2025)
- [Validation — the self-test](#validation--the-self-test)
- [Known limitations](#known-limitations)
- [Data provenance](#data-provenance)
- [Contributing](#contributing)
- [License](#license)

---

## What it is

At altitude, your usable speed window is not a fixed thing. It closes as you climb, it closes when you get heavy, and — critically — **ice moves the floor without moving the ceiling.** The calculator makes that explicit rather than leaving it to a mental model.

It answers four questions, live:

| Question | Where |
|---|---|
| What speed can I legally fly right now, with the margin I am supposed to keep? | **BAND** |
| How fast was the aircraft really going, and how much dynamic pressure was on it? | **DIVE** |
| What did this accident actually look like, and what remains unknown? | **CASE** |
| What are the equations, and where is the recovery doctrine? | **REF** |

**Design constraints:** one file · no build step · no dependencies · no network requests · works offline · works from `file://` on a phone, tablet, or laptop · every calculation shown with its substitution.

---

## Quick start

### Desktop
1. Download or copy `icing-calculator.html` into any folder.
2. Double-click it. It opens in your default browser.
3. Confirm the green bar reads `BUILD v2.0 | SCRIPT OK | SELF-TEST PASS 24/24`.

### Phone / tablet
1. Save the file to **Files** (iOS) or **Downloads** (Android).
2. Tap it → **Open in Safari / Chrome**.
3. On iOS: **Share → Add to Home Screen** for a full-screen, app-like experience.

### The integrity check (do this once)
| Search for | Expected |
|---|---|
| `CC-CALC-END` | found — file complete |
| `NCHK=24` | found |
| `497.9` | found (tropopause sound speed) |

**If the page is blank:** it is almost always an encoding or preview issue, not a code issue. Re-save as **UTF-8** and open the file **from disk** rather than through an in-browser preview.

---

## The four tabs

### 1 · BAND — the margin
Live speed-margin calculation with:
- the **band graphic** (amber raw band, green policy window, stall / ceiling / YOUR MACH markers)
- the **ICE COUNTDOWN table** — your stall margin at 0 / 5 / 10 / 12.5 / 15 / 20 % accretion, with the margin-zero row highlighted
- computed breakpoints for **the ice percentage at which your margin reaches zero** and **the ice percentage at which your policy window closes**
- live **stall and overspeed margins** in both Mach and KIAS
- **q** with the density × speed decomposition
- **validation**: out-of-range inputs are flagged and clamped; missing fields fall back to a documented default
- **advisories** including the icing-temperature consistency check and the automation-in-icing warning

### 2 · DIVE — ADS-B reconstruction
Enter points (time, GPS altitude, ground speed, reported vertical rate) and get TAS, flight-path angle, Mach and q for each.

It then does the thing editorial reconstructions usually get wrong: **it derives the true vertical rate from the altitude change over the timestamp interval, and compares it with the published field.** When the field saturates at its ±32,640 ft/min ceiling, the derived rate is the truth.

The impact estimate is built from the **derived** rate, not the saturated field.

### 3 · CASE — N722BD and the Columbia comparison
The 16 February 2025 Palm Beach County Learjet 60 accident as a **labelled evidence table**, the two-accident comparison, the storm-avoidance calculator, the "what should have been recognised" list, and the open investigative questions.

### 4 · REF — physics and doctrine
ISA reference atmosphere, the full equation set and assumptions, the recovery table (including *pull versus push*), and the evidence-discipline reference.

---

## The physics on one screen

ATMOSPHERE — ICAO US Standard Atmosphere 1976-80
h_m = alt_ft × 0.3048
T_ISA = 288.15 − 0.0065·h_m [K]
TROPOPAUSE at h_m = 11,000 m → T = 216.65 K, CONSTANT above
p = 101325·(1 − 0.0065·h_m/288.15)^5.25588 [Pa]
T = OAT + 273.15 (or T_ISA)
ρ = p/(R·T), R = 287.05287
σ = ρ/1.225
a = √(1.4·R·T) → ×1.68781 = kt → ×1.15078 = mph
a·√σ = a_kt·√σ [kt] ← IAS per unit Mach

STALL
Vs(W) = Vs_ref·√(W/W_ref) + config_increment [KIAS]
Vs_ice = Vs(W)·(1 + ice%/100)
Mstall = Vs_ice / (a·√σ)

CEILING (the more restrictive limit always binds)
MMO_eff = stepped Mach if the step applies, else general MMO
M_VMO = VMO(KIAS) / (a·√σ)
Ceiling = min(MMO_eff, M_VMO)

POLICY WINDOW
Floor = (Vs_ice + stall_margin) / (a·√σ)
Ceiling_pol = Ceiling − overspeed_margin / (a·√σ)
If Floor ≥ Ceiling_pol → BAND CLOSED → NO-GO at any Mach

q
V = Mach·a
q = ½·ρ·V² (1 psf = 47.8803 Pa)
q_ratio = (ρ/ρ_ref)·(V/V_ref)²


### ISA reference (sound speed in **knots**)

| Altitude (ft) | a (kt) | a (mph) | T (°C) | σ | √σ | a·√σ (kt) | IAS at M0.76 (KIAS) |
|---:|---:|---:|---:|---:|---:|---:|---:|
| 10,000 | 553.6 | 637.3 | −4.8 | 0.7385 | 0.8594 | 475.8 | 408.6 |
| 20,000 | 532.9 | 613.7 | −24.6 | 0.5328 | 0.7300 | 389.0 | 333.8 |
| 25,000 | 522.7 | 601.8 | −34.5 | 0.4482 | 0.6695 | 349.9 | 300.1 |
| 30,000 | 511.7 | 588.9 | −44.4 | 0.3741 | 0.6116 | 313.0 | 268.3 |
| 35,000 | 500.5 | 576.2 | −54.3 | 0.3100 | 0.5567 | 278.6 | 238.9 |
| **36,089** | **497.9** | 573.0 | **−56.5** | 0.2971 | 0.5451 | 271.4 | 232.8 |
| 37,000 | 497.9 | 573.0 | −56.5 | 0.2843 | 0.5332 | 265.4 | 227.6 |
| 39,000 | 497.9 | 573.0 | −56.5 | 0.2583 | 0.5082 | 253.0 | 217.0 |
| 41,000 | 497.9 | 573.0 | −56.5 | 0.2346 | 0.4844 | 241.2 | 206.8 |
| 43,000 | 497.9 | 573.0 | −56.5 | 0.2131 | 0.4616 | 229.8 | 197.0 |
| 45,000 | 497.9 | 573.0 | −56.5 | 0.1936 | 0.4400 | 219.1 | 187.9 |

> **Two things students get wrong, guarded against in code:**
> 1. **Unit slip.** At 35,000 ft, *a* = **500.5 kt**, not 576. The mph figure is the trap.
> 2. **The tropopause.** Above **36,089 ft (11,000 m)** the ISA is **isothermal**, so *a* is **497.9 kt at every level from FL370 to FL450**. It stops falling with altitude. Above FL360 the stall Mach rises from **density alone** — sound speed no longer contributes.

### Band table (Vs 185 KIAS illustrative; ceiling M0.78 above FL370, M0.81 below)

| Altitude | Mstall | Ceiling | Raw band | 10/10 policy window |
|---|---:|---:|---:|---|
| FL350 | M0.664 | M0.81 | 40.7 kt | 195.0 – 225.7 KIAS ✅ |
| FL370 | M0.697 | M0.78 | 22.0 kt | 195.0 – 197.0 KIAS ⚠️ 2.0 kt |
| FL390 | M0.731 | M0.78 | 12.3 kt | CLOSED |
| FL410 | M0.767 | M0.78 | 3.1 kt | CLOSED |
| FL430 | M0.805 | M0.78 | CLOSED | CLOSED |

**The band closes at approximately FL415** — and that altitude is different for every aircraft, weight, configuration and ice state. **The coffin corner is a margin you lose, not an altitude you pass.**

---

## Worked example: the top of the climb

N722BD, **35,250 ft pressure altitude**, OAT −47 °C, **M0.76**, **+10 % ice**, Vs 185 KIAS illustrative.

| Ice accretion | Vs ice (KIAS) | Mstall | Margin to stall | Policy window | Status |
|---:|---:|---:|---:|---:|---|
| 0 % | 185.00 | M0.668 | +25.5 kt | 16.1 kt | OK |
| 5 % | 194.25 | M0.701 | +16.3 kt | 6.4 kt | MARGIN BELOW POLICY |
| **10 %** | **203.50** | **M0.735** | **+7.0 kt** | **0.87 kt** | MARGIN BELOW POLICY |
| 12.5 % | 208.13 | M0.751 | +2.4 kt | CLOSED | MARGIN BELOW POLICY |
| **13.8 %** | **210.52** | **M0.793** | **0.0 kt** | CLOSED | **MARGIN ZERO** |
| 15 % | 212.75 | M0.768 | −2.2 kt | CLOSED | **BELOW THE STALL** |
| 20 % | 222.00 | M0.801 | −11.5 kt | CLOSED | **BELOW THE STALL** |

**The lessons this single table carries:**

- The **10/10 policy window was 0.87 kt wide** — less than a quarter of a Mach point.
- A **0.01-Mach** speed-bug notch is **2.8 kt** at this altitude — **3.2× the entire compliant window**.
- The margin reached **zero at ~13.8 % ice**; the policy window closed at **~10.5 %**. Ice was still accreting.
- **The geometric band here was 40.7 kt wide.** The aircraft was lost inside a *wide* window.

> **The margin was not a margin. It was a countdown, and nobody could see it running.**

---

## The storm rule: 8 NM vs 20 NM

| Standard distance | Applies to |
|---|---|
| **10 NM** | any thunderstorm |
| **20 NM** | severe thunderstorm (hail, damaging wind, tornado) |
| **20 NM** | IFR avoidance around cells |

### Why 8 NM is not a safe blanket rule

**1 · Reaction time is too short.** At 450 kt ground speed:

| Distance | Time available |
|---|---|
| 8 NM | **64 seconds** |
| 10 NM | 80 seconds |
| 20 NM | **160 seconds** |

You cannot clear a cell, obtain a new clearance, and descend in 64 seconds. **8 NM is not a margin — it is a reaction time too short to use.**

**2 · The hazard extends beyond the rain.** Microburst outflow and damaging straight-line wind reach 10–20 NM from the core; hail travels far downwind. **Measure from the most intense part of the cell, not the visible edge.**

**3 · Reflectivity is not the measure — and this is the N722BD lesson.** In that accident the aircraft flew through a region measured at only **25–35 dBZ at FL350**, while the cell's **55–60 dBZ core sat 10,000 ft *below* it**. A pilot looking at radar at the right flight level could have seen something that did not look like a major storm. **The upper flank of a cell — where the updraft, the shear and the icing live — can look mild where you are and be lethal.** A blanket "8 NM from the nearest echo" is not survivable.

**4 · Cells move and grow.** The buffer erodes while you are using it.

> **20 NM from the most intense part of any severe cell. 10 NM from a non-severe cell. And that is a floor, not a ceiling.**
> **In the terminal area with a 60° turn, go further.**

---

## The case: N722BD, 16 February 2025

Learjet 60, N722BD · Austin–Bergstrom → Nassau · nine on board, all killed · impact near Military Trail, Palm Beach County, Florida · the left elevator separated before impact.

> **No victims, crew, or family members are named in this repository.** Identify people only from the official NTSB record.

### Timeline, with evidence labels

| Time (EST) | Event | Label |
|---|---|---|
| 12:46 | FL350 level, near the cell; crew reports ice and ~700 ft excursions | REPORTED |
| 12:47:51 | Tracked position overlaps moderate precipitation reflectivity (25–35 dBZ at FL350); the 55–60 dBZ core was 10,000 ft below | MODELLED |
| 12:47:xx | *"Picking up precipitation. We need to climb 370."* — ATC approves | REPORTED |
| 12:48:07 | Top of climb **35,250 ft** — only +250 ft above cruise. M0.76, IAS 210.5, stall margin ~7 kt, policy window ~0.9 kt | MEASURED |
| ~12:48:28 | Loss of control. Smooth, monotonic acceleration; no step visible at 1 Hz | MODELLED |
| 12:48:58 | GPS 26,272 ft, GS 484 kt, VS −26,368 fpm. **M0.92**, q 439 psf (2.18×) | MEASURED |
| 12:49:16 | GPS 14,806 ft, GS 362 kt, VS field SATURATED at −32,640. True rate ≈ 38,200 fpm. M0.82, q 573 psf (2.85×) | MEASURED / MODELLED |
| 12:49:26 | GPS 6,506 ft, GS 251 kt. Pressure-altitude field FROZEN at 8,750 ft (NO NEW VALUE). True rate ≈ 49,800 fpm. M0.85, q 850 psf (4.23×) | MIXED |
| ~12:49:34 | Impact, near vertical | MEASURED |
| — | Left elevator recovered in three pieces, 1.3–3.8 mi from the crater — separated before impact | MEASURED |
| — | Separation altitude FL14,000–27,000 (debris-drift model) | MODELLED — *may use a mis-dated sounding* |
| — | Impact speed 500–950 km/h · 100–500+ G · 30–100 ms stopping time | DRAWN — editorial graphic, not a measurement |
| — | Cause of the separation · cause of the loss of control · whether recorders were recovered · crew names | UNKNOWN |

**Evidence labels used throughout:**

| Label | Meaning | How to use it |
|---|---|---|
| **MEASURED** | an instrument recorded it | teach with attribution |
| **REPORTED** | a human said it | teach with attribution |
| **MODELLED** | calculated from data by a stated method | teach as a hypothesis, show the method |
| **DRAWN** | an illustrator drew it | **never** teach as fact |
| **UNKNOWN** | the investigation has not answered it | teach as an open question |

**Evidence ladder — highest first:** NTSB findings → FDR/CVR → wreckage and structural examination → raw ATC audio → official weather and radar → independently documented flight tracking → **editorial reconstructions** → social-media speculation.

> **A probable cause is one sentence. The compression is in the product, not the newsroom.** A legally correct probable-cause sentence can still produce a publicly wrong understanding — which is exactly what happened with the 2008 Columbia Learjet 60 and can happen again.

### Two accidents, one failure archetype

|  | **COLUMBIA N999LJ** — 19 Sep 2008 | **PALM BEACH N722BD** — 16 Feb 2025 |
|---|---|---|
| Initiation | Tyres ~140 psi against a 219 psi specification | Ice, convective cell, FL350 |
| What the aircraft "believed" | Squat switch said "airborne" → reversers stowed → **forward thrust** | The wing could no longer carry the same load |
| What it did | Exactly what its inputs dictated | Exactly what the aerodynamics dictated |
| What the cockpit said | **Nothing.** No light, no voice, no lever movement | **Nothing.** No stall-speed indicator exists |
| The cure | Push the levers **forward** — opposite of instinct | **Descend** |
| Prior warning? | **Yes** — Troy, Alabama, 14 Jan 2001, same mechanism, two pilots seriously injured | **Maybe** — a May 2024 severe-turbulence encounter on this airframe; AMM special inspection required? |
| Public story | "The captain panicked" | "The twenty-year-old copilot" |
| Held by | Both pilots — dead | All nine — dead |

> ### The one-line why
> **The aircraft did not represent its own state.** In both accidents the pilot was **correct for the state the cockpit showed** and **catastrophic for the state the aircraft was actually in.** Neither was caused by not knowing the procedures.
>
> ### The question to carry forward
> Columbia asked: *"did anyone examine what happens when the wheel well is destroyed on the ground?"*
> Palm Beach asks: *"did anyone examine what happens to the empennage at high Mach, with ice, aft CG, at max landing weight?"*
> **Both are certification-basis questions.**

### Open questions — the investigative agenda

1. **Were the flight and cockpit recorders recovered?** One verified fact decides whether this case is open or closed.
2. **The recorded OAT versus the "clear ice" report.** Clear ice requires supercooled liquid water, which peaks near **0 to +5 °C**. At −47 °C it should be essentially absent. Either the temperature or the ice-type call is wrong — and **the two scenarios are different accidents.**
3. **The correct 16 February 2025 Miami sounding.** A published graphic is labelled "Nov. 1". The debris-drift model is the *only* thing separating "cause" from "consequence", and it may be running on the wrong day's wind.
4. **Was the performance to climb to FL370 actually available in the icing encountered?** And did a clean-wing automation model mislead the crew?
5. **Was the elevator separation the cause or a consequence?** Requires the wreckage: fracture surfaces, bend direction, right-elevator condition, ice-accretion pattern.
6. **Was the May 2024 severe-turbulence inspection done?** The one concrete institutional thread.
7. **Any AD or SB on the empennage, elevator attach fittings, hinge or trim actuator** for this serial? And the MEL status of the flight controls?

---

## Validation — the self-test

The tool runs **24 checks on every load** and prints PASS/FAIL to the header, plus a full listing under **REF → Self test**.

The suite pins the cases that caught real errors during development:

| Group | Pins |
|---|---|
| T1–T2 | FL370 +10 % ice (stall margin −1.8 kt) and FL350 clean (window 195.0–225.7 KIAS) |
| T3–T4 | tropopause sound speed (497.9 kt above FL360) · `a·√σ` in knots · reference-cruise q = 201 psf |
| T5 | **FL35,250 worked case:** margin +7.0 kt · policy window 0.87 kt · margin zero at 13.8 % ice · policy closed at 10.5 % ice |
| T6 | storm reaction times: 8 NM at 450 kt = 64.0 s · 20 NM = 160.0 s |
| T7 | dive reconstruction: point B q = 439 psf · point D q = 850 psf and M0.854 |

If a check fails, **the tool says so loudly rather than producing a confident wrong answer** — and the build stamp turns red.

---

## Known limitations

- **Not certified performance.** q is a load proxy, not a certified limit. No buffet, gust, manoeuvre-load, flutter or upset dynamics.
- **Steady flight only.** A stall, spiral or departure is out of scope.
- **Stall-speed scaling is first-order** (`Vs ∝ √W` for constant C_Lmax). Use AFM/POH data when you have it.
- **The weight-based stall model is generic.** Your aircraft's own tables are better.
- **OAT is assumed valid at the pressure altitude** entered.
- **Geometric altitude is a first-order approximation**, and is least reliable near and above the tropopause.
- **Icing is modelled as a single stall-speed penalty.** Real ice also raises drag, shifts the stall AoA, degrades ailerons and elevator power, adds weight, and can tailplane-stall at high Mach.
- **The debris-drift separation altitude in the CASE tab is inherited from a public reconstruction** and may rest on a mis-dated sounding.
- **It cannot order cause from effect.** Only the investigation can.

---

## Data provenance

**Tool physics:** ICAO US Standard Atmosphere 1976-80 · FAA **AC 61-107B** *High-Altitude Operations* · FAA **AC 91-74B** *Pilot's Handbook of Aeronautical Knowledge* · FAA / Transport Canada **In-Flight Icing Training Aid** · FAA **AC 120-111** *Upset Prevention and Recovery Training* · NTSB *Airplane Upset Management* · 14 CFR 91.137, 135.313, 61.57, 61.31 · NTSB *Airplane Accident Report* N999LJ (Columbia, SC, 2008) and the 2001 Troy, Alabama accident report · FAA airworthiness directives on tyre-pressure checks and thrust-reverser voice warning.

**Case facts:** public ATC audio; public ADS-B flight tracking; archived NEXRAD (KAMX); NTSB on-scene briefing statements; and contemporaneous broadcast and documentary coverage (Airliner Archive; ABC News). **Every case claim is labelled MEASURED / REPORTED / MODELLED / DRAWN / UNKNOWN, and all of it must be verified against the NTSB record.**

### Verify against the primary record

1. **NTSB CAROL** — search by registration **N722BD** (search by the date if you must; the date has been mis-stated in secondary sources).
2. Pull the **preliminary report**, then the **factual/circumstantial**, **aircraft performance**, **structures**, **wreckage**, **maintenance** and **human factors** reports.
3. Read the final **probable cause** — and read **beyond the first sentence**.

---

## Contributing

Contributions are welcome, particularly:

- **Certified data sets.** Replacing the illustrative Learjet 60 preset with figures taken from a public AFM, for an airframe where the manufacturer permits publication.
- **Additional aircraft presets**, with the same PUBLISHED / ILLUSTRATIVE labelling discipline.
- **Verification of case facts** against NTSB CAROL, with the source and access date recorded.
- **A "what changed" note** in every pull request that alters a number.

**Please do not add:** victim or crew names; unlabelled case claims; performance figures presented as certified; or any suggestion that a person caused the accident without an official finding.

Bugs and corrections are the most valuable contributions. If you change a physical constant, update the matching self-test check in the same commit — **the test suite is the specification.**

---

## License

MIT — see [LICENSE](LICENSE).

The code is MIT licensed. The aviation content is provided for education and safety-of-flight study; it is not approved for operational use, and the maintainers accept no liability for any decision made using it.

---

## Acknowledgements

- The **NTSB** for the Columbia Learjet 60 report and the 2001 Troy accident report — the clearest available demonstration of how a system failure can hide behind a one-sentence probable cause.
- The **crew, investigators and public contributors** whose work on the Palm Beach County accident made the flight-path reconstruction possible.
- Broadcast and documentary teams who chose caution over speculation — and who declined to air distress audio without a public-interest reason.
- Every instructor who teaches that **a probable cause is a summary, not the report.**

- 📁 REPOSITORY STRUCTURE

Collapse
Save
Copy
1
2
3
4
5
6
7
8
icing-calculator/
├── icing-calculator.html      the entire application — one file, zero dependencies
├── README.md                  what it is, how to run it, the physics, the case
├── CONTRIBUTING.md            workflow, evidence-label rules, PR checklist, safety issues
├── LICENSE                    MIT + explicit carve-out on the aviation content
└── docs/
    └── case-notes.md          the working case file: timeline, derivations,
                                evidence labels, open questions, correction log
1️⃣ LICENSE
markdown

Collapse
Save
Copy
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
⌄
⌄
⌄
⌄
⌄
MIT License

Copyright (c) 2025 Deven

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

---

ADDITIONAL NOTE ON THE AVIATION CONTENT

The SOFTWARE licence above covers the source code of icing-calculator.html.

The aviation reference and case-study content in this repository is provided
for education and safety-of-flight study only. It is NOT an approved document,
NOT certified performance data, and NOT authorised for operational use. It does
not replace the AFM/POH, the operator's manual, an approved performance tool,
or the instruction of a qualified instructor.

The default aircraft data set is ILLUSTRATIVE and must be replaced with
certified figures from the applicable flight manual before use.

The maintainers accept no liability for any decision made using this material.
📌 Change Copyright (c) 2025 Deven to whatever name or organisation you want on it, and adjust the year if you're publishing later. That's the only edit needed.

2️⃣ CONTRIBUTING.md
markdown

Collapse
Save
Copy
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
⌄
⌄
⌄
⌄
⌄
⌄
⌄
⌄
⌄
# Contributing

Thanks for helping. This repository is a **teaching tool** about high-altitude speed margins in icing, built around a fatal-accident investigation and a second accident used to teach systems thinking.

The bar for accuracy here is high, and the tolerance for speculation is zero. Most of the value you can add is **making the numbers and the evidence labels more accurate** — not adding features.

---

## Two-minute tour

| Path | What it is |
|---|---|
| `icing-calculator.html` | The entire application. One file, no dependencies, no build step. **All changes to the tool happen here.** |
| `README.md` | What the tool is, how to run it, the physics, and the case summary. |
| `docs/case-notes.md` | The working case file: timeline, derivations, evidence labels, open questions, and a correction log. |
| `LICENSE` | MIT for the code, with an explicit carve-out on the aviation content. |
| `CONTRIBUTING.md` | This file. |

**Read `docs/case-notes.md` before changing anything.** Almost every number in the tool traces to a labelled entry in that file.

---

## The one rule that matters most

> ### Every factual claim carries an evidence label.
>
> | Label | Meaning | How it may be used |
> |---|---|---|
> | **MEASURED** | an instrument or recovered wreckage recorded it | teach with attribution |
> | **REPORTED** | a human said it | teach with attribution |
> | **MODELLED** | calculated from data by a stated method | teach as a hypothesis, show the method |
> | **DRAWN** | an illustrator or graphic produced it | **never** teach as fact |
> | **UNKNOWN** | the investigation has not answered it | teach as an open question |

If you cannot label a claim, either label it **UNKNOWN** or remove it. **Do not blur the labels to make a point land harder.** The entire pedagogical value of this tool is that it demonstrates what is known, what is inferred, and what is merely drawn.

---

## Ground rules

1. **Never name victims, crew members, or family members.** Identify people only from the official NTSB record, and only where the NTSB has released them. This repository exists because a nine-person family should not become a case study's headline.
2. **Do not attribute cause** to any individual or organisation without an official finding. You may explain *mechanisms*, *how something could happen*, and *what evidence would distinguish two hypotheses*. You may not state that a person "caused" an accident.
3. **Do not present modelled values as measurements.** If you compute a G-force, an impact speed, a drift distance or a separation altitude, label it MODELLED or DRAWN and show the method.
4. **Do not edit case facts without a source.** Every case fact needs the source, the access date, and the verification status.
5. **Do not change a physical constant without changing its self-test.** The test suite is the specification — if the specification moves, the tests move with it in the same commit.
6. **No sensational framing.** No "the 30-second window of death." Use numbers and mechanisms; let the reader draw the conclusion.
7. **Keep the disclaimers.** The safety notice at the top of the README, in the app banner, and in the licence carve-out are load-bearing.

---

## Setting up

There is no build. You need a text editor and a browser.

```bash
# clone, then simply open the file
git clone <your-fork-url>
cd icing-calculator
open icing-calculator.html          # macOS
xdg-open icing-calculator.html      # Linux
start icing-calculator.html        # Windows
Use a plain-text editor. Word processors may convert straight quotes to smart quotes and silently break the JavaScript.

Save as UTF-8. The file contains degree signs and multiplication symbols in visible strings.

Testing — the self-test is the specification
The tool runs 24 checks on every page load and reports PASS/FAIL in the header. Open icing-calculator.html and read the header, then expand REF → Self test for the full listing.


Collapse
Save
Copy
1
BUILD v2.0 | SCRIPT OK | SELF-TEST PASS 24/24
Your change is not finished until the self-test passes. If it fails, the tool is telling you something true. Find out what.

Manual checks to run before every pull request
#
DO
EXPECT
1
Load the page
green stamp, SELF-TEST PASS 24/24
2
BAND → N722BD TOP OF CLIMB – FL35250 +10% M0.76
margin +7.0 kt, window 0.87 kt, margin zero at 13.8 %, policy closed at 10.5 %
3
BAND → FL370 – +10 % ice – M0.76
NO-GO – BELOW STALL
4
BAND → FL350 – clean – M0.76
GO, window 195.0–225.7 KIAS
5
ICE COUNTDOWN table
the row between 10 % and 15 % shows the margin crossing zero
6
DIVE → Load N722BD points
derived rates 38,200 and 49,800 fpm; FIELD SATURATED flagged; impact ≈ 7.8 s
7
CASE → storm calculator at 8 NM / 450 kt
64 seconds, verdict "LESS THAN THE 10 NM MINIMUM"
8
REF → ISA table
500.5 kt at FL350; 497.9 kt at every level FL370–FL450
9
Narrow the window to phone width
layout stacks, no horizontal scroll except inside tables
10
Copy brief → paste
text block contains the verdict and both margins


Making a change
Changing a physical constant
Physical constants live in one place at the top of the script:

js

Collapse
Save
Copy
1
2
var R=287.05287, GA=1.4, RHO0=1.225, KT=1.687809857, MPH=1.150779448, PSF=47.8803;
var T0=288.15, P0=101325, LC=0.0065, G=9.80665;
If you change one, update the matching check in selfTest() in the same commit, and add a note to the correction log in docs/case-notes.md.

Adding an aircraft preset
Presets are declared in one object:

js

Collapse
Save
Copy
1
2
3
4
5
⌄
⌄
var PRE={
  lr60:{vsRef:185,wRef:27500,vmo:340,mmo:0.81,stepAlt:37000,stepMach:0.78,
        t:'Learjet 60 TEACHING CASE - ILLUSTRATIVE (commentary-chart values, NOT certified data). ...'},
  custom:{t:'Custom - enter ... from your AFM.'}
};
To add a type:

Add the entry to PRE.
Add an <option> to #preset in the HTML.
Label it. If the figures are from public data rather than a flight manual, say so in the t: string and in the option text.
Add a preset button in the Presets card if it is worth one-click access.
Add a self-test check that pins one or two of its derived values.
Never add a preset whose stall speed, VMO or MMO you cannot source. A placeholder is fine; a guess is not.

Adding case facts
Add the fact to docs/case-notes.md with a label and a source.
If it appears in the app, mirror it in the CASE tab and keep the labels.
Update the Verification status table at the top of the case notes.
Never add a case fact that has not been checked against the NTSB record unless it is explicitly labelled MODELLED or UNKNOWN.

Working on the dive reconstruction
The DIVE tab computes derived vertical rates from the timestamps, not from the reported field. If you add points, add times in HH:MM:SS or the derived-rate table will show --.

Pull-request checklist

The self-test passes 24/24 (or, if you added checks, all of them)

No change to a physical constant without a matching test update

No new factual claim without an evidence label

No victim, crew or family names

No change to the disclaimers

docs/case-notes.md updated if any number or fact moved

Correction log updated if anything was wrong and has been fixed

Manual checks 1–10 above pass

Tested on a phone viewport if you touched layout

Commit message explains what changed and why the physics says so
What we will not merge
Case facts without a source, a label, and a verification status
Victim or crew names
Any statement that a person caused an accident, absent an official finding
Modelled values presented as measurements
Certified performance figures that cannot be publicly sourced
Removed disclaimers
A change to a physical constant with no test update and no correction-log entry
Edits that make the tool prettier but less honest
Reporting a safety issue
If you find that the tool outputs a number that could lead someone to a wrong operational decision, report it as a safety issue, not a bug — even though the tool is educational, people will use it.

Include: what you entered, what it produced, what you believe is correct, and how you verified it.

Label it [SAFETY] in the issue title and do not open a public pull request for it until a maintainer has responded.

Severity guide

LEVEL
MEANING
EXAMPLE
S1 — safety
A wrong number could be used operationally
A unit error; a sign error in a margin; a ceiling computed from the wrong limit
S2 — correctness
A number is wrong but cannot mislead a user who understands the tool
A wrong tick in a table; a stale label
S3 — usability
Confusing, broken, or missing
Layout, labels, responsiveness


Respectful conduct
This repository concerns nine people who died, and two pilots from 2008 who could not answer questions about their own accident. Keep the discussion technical and about systems.

Never characterise a crew member's age, gender, race, nationality, voice, manner, or hours as a causal factor. If a demographic characteristic is relevant to an operational requirement, cite the requirement and the evidence. Otherwise leave it out.

If discussion drifts toward blaming an individual, redirect to the mechanism, the evidence, and the question of what the design, the maintenance programme, the training, the certification basis and the regulator each contributed.

Assume good faith. People share this case for different reasons and with different levels of training. Correct the facts, not the person.

Licence
By contributing you agree that your contributions are licensed under the MIT Licence (see LICENSE), including the additional note that the aviation content is educational only and not authorised for operational use.


Collapse

Run
Save
Copy
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63
64
65
66
67
68
69
70
71
72
73
74
75
76
77
78
79
80
81
82
83
84
85
86
87
88
89
90
91
92
93
94
95
96
97
98

---

## 2. Verification status

| Item | Status | How to close it |
|---|---|---|
| **Accident date** | **16 February 2025** — high confidence. Secondary sources state "Nov 1 2026"; that is an error and it appears to have propagated from the sounding graphic (below) | Confirm on NTSB CAROL by registration |
| **Time of impact** | ≈ 12:49:34 EST (17:49:34 UTC) — MODELLED from ADS-B altitude deltas | FDR |
| **Aircraft** | Learjet 60, registration N722BD, 2004 model year — high confidence | CAROL |
| **Serial number** | ≈ 271 (from the registered-owner entity name) — verify | CAROL |
| **Operator** | Part 135 charter operator, Fort Lauderdale FL — high confidence | CAROL |
| **Registered owner** | Delaware limited-liability entity — high confidence | FAA registry |
| **Route** | Austin–Bergstrom → Nassau — high confidence | CAROL |
| **Occupants** | 9 (2 flight crew, 7 passengers) — high confidence | CAROL |
| **Crew names** | **NOT released by the NTSB.** Secondary sources disagree on spelling and add names that no official source confirms | NTSB |
| **Impact point** | Military Trail area, Palm Beach County FL — high confidence | CAROL / wreckage |
| **Elevator separated before impact** | Three pieces recovered 1.3–3.8 mi from the crater — MEASURED (NTSB on-scene briefing) | Structures Report |
| **Separation altitude** | FL14,000–27,000 — MODELLED from debris drift. **May rest on a mis-dated sounding** | Re-run with the correct sounding |
| **Cause of separation** | **UNKNOWN** | Structures Report |
| **Cause of the loss of control** | **UNKNOWN** | Performance Report / FDR / CVR |
| **Recorders recovered** | **CONTRADICTED.** One broadcast states the black boxes were recovered; earlier reporting says the CVR had not been | NTSB statement |
| **Flight data / cockpit audio** | Exist as a product; availability unknown | NTSB |
| **Reported OAT −47 °C** | REPORTED, and **physically inconsistent with "clear ice"** (see §5) | FDR OAT + sounding |
| **Ice type** | "Light clear ice" — REPORTED. Inconsistent with the reported temperature | Wreckage ice pattern |

**Source note:** public ATC audio, public ADS-B tracking, archived NEXRAD, an NTSB on-scene briefing, and contemporaneous broadcast and documentary coverage were used to reconstruct the flight path. None of these substitutes for the NTSB record.

---

## 3. Master timeline

| Time (EST) | Event | Label | Source |
|---|---|---|---|
| 12:46 | FL350 level, near the cell. Crew reports light clear ice, "manageable", and moderate turbulence with ~700 ft altitude excursions | REPORTED | Cockpit radio |
| ~12:47 | Crew requests a climb to 37,000 ("Picking up precipitation. We need to climb 370"). ATC approves | REPORTED | Cockpit radio |
| 12:47:51 | Tracked position overlaps a region of moderate precipitation reflectivity (25–35 dBZ at FL350). The cell's 55–60 dBZ core was ~10,000 ft **below** the aircraft. Cell top ~45,000 ft | MODELLED | ADS-B + KAMX (beam geometry §6) |
| 12:48:07 | **Top of climb: 35,250 ft** — only +250 ft above cruise. M0.76, IAS 210.5 KIAS. Stall margin ≈ +7 kt; 10/10 policy window ≈ 0.9 kt | MEASURED | ADS-B |
| ~12:48:28 | Loss of control. Vertical rate accelerates smoothly and monotonically; no step change visible at 1 Hz | MODELLED | ADS-B altitude deltas |
| 12:48:58 | GPS 26,272 ft · GS 484 kt · VS −26,368 fpm. **M0.92** (peak). q 439 psf (2.18× cruise) | MEASURED | ADS-B |
| 12:49:16 | GPS 14,806 ft · GS 362 kt · VS field **SATURATED at −32,640**. True rate ≈ 38,200 fpm. M0.82. q 573 psf (2.85×) | MEASURED / MODELLED | ADS-B (PA and GPS agree) |
| 12:49:26 | GPS 6,506 ft · GS 251 kt. Pressure-altitude field **FROZEN** at 8,750 ft ("NO NEW VALUE"). True rate ≈ 49,800 fpm. M0.85. q 850 psf (4.23×) | MIXED | ADS-B |
| ~12:49:34 | Impact, near vertical, Military Trail area | MEASURED | Wreckage + ADS-B |
| — | Left elevator recovered in three pieces, 1.3–3.8 mi from the crater | MEASURED | NTSB on-scene briefing |
| — | Impact speed 500–950 km/h · 100–500+ G · 30–100 ms stopping time · illustrated 60° final attitude | **DRAWN** | Editorial graphic |

**Additional reported transmissions:** an indication of severe turbulence ("We're in an overspeed condition, descending 20,000 feet per minute"), a reported 60° bank, and the final "we're going to crash into the ground" followed by "we're out of time." These are **REPORTED**; exact wording must be taken from the raw recording.

---

## 4. The physics, derived

### 4.1 Atmosphere at 37,000 ft (ISA)

The ISA is **isothermal above 11,000 m (36,089 ft)**, so the sound speed stops falling with altitude:

h_m = 37,000 × 0.3048 = 11,277.6 m (> 11,000 m)
T_ISA = 216.65 K (−56.5 °C) constant above the tropopause
p = 22,632.1 · e^(−g(h−11,000)/(R·216.65)) = 21,663 Pa
σ = 0.2843 √σ = 0.5332
a = √(1.4·287.053·216.65) = 295.1 m/s = 497.9 kt
a·√σ = 265.4 kt ← IAS per unit Mach


Collapse

Run
Save
Copy
1
2
3
4
5
6

> **Unit check:** at 35,000 ft, *a* = **500.5 kt = 576.2 mph**. The mph figure is the trap.
> **Consequence:** above FL360 the stall Mach rises from **density alone** — sound speed no longer contributes.

### 4.2 The band at the top of the climb (35,250 ft, OAT −47 °C, Vs 185 KIAS)

T = 226.15 K ρ = 0.36317 kg/m³ σ = 0.29646 √σ = 0.54448
a = 301.46 m/s = 508.8 kt a·√σ = 277.0 kt


Collapse

Run
Save
Copy
157
158
159
160
161
162
163
164
165
166
167
168
169
170
171
172
173
174
175
176
177
178
179
180
181
182
183
184
185
186
187
188
189
190
191
192
193
194
195
196
197
198
199
200
201
202
203
204
205
206
207


---

## 11. Correction log

Every entry here is a mistake that was found, corrected, and **pinned with a self-test check** so it cannot recur silently.

| # | Error | Correction |
|---|---|---|
| 1 | Speed of sound used in **mph mislabelled as knots** (e.g. 576.5 kt at FL350) | *a* at FL350 = **500.5 kt**; at FL370 = **497.9 kt** |
| 2 | `a·√σ` built from `aMs` (m/s) but used in knots contexts — every derived Mach was 1.688× too large | Built from `aKt`: **278.6 kt** at FL350, **265.4 kt** at FL370 |
| 3 | Reference-cruise constant double-converted (kt and m/s both applied), inflating every q ratio by 33 % | `REF.q` = **9,641 Pa = 201.3 psf** |
| 4 | `id="mStall"` used twice — the margin-policy input and the band-graphic marker. `getElementById` returned the input, so the marker never moved | Marker renamed `mStallMk` |
| 5 | `id="qTbl"` placed on the `<tbody>` instead of the `<table>`, so the fill silently wrote nothing | `<tbody>`-aware `fill()` helper added |
| 6 | **Tropopause ignored** — the tropospheric lapse rate was extended above 11,000 m | Isothermal layer implemented; `a` constant at 497.9 kt above FL360 |
| 7 | q-peak over-corrected to 585 psf (2.9×) using the field explicitly flagged **NO NEW VALUE** | Corrected to **850 psf (4.23×)** from the live GPS altitude. *A field marked stale must never be used to correct the data.* |
| 8 | Impact time estimated at 12:49:48–50 using the understated descent rate | **≈ 12:49:34**, from the derived rate and the requirement that the last transmission be within ~10–15 s of impact |
| 9 | A secondary summary converted 350 ft/s to 1,200 km/h | 350 ft/s = **384 km/h** — and it is the *vertical* component, matching the 20,000–22,000 fpm radio call. The apparent discrepancy was a vertical-versus-total category mix-up |
| 10 | Accident date stated as "Nov 1 2026" in downstream graphics and summaries | **16 February 2025**. The error traces to the sounding graphic labelled "Nov. 1", which is also the input to the debris-drift model |
| 11 | An infographic labelled "rapid descent begins" at 31,350 ft | The descent began at the top of climb (~35,250 ft). Rates: 7,300 → 20,950 → 37,300 → 49,800 fpm, smooth and monotonic |
| 12 | The same infographic's stated "peak 20,000–22,000 fpm" is its own **average**; the segment average reaches 30,800+ fpm and the field saturates at 32,640 | Stated peak is the radio call, not the rate. True rate ≈ **49,800 fpm** in the final segment |
| 13 | 100–500+ G presented as an estimate | It is a **modelled rigid-body stop**, and after a breakup there is no single G — only a fragment velocity field. Labelled DRAWN in the tool |
| 14 | Radar "core entry" asserted from a 25–35 dBZ value at 100 NM range | **MODELLED**, with the ~3.3° beam and ~10,000 ft vertical resolution stated |
| 15 | The "the computer is going to be wrong" point absent | Added: the clean-wing performance assumption in the FMS and autothrust is invalid in ice, and a cleared level may be physically unreachable |
| 16 | "Pull versus push" absent | Added, with the 7–10 second jet stall-recognition window and the AF447 parallel |

---

## 12. Sources and how to verify them

| Source | Supports | Verify at |
|---|---|---|
| **NTSB CAROL** — accident record for N722BD | date, aircraft, serial, crew (if released), report links | <https://data.ntsb.gov/carol-main-public/basic-search> |
| **NTSB AAR** — N999LJ, Columbia SC, 19 Sep 2008 | probable cause, tyre findings, reverser and squat-switch findings, safety recommendations | ntsb.gov |
| **NTSB report** — Troy, Alabama, 14 Jan 2001 | the prior occurrence of the same mechanism | ntsb.gov |
| **FAA ADs** — tyre-pressure checks (2010); reverser voice warning (2020) | the regulatory timeline | faa.gov |
| **14 CFR** — 91.137, 135.313, 61.57, 61.31 | icing and PIC duties; currency | <https://www.ecfr.gov/current/title-14> |
| **FAA ACs** — 61-107B; 91-74B; 120-111; In-Flight Icing Training Aid | high-altitude operations; stall-speed effects of ice; upset recovery | faa.gov |
| **ICAO US Standard Atmosphere 1976-80** | all atmosphere values | NOAA/NASA/USAF document |
| **Public ADS-B tracking** | the flight path and altitudes used in §3–§4 | flight-tracking archives; ADSB.LOL and similar |
| **Archived NEXRAD (KAMX)** | cell structure and reflectivity | NOAA archive / Iowa Environmental Mesonet |
| **Public ATC audio** | the transmissions quoted in §3 | the recording itself — the only controlling source for wording |
| **Broadcast and documentary coverage** | chronology and public reaction | named programmes; **all case claims above must be confirmed against the NTSB record** |

---

## 13. Change control for this file

- **Any number that changes gets a new row in §11.** A correction with no log entry is a defect.
- **Any new fact gets a label and a source.** No exceptions.
- **The tool is regenerated from this file**, not the other way round.
4️⃣ README.md
Unchanged from my previous message — copy it straight across. For convenience, its opening and structure:

markdown

Collapse
Save
Copy
1
2
3
4
5
6
7
8
9
10
11
12
⌄
⌄
⌄
# Coffin Corner — Icing & Speed-Margin Calculator
...
> ### ⚠️ Safety notice
> **This is an educational study aid. It is NOT an approved document...**
...
| Path | What it is |
|---|---|
| `icing-calculator.html` | the entire application |
| `README.md` | what it is, how to run it, the physics, the case |
| `docs/case-notes.md` | the working case file |
| `LICENSE` | MIT + carve-out |
| `CONTRIBUTING.md` | workflow and rules |
✅ PUSH IT
bash

Collapse
Save
Copy
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
mkdir icing-calculator && cd icing-calculator
# place icing-calculator.html, README.md, CONTRIBUTING.md, LICENSE
mkdir -p docs && place docs/case-notes.md
git init
git add .
git commit -m "Coffin Corner: icing & speed-margin calculator v2.0

Single-file offline calculator: band margin with ice countdown, ADS-B dive
reconstruction with derived vertical rates, N722BD case material with evidence
labels, storm-avoidance arithmetic. 24-check self-test.

Case facts are labelled MEASURED / REPORTED / MODELLED / DRAWN / UNKNOWN and
must be verified against the NTSB CAROL record."
git branch -M main
git remote add origin <your-repo-url>
git push -u origin main
POST‑SUMMARY
1. Four files, one coherent policy. The LICENSE (MIT for code + an explicit carve‑out that the aviation content is educational and not authorised for operational use), CONTRIBUTING.md, README.md, and docs/case-notes.md all say the same three things: label every claim, never name the people, and the test suite is the specification.

2. docs/case-notes.md is the intellectual core. It holds the timeline with evidence labels, every derivation, the temperature/ice‑type contradiction, the radar geometry, the two‑accident comparison, the nine‑question investigative agenda, a sixteen‑row correction log, and the source table with verification instructions. The tool is explicitly documented as being generated from that file — not the other way round.

3. The correction log is the part that makes this credible. Sixteen mistakes, each with what was wrong, what it is now, and the self‑test check that pins it. A reader can audit the whole analysis and see exactly where it has been wrong. That is what separates a teaching tool from a graphic.

4. CONTRIBUTING.md makes the right refusals explicit — no victim or crew names, no cause without an official finding, no modelled value presented as a measurement, no constant changed without a test update — and adds a [SAFETY] issue class for anyone who finds a number that could mislead operationally. Given the tool's subject, people will use it operationally, and the file says so.

5. Two edits to make before you push: the copyright holder/year in LICENSE, and — if you can spare five minutes — verify the date and crew‑name release status on CAROL, because README.md and docs/case-notes.md both assert 16 February 2025 and mark the crew names NOT RELEASED. If either has changed, update §2 of the case notes and the CASE tab together.

6. And the sentence the repository exists to make: the corner is a margin, not an altitude — ice moves the floor — and in this accident the margin reached zero at roughly 13.8 % ice accretion, faster than anyone could see it happening.



### Core relationships

