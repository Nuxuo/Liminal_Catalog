# Combat: damage types, on-hit and effects

*Agreed on 2026-09-26 in a refine session, from the topic "Damage types, on-hit and effects" in [`Docs/Ideas.md`](../Ideas.md). Needed before step 6 of the plan in [`Docs/Architecture.md`](../Architecture.md), where hits and damage get built. Step sessions build from it, after challenging the part they build.*

> **This is an iteration, not a final plan or decision.** It's where the discussion stood on 2026-09-26, and any part of it can still change.
>
> When it's time to build a part of it, that part must be challenged and discussed first, against the code and the plan as they are then. If something better turns up, this file changes before anything is built.

## In short

- Every building has an **attack**: the turret's shot, the mortar's shell. The attack is marked with a **damage type**, **Physical** or **Magic**, and so is every other piece of damage.
- A hit comes from the attack itself (a **direct hit**) or from an **effect** the attack starts: an **explosion**, whose hits are **area hits**, or a **chain** like **Arc**. AOE is a kind of hit, not a damage type.
- **Armor** lowers Physical damage and **magic resist** lowers Magic damage, on a curve that never makes an enemy immune. Statuses like **Shred** can push them below zero, so the enemy takes extra damage. A building's **penetration** only strips them down to zero.
- **On-hit** is a list each building has: damage, statuses, and "damage over time once more". It runs once on every direct hit. A chain's hits get it only through an augment ("Arc applies on-hit"), and area hits never do. What on-hit adds sets off nothing.
- **Effects** have names (Arc, Shell Blast, Explosive Bullets) and their own numbers. Only a building's attack starts them, each with its own chance. Their hits never start another effect.
- **Rules** react to a hit: to the attack's damage type ("magic applies Burn"), to direct or area hits ("area hits deal +10%"), or to a named effect. They add statuses and raise damage, and never add damage or start anything.
- **Statuses** (Burn, Poison, Shred …) are named kinds that share one rule set: what each stack does, a stack limit, a duration, and what happens when they land again. Damage over time is never a hit.
- **Stats come in two kinds:** a **percentage** stops at 100%, a **number** is endless. So a building can reach 100% crit, and fire rate without end.
- **Nothing is cut to protect speed.** When a tick has more work than the PC can do, the game slows down: the PC is the limit.
- Together, these rules make loops impossible: every hit traces back to one attack, at most one step away.
- Nearly all of it is worked out when stats are baked, the plan's word for working out final values once, when something changes. While the game runs, the Sim only sees plain numbers and steps.

## Words

| Word | Meaning |
|---|---|
| **Attack** | What a building fires: the turret's shot, the mortar's shell. Marked with a damage type. |
| **Hit** | One enemy struck at one moment. Damage over time and reaching the core aren't hits. |
| **Direct hit** | The attack itself struck the enemy. Every direct hit runs on-hit. |
| **Effect** | Something an attack starts that makes more hits: an explosion or a chain. Each has a name and its own numbers: Arc, Shell Blast, Explosive Bullets. |
| **Area hit** | An enemy caught in an explosion. Area hits never run on-hit. |
| **Damage type** | Physical or Magic, more later. Every piece of damage has one. |
| **Armor, magic resist** | The two defenses. Armor lowers Physical damage, magic resist lowers Magic damage. |
| **Reduction** | A status lowering an enemy's armor or magic resist, for every building. It can go below zero. Shred is one. |
| **Penetration** | A building's own stat that ignores part of a defense, for its own hits only, and only down to zero. |
| **On-hit** | A building's list of damage and statuses that runs once on every direct hit its attack makes. |
| **Status** | A named, timed condition on an enemy, with stacks: Burn, Poison, Shred, Slow, Freeze. |
| **Rule** | An augment that reacts to a hit: to the attack's damage type, to direct or area hits, or to a named effect. |
| **Bridge** | An augment that lets a chain's hits run on-hit: "Arc applies on-hit". Explosions never get one. |
| **Percentage, number** | The two kinds of stat. A percentage stops at 100%; a number has no cap. |

## 1. Attacks and damage types

- Every building kind has an attack, and the attack is marked **Physical** or **Magic**. The turret's shot and the mortar's shell are Physical.
- Every other piece of damage is marked too:
  - on-hit's damage names its own type: "on-hit: +5 Magic";
  - damage over time takes its status's type: Burn is Magic, Poison is Physical;
  - the hits of an explosion or a chain carry the attack's damage and its mark.
- One hit can carry both types. A Physical turret with "on-hit: +5 Magic" hits for 2 Physical and 5 Magic, and each part goes through its own defense (section 3).
- Rules for a damage type react only to the attack's mark (section 6). On-hit's damage sets off nothing.
- A new damage type later is data: the type, and the defense that lowers it.

**Why two types:** a damage type's job is to say which defense lowers it. There are two defenses, so there are two types.

## 2. Where a hit comes from

| The enemy … | It's a … | Example |
|---|---|---|
| was struck by the attack itself | **direct hit** | a turret's shot |
| was caught in an explosion the attack started | **area hit** | the mortar's Shell Blast, a turret's Explosive Bullets |
| was reached by a hop of a chain the attack started | **hit of that chain** | Arc |

- The attack starts its effects from its direct hit, or from where its shell lands. Explosive Bullets explode where the turret's shot strikes, and Arc jumps on from there.
- A chain's hits count as that chain's, not as direct hits. That keeps on-hit off them unless an augment adds it (section 4).

**Why AOE is a kind of hit, not a damage type:**
- If AOE were a type, the game would need to say which defense it meets. Armor makes it Physical under another name. Neither makes it the answer to every tough enemy. A new "area resist" is really a resist against explosions, not against a kind of damage.
- A magic building that explodes would have to be Magic or AOE, never both, so it would miss either the magic rules or the area rules.

Everything one would say about AOE still works, as a rule on area hits: "area hits deal +10%", "AOE applies Poison". If enemies that shrug off explosions are ever wanted, they get a resist against area hits.

## 3. Armor and magic resist

The two defenses replace the plan's armor pool, a layer that had to be broken before damage reached health.

**Why:**
- A pool only matters until it breaks. A resist matters on every hit, so which damage type you bring always matters.
- Two defenses give the two damage types a real job.
- A resist costs about the same as the pool, per hit and per enemy, and copes with endless growth because it never makes an enemy immune.

What's gone: breaking the shell, and conditions like "+30% vs broken". A shield that has to be broken can still come later as its own thing.

### The curve

The first points of armor block the most, each further point blocks a little less, and nothing ever blocks 100%. Below zero the curve mirrors: extra damage grows the same way and never passes double. Magic resist works the same against Magic damage.

| Armor | Damage blocked | Armor | Extra damage taken |
|---|---|---|---|
| 10 | 9% | −10 | +9% |
| 50 | 33% | −50 | +33% |
| 90 | 47% | −90 | +47% |
| 100 | 50% | −100 | +50% |
| 300 | 75% | −300 | +75% |

In numbers, with R the armor left after reductions and penetration: damage × 100 ÷ (100 + R) when R is zero or more, and damage × (2 − 100 ÷ (100 − R)) when R is below zero.

- **The half point.** The 100 in those numbers is the armor that blocks half. It's one tuning number: at 60, 90 armor blocks 60% and 10 armor blocks 14%.
- **Why not steeper.** A curve that blocked exactly 10% at 10 armor and 60% at 90 would block 96% at 300, so enemies whose armor grows every wave would soon be nearly immune. With this curve every point of armor adds the same toughness (with the half point at 100, each point is 1% more health against that damage type), so armor growing each wave works like health growing each wave.

### Going below zero: reductions

- Statuses on the enemy lower its armor or magic resist for **every** building, and can push it below zero. **Shred** is the first: "−10 armor per stack". A twin of Shred lowers magic resist the same way.
- Every source adds to the same stacks. The mortar's explosion adds one Shred stack and a turret's on-hit adds another: 2 stacks, −20 armor.
- Enemy types can also start below zero, for fragile enemies.

### Penetration

- A building's own stat. **Percent** penetration ignores a share of the defense, **flat** penetration a fixed amount.
- It only works for that building's own hits, and only down to zero.
- **Why stop at zero:** otherwise endless penetration stacks would turn into plain extra damage against everything, even enemies with no armor. Stopped at zero, penetration stays the answer to armor, and going below zero is teamwork: one building shreds, and every building profits.

### The order

For each piece of damage in a hit:
1. Start from the enemy type's armor, or its magic resist for Magic damage.
2. Subtract reductions from statuses. This can go below zero.
3. If it's still above zero, subtract the attacking building's penetration: percent first, then flat, stopping at zero.
4. Apply the curve.

The damage-taken multiplier from statuses like Vulnerable (a plan status) applies as well. Damage over time goes through the same order, with the penetration set on its status kind.

**Examples:**
- An enemy with 120 armor carries Shred −40, so it's at 80. A turret with 25% and 10 flat penetration: 80 × 0.75 = 60, minus 10 = 50, so it takes ×0.67.
- The same turret against an enemy with 20 armor and the same Shred: it's at −20, penetration does nothing, and it takes ×1.17.
- An enemy with 15 armor and 2 Shred stacks is at −5, and takes 5% more Physical damage from every building.

## 4. On-hit

- **What it is.** A list every building has. Augments add to it for one building kind ("turret on-hit: +1 damage") or for every building ("on-hit: +1 damage").
- **What it can hold:**
  - damage, marked with its own type: "on-hit: +5 Magic";
  - statuses: "on-hit applies Poison";
  - **damage over time once more:** each on-hit deals one second's worth of every damage over time on that enemy, right away. The statuses keep running.

  It never holds explosions or chains. Those belong to a building's attack (section 5).
- **When it runs:**
  - once on every direct hit, whatever the attack's damage type;
  - on a chain's hits only when an augment adds it: "Arc applies on-hit";
  - never on area hits, and never set off by damage over time.
- **It's a one-time thing.** What on-hit adds sets off nothing: no other on-hit, no effect, no rule. "On-hit: +5 Magic" goes through magic resist, and that's all it does.
- **How strong:** on-hit's damage is added as it is. Crit, rules and conditions raise only the attack's damage (sections 6 and 8). Along a chain, on-hit's damage weakens by the chain's falloff like the rest.

**Examples:**
- Three picks of "on-hit: +1" take today's turret from 2 to 5 per shot (×2.5), but a slow 20-damage building only from 20 to 23 (×1.15). Flat on-hit favours many small hits, which gives fast buildings their own path. Because enemy health grows every wave, flat on-hit will need percent augments later (the plan's decision 9).
- Damage over time once more: an enemy has 5 Burn at 4 per second and 3 Poison at 2 per second. An on-hit deals +20 Magic and +6 Physical, through magic resist and armor. The "one second" is a stat to tune: the faster a building attacks, the more it counts.

**Why direct hits, and never area hits:**
- **Cost follows attacks, not crowds.** An explosion catching 300 enemies (thousands in the jam at the core), each running on-hit, was the heaviest case in the design.
- **Balance.** Flat on-hit damage times 300 enemies per shell was the biggest multiplier in the design. On direct hits, a "+1" is easy to judge.
- **Fewer edge cases.** On-hit never has to be weakened at an explosion's edge.
- **Clearer builds.** On-hit is the path for fast single-target buildings. Area buildings grow through rules on area hits. Statuses link the two.

## 5. Effects: explosions and chains

- **Every effect has a name and its own numbers:**
  - a chain, such as Arc: how many hops it makes, how far each hop reaches, and how much weaker each hop is;
  - an explosion, such as Shell Blast or Explosive Bullets: its radius, how much weaker it is at the edge, and the most enemies it can catch.
- **Only a building's attack starts them,** each with its own chance. Augments give them to a building kind: "Turrets: Explosive Bullets" (every shot explodes where it strikes), "Turrets: 10% chance to Arc", "Mortars: 10% chance to Arc" (the mortar's Arc jumps outward from its explosion). A building can have one built in: the mortar's shell always ends in Shell Blast.
- **Their hits never start another effect.**
- **Order:** within one attack, the explosion comes first and the chain second, as in the plan. A chain that follows an explosion reaches past it.
- **Their hits carry the attack's damage and mark,** weaker by distance in an explosion and per hop in a chain.
- **A full explosion** hits the nearest enemies up to its most targets. When more are inside, the rest aren't touched, and the ones it hits don't take extra damage for them.
- **Sharing:** several buildings can use the same effect, and one building can have several, like a turret with Arc and Explosive Bullets. Each keeps its own numbers.
- **Aiming at them:** "Arc hops +1" reaches every building's Arc. "Shell Blast radius +20%" and "Shell Blast applies Shred" reach only Shell Blast. "Turrets: Arc chance +5%" changes only the turrets' chance. A chain can get a bridge ("Arc applies on-hit"); an explosion never can.

**Why names:**
- Augments can reach exactly one effect.
- They're needed anyway: once a turret can have Arc and Explosive Bullets, one set of explosion or chain numbers per building isn't enough, and the plan would merge them.
- An effect several buildings share lets every augment on it reach all of them.
- They cost nothing while the game runs: names are sorted out when stats are baked.

**Why on buildings,** and not "all Physical attacks: chance to Arc" or on-hit's own chance to explode:
- Each building gets its own path, and the augment says plainly where the effect goes.
- Each building's chance is tuned on its own: a 10% Arc on a fast turret and on a slow mortar are very different things.
- On-hit's "10% chance to explode" would do the same, for a turret, as Explosive Bullets at 10%, so keeping both would be two ways to do one thing.

## 6. Rules

Most augments are plain numbers on buildings, as in the plan ("turrets fire 5% faster"). **Rules** are the augments that react to a hit:
- **to the attack's damage type:** "magic applies Burn", "Physical damage +10%". They react to the attack's mark, on its direct hit and on its effects' hits. On-hit's damage never sets one off.
- **to direct or area hits:** "direct hits deal +10%", "area hits deal +10%", "AOE applies Poison".
- **to a named effect:** "Arc hits apply Shred", "Shell Blast applies Shred".

Rules can add statuses and raise the attack's damage in the hits they react to. On-hit's damage stays as it is.

Rules never add damage or start an explosion or chain. They react to hits, so if they could add damage, "Physical adds Magic" and "Magic adds Physical" would feed each other forever inside one hit.

Conditions on the enemy work as in the plan: "+20% damage to burning enemies". A hit sees the enemy as it was before the hit, so the hit that sets Burn doesn't get "+20% vs burning"; the next one does.

## 7. Statuses

Every status is a named kind, and all of them share one rule set. Each kind sets:
- **what each stack does:** damage per second (Physical or Magic), lower armor, lower magic resist, slow, "takes more damage", or can't move;
- **a stack limit;**
- **a duration;**
- **what happens when it lands again:** add stacks and restart the timer; only restart the timer; keep the strongest; or ignore it, optionally followed by a pause before it can land again;
- **a priority,** for which one gives way when an enemy is full.

| Status | Each stack | Stack limit | Lands again | Lasts |
|---|---|---|---|---|
| Burn | Magic damage per second | 5 | adds stacks, restarts timer | 3 s |
| Poison | Physical damage per second | 10 | adds stacks, restarts timer | 4 s |
| Shred | −10 armor | 5 | adds stacks, restarts timer | 5 s |
| Slow | speed ×0.6 (counts once, not per stack) | 1 | keeps the strongest | 2 s |
| Freeze | can't move | 1 | ignored, then 3 s before it can land again | 1 s |

The numbers are starting points.

- **Stacks:** damage over time counts per stack. 3 Poison at 2 per second deal 6 per second, through armor.
- **Stack limit:** one per status for the whole run. "Poison stack limit +2" raises Poison's to 12, and there's no cap. The plan's status storage has room for only 127 stacks, so step 14 gives the count more room.
- **Numbers come from the applier:** how strong each stack is, how long it lasts and how many stacks one hit adds come from the building that applies it: its own value, else the augment that gave it the status, else the status's default. So "Poison damage +20%" and "mortar Poison adds 2 stacks" work like any other augment.
- **One pile per status:** Poison from the mortar and from a turret's on-hit go into the same pile. If their strengths differ, the pile averages them. Once the pile is full, a stronger application replaces weaker stacks and a weaker one only restarts the timer, so a weak source never waters down a strong one.
- **Damage over time is never a hit:** it sets off neither on-hit nor rules. A kill from it goes to the building that applied the status (the plan's decision 16 assumes the most recent one).
- **New statuses:** one that only uses the options above is a new asset, with no code. One that needs something new, like knock-back, needs code once; after that, any status can use it.
- **How many at once:** an enemy carries a limited number of different statuses at once, each with any number of stacks. If one more lands, the lowest priority gives way. The number is decided at plan step 14 (section 12).

## 8. Stats

Every number is a stat, worked out with the plan's formula when something changes: the base, plus adds, then percents, then multipliers (the plan's §2.10). Kinds give the base values; augments, cells and statuses change them.

### Two kinds of stat

| Kind | Examples | Cap |
|---|---|---|
| **Percentage** | crit chance, the chance to start an effect, percent penetration, how much weaker an explosion is at its edge, chain falloff | stops at 100%; after that, augments that raise it aren't offered anymore |
| **Number** | damage, fire rate, range, explosion radius, most targets, hops, stack limit, crit damage, flat penetration, on-hit damage, health, armor, magic resist | none: endless |

So a building can reach 100% crit, and fire rate without end. That's the point of the game: endless and crazy. Nothing past 100% turns into anything else, and a percentage augment simply stops being offered.

Endless numbers cost more as they grow. A huge explosion radius catches more enemies, and a huge fire rate fires more shots each tick. Nothing caps them: when a tick has more work than the PC can do, the game slows down (section 11). Two of them need room in how the data is built:
- **Fire rate above 30 attacks a second** means several attacks from one building in the same tick. The plan's salvos carry them as one shot row, applied one after another. Here they have no ceiling: every shot counts.
- **A stack limit past 127** needs a wider stack count in each status spot, at step 14.

**Enemy stats,** per enemy type (one row per type and wave; the plan's decision 5 says which grow each wave):

| Stat | What it does |
|---|---|
| Health | how much damage it takes to kill |
| Armor | lowers Physical damage (section 3) |
| Magic resist | lowers Magic damage (section 3) |
| Speed, radius, damage to the core, bounty, score | as in the plan |

Each enemy also keeps its current health, and the reductions its statuses put on its armor and magic resist. Health regen may come later: it's a raw topic in [`Docs/Ideas.md`](../Ideas.md).

**Building stats,** per building kind, changed by augments:

| Stat | What it does |
|---|---|
| Damage and its type | what one attack deals, and its mark (section 1) |
| Fire Rate | attacks per second: 1.5 means one and a half attacks a second |
| Range | how far it reaches |
| Armor penetration, magic penetration | each percent and flat; ignores part of the matching defense for its own hits, down to zero (section 3) |
| Crit Chance, Crit Damage | the chance an attack crits, and how much a crit multiplies its damage (×2 to start) |
| On-hit | its damage (each part with a type), the numbers of its statuses, "damage over time once more" (section 4) |
| Chance to start an effect | per effect: "Turrets: 10% chance to Arc" (section 5) |
| An effect's numbers | per effect: Arc's hops, reach and falloff; Shell Blast's radius, edge and most targets (section 5) |
| The numbers of each status it applies | strength per stack, duration, stacks per hit (section 7) |

**Per status kind, for the whole run:** the stack limit (section 7).

### Crit

Each attack rolls once for a crit. A crit multiplies the attack's damage in every hit it makes: direct, explosion and chain. It doesn't multiply on-hit's damage or damage over time, so crit is the path for big hits and on-hit the path for many hits.

## 9. What sets off what

Four rules:
1. **Effects** (explosions and chains) start only from a building's attack: from its direct hit, or where its shell lands, each by its own chance. Their hits never start another effect.
2. **On-hit** runs once on every direct hit, and on a chain's hits when an augment adds it; never on area hits. It only acts on the enemy it hit, and what it adds sets off nothing.
3. **Rules** react to the attack's damage type, to direct or area hits, or to a named effect. They add statuses and raise damage. They never add damage or start anything.
4. **Damage over time** is never a hit.

So nothing can loop: every hit traces back to one attack, at most one step away.

| The enemy was struck by | Its damage | Rules | On-hit |
|---|---|---|---|
| the attack, directly | the attack's | yes | yes, always |
| an explosion | the attack's, weaker toward the edge | yes | never |
| a chain, such as Arc | the attack's, weaker each hop | yes | only with that chain's bridge |
| damage over time | the status's | no | no |

**Within one hit,** in this order:
1. The attack's damage, weakened for explosion and chain hits; crit, rules and conditions raise it.
2. On-hit's damage, if on-hit runs, added as it is.
3. Each piece of damage goes through its defense (section 3) and comes off health.
4. The statuses from the attack, the rules and on-hit are applied, unless the enemy just died.
5. If this was the attack's direct hit, or where its shell landed, the attack's effects start from here, each by its chance.

The plan's on-death effects ("burning enemies explode when they die", room for now) would be a new way to start hits. They'll need to fit these rules when they're refined.

## 10. Examples

**A turret with a chain.** Physical, 2 damage, with "Turrets: 10% chance to Arc", "Arc applies on-hit", and an on-hit that applies Poison.
- Enemy A, struck directly: 2 Physical through armor, and on-hit's Poison. One time in ten, Arc starts from A.
- Arc's hops B and C: the turret's 2 damage, weaker each hop. The bridge lets on-hit run, so they get Poison too. They can't start anything.
- Add "Turrets: Explosive Bullets": every shot also explodes around A, and the enemies it catches take the turret's damage, weaker toward the edge. They never get on-hit.

**The mortar.** A Physical shell that always ends in Shell Blast, with "Shell Blast applies Shred" and "area hits deal +10%".
- Every enemy caught takes the shell's damage +10%, weaker toward the edge, and a stack of Shred. No on-hit.
- Turrets then hit the shredded crowd harder: the mortar and the turrets link through the status.

**Two damage types in one hit.** A Physical turret with "on-hit: +5 Magic" deals 2 Physical through armor and 5 Magic through magic resist. With "magic applies Burn" taken, it still doesn't burn, because its attack is Physical. A Magic building with Arc does burn, on its direct hits and on every hop.

**Loops that can't happen:**
- A burn tick isn't a hit, so "magic applies Burn" can't renew Burn from Burn's own ticks.
- On-hit's damage sets off nothing, so it can't start on-hit again or set off a rule.
- Arc's hops can't start Arc or Explosive Bullets.
- Rules can't add damage, so "Physical adds Magic" can't bounce back and forth.

## 11. Cost

Cost follows how many enemies are struck per tick, not how many are alive. The plan's prototype measured 30–50 ns per enemy struck (damage, defense and one status) and 35–100 ns per extra status, on one thread (the plan's A.6). The rest here is estimated from that.

- **On-hit:** its damage is almost free, added to the hit. Each on-hit status is one more application. "Damage over time once more" reads the enemy's status slots, one small read. On-hit follows direct hits and bridged chain hits, never the crowds explosions catch.
- **Explosions and chains** always do all their work. There's no per-tick budget and no cap on numbers: a bigger radius catches more enemies, and more hops reach more. When a tick has more to do than the PC can manage, it takes longer and the game slows down.
  - **What slowing down looks like:** the plan's clock waits for a long tick and never runs extra ticks to catch up, so the game moves in slow motion instead of freezing. The picture keeps being drawn, though the frame rate can drop too.
  - **For scale:** one explosion catching 300,000 enemies in the jam would take about 20–45 ms on its own.
- **Direct hits** grow with fire rate, since one building can land several shots in the same tick. A direct hit with two on-hit statuses costs about 100–250 ns, so 10,000 of them in a tick add about 1–2.5 ms.
  - When shots of one salvo pour into the same enemy, their statuses are applied as one update, as the plan already does for shells.
  - The stress scene's late-game load gets on-hit with two statuses, "damage over time once more" and chain bridges.
- **Chances** stop at 100%, so an effect never starts twice from one chance.
- **Memory:** base armor and magic resist live in the enemy type's row. Per enemy, the plan's armor pool (4 bytes) becomes an armor reduction (4 bytes), and a magic-resist reduction (4 bytes) joins with the first status that lowers magic resist.
- **Defenses per hit** cost about the same as the pool did.

## 12. Decided later

| What | When | Starting point |
|---|---|---|
| **How many different statuses an enemy can carry at once** | plan step 14 | 4, built as one setting. Raised to 8 if the count of statuses that had to give way shows it matters: that costs about 0.8 ms per tick and 64 MB more at a million enemies, plus one byte per enemy to mark which spots are in use. |
| **How much room a stack count gets** | plan step 14 | the plan's 127 is too small for a stack limit without a cap |
| **The half point** of the defense curve | when enemies with armor or magic resist arrive | 100 |
| **How much "damage over time once more" deals** | when it's built | one second's worth |
| **Whether armor and magic resist grow each wave** | plan step 8 (decision 5) | the plan assumed armor grows with toughness |

## 13. How this changed from the idea

- **Damage types.** "Attack, magic, explosion" became Physical and Magic marks on attacks and on every piece of damage. Explosion (AOE) became a kind of hit.
- **On-hit.** "Attack damage always triggers on-hit, other types through augments" became: on-hit runs once on every direct hit, whatever its damage type; a chain's hits get it through an augment; area hits never do; and what on-hit adds sets off nothing.
- **"Explosions apply on-hit"** was dropped, for the reasons in section 4.
- **"Magic damage applies burn for 1 s"** stayed, as a rule set off by a Magic attack's hits, not by on-hit's Magic damage.
- **"On-hit deals +x damage"** stayed: flat, and marked with a type.
- **Loops** can't happen, by the four rules, so the plan's "generation cap" for combat rules isn't needed.
- **No per-tick budget.** The plan cut explosions and chains on very busy ticks to protect speed, and moved their damage onto the enemies it kept. Now the game does all the work and slows down instead: the PC is the limit. Nothing else moves damage around either. For example, an explosion with more enemies inside than its most targets simply hits the nearest ones.
- **Armor**, a "maybe" the plan had read as a pool, became armor and magic resist, with a curve, reductions and penetration.
- **New along the way:** named effects, "damage over time once more", effects given to buildings with a chance, one rule set for all statuses (mostly the plan's), a list of stats, crit, and two kinds of stat: percentages stop at 100%, numbers are endless.
- **Health regen** came up and went to [`Docs/Ideas.md`](../Ideas.md) as its own topic.

## 14. Where it goes in the plan

- **Step 6:** hits carry a damage type; only Physical exists yet. Fire rate has no cap: a building's shots due in one tick travel as one salvo, as in the plan, but without its ceiling, so every shot counts.
- **Step 10:** attacks carry a damage type; each effect step says which hits it runs on (direct, area, or a named chain's); explosions and chains become named effects with their own numbers (stat keys by the effect's name, as status stats are keyed by the status kind).
- **Step 11:** on-hit and its first augment ("on-hit: +1 damage"); rules for damage types, direct or area hits, and named effects; Crit Chance and Crit Damage; the two kinds of stat, and augments that stop being offered once a percentage is at 100%.
- **Step 12a:** no per-tick budget: its cuts, the damage it moves onto kept targets, and its hard stop aren't built.
- **Step 12b:** the mortar's Shell Blast as the first named explosion, hitting the nearest enemies up to its most targets; rules on area hits.
- **Step 13:** Arc; "Turrets: X% chance to Arc"; "Arc hops +1"; "Arc applies on-hit".
- **Step 14:** Burn, Poison and Shred; armor and magic resist instead of the pool; penetration; Magic, with a first magic building; "magic applies Burn"; "damage over time once more"; how many statuses an enemy can carry, and room for stack counts past 127.
