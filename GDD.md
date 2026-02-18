# 📘 Game Design Document (GDD) — *DataCenter Clicker: AI Tycoon*

## 1) High Concept
A **cute pixel-art incremental clicker** where the player is an AI cloud company building data centers worldwide. They **buy generators** (data center types), **unlock upgrades**, **survive events** (heatwaves, water bans, taxes), and **prestige** by “Rebooting the AI Model” to gain permanent multipliers. Modeled on **Adventure Capitalist** simplicity with sustainability and regulation twists.

**Core Pillars**
- Simple: one-click + automated income
- Strategic: diversify by region/type to mitigate events
- Satisfying growth: exponential upgrades and prestige loops
- Accessible: readable pixel interface, short sessions

---

## 2) Player Goals & Win/Lose
**Win Condition(s)**
- Reach target valuation (e.g., $1e15) **or**
- Achieve “Net-Zero Empire” (carbon score ≤ target for N turns) **or**
- Complete “Global Coverage” (own ≥1 DC in every region)

**Lose/Friction States**
- Temporary slowdowns (e.g., tax spikes, water bans)
- Bankruptcy soft-lock avoided with baseline click income (always ≥ 1)

---

## 3) Core Loop
1. Click the **Compute Button** to earn **Compute Credits** (CC).
2. Spend CC to buy **Income Generators** (automated).
3. Buy **Upgrades** (multipliers) and **Region Permits** (unlock regions).
4. React to **Events** that impact output/cost/rules.
5. **Prestige** to gain **AI Research Points (RP)** for permanent boosts.
6. Repeat with higher tiers & faster growth.

---

## 4) Resources & Currencies
- **Money** (Compute Credits, `cc`): main currency
- **Income** (`cc_per_sec`, `ccps`): passive CC
- **Click Power** (`click_cc`): CC per manual click
- **Carbon Score** (`carbon_rate`): emissions level (lower is better)
- **Water Usage** (`water_rate`): affects events & region penalties
- **Reputation** (`rep`): modifies event severity and bonuses
- **AI Research Points** (`rp`): prestige meta-currency

---

## 5) Economy & Formulas

### 5.1 Cost Growth (per generator)
- **Base Cost**: `base_cost`
- **Cost Multiplier per Purchase**: `cost_mult` (e.g., 1.15)
- **Nth Purchase Cost**:  `cost_n = base_cost * (cost_mult ^ owned_n)`

### 5.2 Income
- **Base Income**: `base_ccps`
- **Owned Generators**: `owned`
- **Type Multiplier**: `type_mult` (from upgrades)
- **Global Multiplier**: `global_mult` (from RP, achievements, events)
- **Event Multiplier (temporary)**: `event_mult`

**Total Income for Type**:  
`income_type = base_ccps * owned * type_mult * global_mult * event_mult`

### 5.3 Click Income
`click_cc = base_click * (1 + click_mult_global + click_mult_type)`

### 5.4 Carbon & Water
`carbon_rate = Σ(owned_type * base_carbon * type_mult_carbon * event_mult_carbon)`  
`water_rate = Σ(owned_type * base_water * type_mult_water * event_mult_water)`

### 5.5 Prestige (AI Research Points)
`rp_earned = floor( A * log10(total_lifetime_cc) + B * regions_owned + C * sustainability_bonus )`  
Suggested defaults: `A=1.5, B=2, C ∈ [0..20]`  
Each `rp` grants `+2% global income` (multiplicative): `global_mult_rp = (1.02 ^ rp)`

---

## 6) Generators (Buyables)
Follows Adventure Capitalist: exponential cost growth, occasional bulk thresholds (25/50/100/200) that grant a `×2` multiplier for that type.

See `./json/generators.json` for machine-readable content.

---

## 7) Upgrades (Multipliers & Unlocks)
- Global & click upgrades
- Type-specific upgrades
- Sustainability upgrades
- Threshold bonuses auto-applied on owned counts: 25/50/100/200 → `×2` each

See `./json/upgrades.json`.

---

## 8) Regions & Permits
- Regions unlock with a one-time **Permit** cost
- Regions apply **modifiers** to cost/output/carbon/event risks

See `./json/regions.json`.

---

## 9) Events & Laws (Forcing Diversification)
Declarative structure: `trigger` + `duration` + `effects` + optional `counters`.

See `./json/events.json`.

---

## 10) Taxes & Sustainability Systems
- Progressive compute tax (5%–25%) applied to carbon-heavy income
- Water restrictions if `water_rate` exceeds cap; remedied by sustainability upgrades
- Reputation reduces severity/duration of negative events

---

## 11) Prestige: AI Model Reboot
- Reset generators, money, upgrades (optionally keep region permits)
- Gain RP based on lifetime performance & sustainability
- Spend RP on permanent perks and unlocks

See `./json/prestige.json`.

---

## 12) UI & UX Layout
- **Top Bar**: Money, CC/s, Carbon, Water, Reputation, Active Events
- **Left**: Big Compute Button (manual income)
- **Center**: Pixel world map with region tiles & placed DCs
- **Right**: Shop (Generators), Upgrades, Permits; bulk buy ×1/×10/×25/Max
- **Bottom**: Prestige panel, Achievements, Settings

---

## 13) Art & Audio
- 16×16 or 32×32 pixel sprites, simple animations (fans, lights, weather)
- Chunky UI buttons, clear icons for water/carbon/tax/events
- Soft click & purchase SFX; ambient DC hum

---

## 14) Balancing Guidelines (MVP)
- Start `click_cc = 1`; first generator affordable within ~3–5 clicks
- First 10 minutes: reach Tier 2 & first event
- Event cadence: every 3–6 minutes; durations 60–180s
- Tax cap 25% to avoid hard stalls
- Prestige cadence: 15–30 minutes to first meaningful reset (5–15 RP)

---

## 15) Systems Pseudocode
```pseudo
loop tick (dt):
  ccps = 0
  for each generator g:
    mult = global_mult * type_mult[g.id] * tags_mult[g.tags] * region_mult[owner_region[g]] * event_mult[g]
    ccps += g.base_ccps * g.owned * mult

  tax = compute_tax(ccps, carbon_rate, active_events)
  net_ccps = max(ccps - tax, 0)
  money += net_ccps * dt

  apply_event_timers()
  update_ui(money, net_ccps, carbon_rate, water_rate, rep)

on_click_compute():
  money += click_cc

buy_generator(id, amount):
  total_cost = sum_for_n(amount) base_cost * cost_mult^(owned + i)
  if money >= total_cost: deduct money; owned += amount; recalc thresholds

buy_upgrade(id):
  if money >= cost and requirements_met: apply effects; deduct cost

prestige():
  rp_gain = floor(A * log10(lifetime_cc) + B*regions_owned + C*green_bonus)
  if rp_gain >= 1: reset_owned_and_money(); rp += rp_gain; apply_rp_perks()
```

---

## 16) Achievements (Optional)
- **First Cloud**: buy `regional_dc`
- **Green Dream**: 80% of income from renewable tags
- **Cooled Off**: survive heatwave with zero output loss
- **Diversifier**: own DCs in 6+ regions
- **Model Reboot**: first prestige

---

## 17) Accessibility & QoL
- Toggle for reduced animations
- Dyslexia-friendly font option
- Colorblind-friendly icons for water/carbon/tax
- Offline progress (optional): simulate `min(8 hours)` at reduced rate (25–50%)

---

## 18) MVP Scope (2–3 weeks)
- 10 generators, 12 upgrades, 6 regions, 5 events, 1 prestige path
- Single save slot (localStorage)
- Basic pixel art + SFX

**Stretch**
- Offline progress
- Multi-save
- Cosmetic skins for generators

---

## 19) Implementation Notes (for chatbot/template)
- Use the JSON under `./json/` as content sources
- Multipliers should be **multiplicative**, not additive
- Bulk-buy costs use geometric series (cost multiplier per purchase)
- Always allow manual clicking income even under events/taxes
- Event system should read `trigger` & `effects` declaratively from JSON

---

## 20) Handoff Notes
- JSON keys kept concise and consistent across files
- Designed to be extended without changing code (content-driven)
- All numbers are starter values; tune to taste
