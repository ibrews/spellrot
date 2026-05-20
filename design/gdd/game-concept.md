# Game Concept: Spellrot

*Created: 2026-05-20*
*Status: Draft*

---

## Elevator Pitch

> A fluid arena brawler where you're the last living wizard holding back an endless zombie horde. Every hit from a wizard zombie corrupts your spells — making them wilder and more powerful, but pushing you closer to becoming the thing you're fighting.

---

## Core Identity

| Aspect | Detail |
| ---- | ---- |
| **Genre** | Action Arena Brawler / Horde Survival |
| **Platform** | PC (Steam / Epic) |
| **Target Audience** | Hardcore/Mid-core action players, 18–35, Hades fans |
| **Player Count** | Single-player |
| **Session Length** | 10–30 minute runs |
| **Monetization** | Premium (TBD — prototype stage) |
| **Estimated Scope** | Large (12–18 months, solo — prototype in weeks) |
| **Comparable Titles** | Hades, Vampire Survivors, Halls of Torment |

---

## Core Fantasy

You are the outnumbered mage who holds the line through skill and will — channeling increasingly corrupted magic to survive ever-larger waves of the undead, dancing on the knife's edge between power and transformation.

The fantasy is not just "powerful wizard." It's "wizard who is barely winning, and the winning itself is changing them." The corruption spectrum creates a running question every player asks without being told to: *how far can I push this before I lose myself?*

---

## Unique Hook

Like Hades' arena combat, AND ALSO getting hit by wizard zombies doesn't just hurt you — it warps your spells into corrupted, more powerful variants. Managing your corruption state (read entirely through particle color shifts and audio cues, no UI bar) is as central as managing your position.

---

## Visual Identity Anchor

**Direction: Necrotic Arcana**

*One-line visual rule:* Magic should glow like it's alive; corrupted magic should look like something is trying to escape it.

**Supporting visual principles:**

1. **Spells are the foreground.** Enemy models and arena geometry stay dark and desaturated so spell effects always read clearly — even from across the screen during heavy spawns.
   *Design test:* "If a spell effect is hard to read during a busy wave, darken the surrounding elements before you simplify the effect."

2. **Corruption shift is screen-readable.** The color change between clean and infected states must be dramatic enough to convey player state without a bar. When in doubt, push the color further.
   *Design test:* "If a playtester couldn't tell what corruption state the wizard was in without looking at a UI element, the effect needs more contrast."

3. **Wizard zombies echo their spells.** Corrupted enemy magic colors telegraph attacks — players learn to read incoming threat type from color before they can identify enemy type.
   *Design test:* "If a wizard zombie attack feels surprising (not in the good way), check whether its projectile color was distinct enough from the background."

**Color philosophy:**
- **Clean wizard:** Cool blues, bright whites, hot gold highlights — arcane and precise
- **Infected:** Shifting toward sickly chartreuse-green and deep necrotic purple — warmth draining out
- **Heavily infected:** Near-inverse palette — void-black with bone-white and toxic green — the wizard's colors nearly reversed
- **Arena:** Muted stone and dark earth tones throughout — never competing with spell effects

---

## Player Experience Analysis (MDA Framework)

### Target Aesthetics (What the player FEELS)

| Aesthetic | Priority | How We Deliver It |
| ---- | ---- | ---- |
| **Sensation** | 2 | Niagara spell effects, corruption particle shifts, adaptive audio on state change |
| **Fantasy** | 3 | "Last wizard standing" identity; the corruption arc is a character arc told in color |
| **Narrative** | 6 | Minimal in prototype; context provided by visual design and enemy design |
| **Challenge** | 1 | Wave escalation, wizard zombie threat prioritization, corruption risk management |
| **Fellowship** | N/A | Single-player |
| **Discovery** | 4 | Spell mutation behavior at higher corruption; edge-case corruption interactions |
| **Expression** | 5 | Spell selection and rotation, arena positioning choices, corruption risk tolerance |
| **Submission** | N/A | Not a relaxation game |

### Key Dynamics (Emergent player behaviors)

1. Players will naturally experiment with letting corruption build higher to discover spell mutations — the "what happens if I let it get worse?" pull
2. Players develop mental maps of arena cleanse zones and factor them into combat routing, creating a second layer of positioning strategy
3. Players learn to prioritize wizard zombie kills over regular zombies — they're both corruption sources and tactical threats
4. Players develop "corruption-aware" positioning — approaching fights at angles that minimize wizard zombie contact while still generating purge from regular zombie kills

### Core Mechanics

1. **Fluid spell casting** — 3 spell types (e.g., Fireball, Force Push, Lightning) on a basic-attack + cooldown rhythm, like Hades
2. **Corruption spectrum** — 3 states (Clean → Infected → Heavily Infected) communicated through particle color and audio only; no UI bar
3. **Spell mutation system** — each spell has a clean and a corrupted behavioral variant; corruption state determines which fires
4. **Dual purge mechanics** — kill regular zombies to slowly cleanse (rewards aggression); stand in arena cleanse zones for faster reset (rewards positioning)
5. **Wave escalation** — endless waves with increasing zombie count and wizard zombie ratio per wave

---

## Player Motivation Profile

### Primary Psychological Needs Served

| Need | How This Game Satisfies It | Strength |
| ---- | ---- | ---- |
| **Autonomy** | Positioning choices, corruption risk tolerance, spell rotation timing | Core |
| **Competence** | Hades-style legible deaths, visible skill growth through run experience, clear cause-and-effect | Core |
| **Relatedness** | Minimal in prototype; long-term: leaderboard competition, score sharing | Minimal |

### Player Type Appeal (Bartle Taxonomy)

- [x] **Achievers** — Wave count score, personal best, survival time leaderboard
- [x] **Explorers** — Spell mutation discovery, pushing corruption edge cases, learning arena geometry
- [ ] **Socializers** — Not served in prototype
- [x] **Killers/Competitors** — Leaderboard potential, personal best competition

### Flow State Design

- **Onboarding curve:** First 2 waves teach basic spell casting against regular zombies. Wizard zombies appear in wave 3, introducing corruption naturally — the first hit is the tutorial.
- **Difficulty scaling:** Each wave increases spawn volume and wizard zombie ratio; later waves introduce formation spawning that creates cleanse zone pressure
- **Feedback clarity:** Death screen shows wave survived and kill count; corruption state transitions are legible progress indicators in real time
- **Recovery from failure:** Immediate restart, no loading screen required, typical run under 15 minutes — failure is educational by design

---

## Core Loop

### Moment-to-Moment (30 seconds)

Channel spells → kill zombies → manage arena position → absorb or dodge wizard zombie hits → observe corruption state shift (particle color, audio) → purge via kills or cleanse zone routing → repeat at escalating intensity.

The intrinsic satisfaction: always something to press, always in motion, always reading the game world for state information. Spell mutations are the spectacle reward for staying alive long enough to get infected.

### Short-Term (5–15 minutes)

Each wave is a breath cycle — escalating tension as zombies close in, brief exhale as the wave clears. The "one more wave" hook lives in the player's belief that they understand what went wrong and can fix it next run. The discovery pull: *I wonder what my spells do if I let corruption get really high.*

### Session-Level (10–30 minutes)

One run. Multiple wave surges of escalating difficulty. Run ends on death. Score displayed (waves survived + kills). Immediate rematch available. The session ends with the player thinking about the specific mistake they made and the specific thing they want to try differently.

### Long-Term Progression

**Prototype:** Personal best score is the only meta-goal. Replayability comes from curiosity about corruption depth and skill mastery.

**Full game:** Spell unlock progression, new enemy types with distinct corruption behaviors, named wizard zombie boss encounters, multiple arenas with different cleanse zone configurations.

### Retention Hooks

- **Curiosity:** "What do my spells do at full corruption?" / "Can I survive wave 20?"
- **Investment:** Personal best wave count; the feeling that the next run will be the run
- **Social:** Leaderboard potential — score sharing is natural ("I hit wave 14")
- **Mastery:** Learning wizard zombie attack patterns, optimal purge routing, corruption threshold management

---

## Game Pillars

### Pillar 1: Aggression Solves Problems

Killing enemies is the answer to every problem, including your own corruption. Retreating only delays the threat.

*Design test:* "If a mechanic rewards passivity or turtling over fighting, it needs to go."

### Pillar 2: Read the World, Not the HUD

Player state — infection level, spell power, danger — is communicated through particle effects, spell color, and audio, not numbers or bars.

*Design test:* "Before adding a UI element, ask if the game world can tell the player the same thing."

### Pillar 3: Every Position is a Decision

The arena has structure. Cleanse zones, spawn points, and spell firing lines create meaningful spatial choices every few seconds.

*Design test:* "If a player can stand in one spot for a full wave, the encounter design needs more spatial pressure."

### Pillar 4: Corruption is Drama

The infection spectrum should be visually and aurally spectacular at every stage. A half-corrupted wizard should look dangerous, not just mechanically different.

*Design test:* "If a corruption effect feels 'fine,' make it more extreme. Players should feel the change."

### Pillar 5: Death is Legible

Players should always understand why they died. Cheap deaths — invisible threats, unclear telegraphing — erode trust and break the mastery loop.

*Design test:* "If a player could honestly say 'I didn't see that coming,' the encounter design failed."

### Anti-Pillars (What This Game Is NOT)

- **NOT passive healing:** Regeneration that bypasses kill-cleanse and cleanse zones undermines the tension/purge design.
- **NOT inventory or loot systems:** Item management pulls focus from fluid combat rhythm. That's a different game.
- **NOT escort or protection objectives:** The "last wizard standing" fantasy requires pure survival stakes.
- **NOT narrative exposition in the prototype:** Validate the feel first. Story can come later.

---

## Inspiration and References

| Reference | What We Take From It | What We Do Differently | Why It Matters |
| ---- | ---- | ---- | ---- |
| **Hades** | Fluid combat rhythm, run-based structure, death as a learning tool | Single-level infinite wave instead of room-clearing roguelite; corruption adds active risk/reward | Proves this feel works and has a large, hungry audience |
| **Vampire Survivors** | Horde wave escalation, survival arc, "one more run" psychology | Active aimed spell casting vs passive; corruption creates reactive decision-making | Validates that wave survival is compelling even without narrative payoff |
| **Metal Gear Solid V** | Emergent tactical decision-making, sandbox spatial awareness | Same emergent quality applied to arena combat rather than open-world stealth | Confirms players enjoy spatial reasoning as a core skill layer |

**Non-game inspirations:** The visual language of body horror (corruption as visible transformation); the sound design aesthetic of supernatural dread; the "last stand" archetype from fantasy literature.

---

## Target Player Profile

| Attribute | Detail |
| ---- | ---- |
| **Age range** | 18–35 |
| **Gaming experience** | Mid-core to Hardcore |
| **Time availability** | 20–45 min sessions, plays on weeknights |
| **Platform preference** | PC (Steam) |
| **Current games they play** | Hades, Vampire Survivors, Dead Cells |
| **What they're looking for** | Tight skill-based combat with emergent moments and a high score to chase |
| **What would turn them away** | Cheap deaths without feedback; repetitive waves with no escalating tension; HUD-heavy UI |

---

## Technical Considerations

| Consideration | Assessment |
| ---- | ---- |
| **Engine** | Unreal Engine 5 — PC primary target, team has UE experience |
| **Key Technical Challenges** | Niagara particle system for corruption visualization; infection state machine; wave spawner pacing curves; corruption-based spell behavior switching |
| **Art Style** | 3D stylized dark fantasy — high-contrast spell effects against muted arena geometry |
| **Art Pipeline Complexity** | Medium — UE5 Mannequin for prototype, custom character and enemies later; Niagara is the art bottleneck |
| **Audio Needs** | Adaptive — corruption state shifts the audio profile (clean magic sounds crisp; corrupted magic sounds wet, hollow, wrong) |
| **Networking** | None |
| **Content Volume** | Prototype: 1 arena, 6 spell states (3 × clean/corrupted), 2 enemy types |
| **Procedural Systems** | Wave spawner (spawn count + wizard zombie ratio escalation per wave) |

---

## Risks and Open Questions

### Design Risks

- **Spell mutation feel requires iteration.** Getting corrupted variants to feel *meaningfully different* (not just cosmetically different) is hard to plan — budget explicit tuning time.
- **Visual clarity without HUD.** Communicating corruption state purely through particles and audio is harder than it sounds. First pass will likely be too subtle.
- **Wave escalation pacing.** Too fast = overwhelming too quickly; too slow = boring early waves. Requires playtest data to tune.

### Technical Risks

- **UE5 setup overhead.** Animation Blueprint, Enhanced Input, and Character Movement Component all need configuring before touching game systems. Expect 1–2 days before any gameplay.
- **Niagara corruption shader complexity.** Color-shifting particle systems that clearly communicate 3 states across a busy screen is non-trivial.
- **Infection state machine.** Managing the clean/infected/heavily-infected spectrum and its interaction with each spell's behavior variant needs clean architecture early.

### Market Risks

- **Wave survival is a crowded genre.** Vampire Survivors dominates the casual end; Hades owns the premium end. Spellrot must earn its place through feel quality, not novelty alone.
- **Corruption mechanic legibility.** If players can't read their state without a HUD bar, the core differentiator fails. Pillar 2 is a UX risk.

### Scope Risks

- **Corruption visual polish has no natural ceiling.** Must timebox particle tuning or it expands forever.
- **Spell mutation variety.** Temptation to add more corruption variants early; prototype with 3 and validate first.

### Open Questions

- **Purge balance:** How many regular zombie kills should it take to purge one corruption stage? Needs playtest data.
- **Cleanse zone size and placement:** How large, how many, and where in the arena? Needs playtesting to avoid trivializing or ignoring them.
- **Corruption ceiling:** Is "heavily infected" a hard cap, or can you go further? What happens at maximum corruption — death, transformation, something else?

---

## MVP Definition

**Core hypothesis:** Players find the corruption-modulated wave survival loop engaging for at least 2 consecutive runs without prompting.

**Required for MVP:**
1. Fluid spell casting — 3 spells, clean variants only (mutation comes after baseline is fun)
2. Endless wave spawner with regular zombies + wizard zombies
3. Kill-based purge — regular zombie kills reduce corruption
4. One cleanse zone in the arena
5. Corruption state visualization — particle color shift across 3 states, basic audio change
6. Death screen with wave count

**Explicitly NOT in MVP:**
- Corrupted spell variants (validate clean combat first)
- Meta-progression or unlocks
- Multiple arenas
- Named enemy types or boss encounters
- Narrative or UI beyond wave count

### Scope Tiers

| Tier | Content | Features | Timeline |
| ---- | ---- | ---- | ---- |
| **MVP** | 1 arena, 3 spells (clean only), 2 enemy types | Wave spawner, kill-purge, 1 cleanse zone, infection visualization | Weeks |
| **Vertical Slice** | 1 arena, 3 spells (clean + corrupted variants), 4 enemy types | Full infection system, multiple cleanse zones, wave escalation tuned | Months |
| **Alpha** | 2–3 arenas, 6 spell states | All core systems, rough meta-progression, basic scoring/leaderboard | 6–9 months |
| **Full Vision** | Multiple arenas, full spell tree, boss encounters | Complete game, polished, meta-progression, Steam release | 12–18 months, solo |

---

## Next Steps

- [ ] Run `/setup-engine unreal 5.8` to configure engine and populate version-aware reference docs
- [ ] Run `/art-bible` to create the visual identity specification before writing GDDs
- [ ] Run `/design-review design/gdd/game-concept.md` to validate concept completeness
- [ ] Decompose concept into systems with `/map-systems`
- [ ] Author per-system GDDs with `/design-system`
- [ ] Plan technical architecture with `/create-architecture`
- [ ] Prototype the core wave mechanic with `/prototype wave-combat`
- [ ] Validate with `/playtest-report` after prototype runs
- [ ] Plan first sprint with `/sprint-plan new`
