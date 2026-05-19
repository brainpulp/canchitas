# Canchitas — Handoff

**Live:** https://brainpulp.github.io/canchitas/  
**Repo:** https://github.com/brainpulp/canchitas  
**Working file:** `F:\code\canchitas\index.html` (single file, no build step)  
**Last commit:** f1c1b1d — 2026-05-19 — repo moved to F:\code\canchitas

---

## What it is
Financial feasibility model for an Argentine sports complex (fútbol + paddle + escuelita) in zona norte GBA. All logic in one `index.html` — Chart.js 4.4.4 + Supabase JS v2, both via CDN.

---

## Stack
| Thing | Detail |
|---|---|
| Hosting | GitHub Pages (main branch → auto-deploy ~2 min) |
| Persistence | Supabase `kbatdnrxfrltcmqvsmyy`, table `canchitas_scenarios` — 4 slots, anon RLS |
| Chart | Chart.js 4.4.4 — `ch2` only (stacked bar + line overlays, toggles per dataset) |
| Auth | None — shared anonymous state, same URL for everyone |
| Theme | Light/dark toggle (`toggleTheme()`), stored in localStorage |

---

## Key globals
```js
P                // all parameters
EMP[]            // employees: {n, q, s, m} (name, qty, ARS salary, start month)
CUSTOM_GASTOS[], CUSTOM_INGRESOS[], CUSTOM_CAPEX[]
OCC_PTS[]        // draggable occupancy curve (quarterly multipliers, 100 = P.oc base)
DEVAL_EVENTS[]   // crisis devaluation events: {month, jump, postRate}
slots[4]         // scenario slots; aSlot = active index
partVis          // {p1, p2, p3}  partner checkbox state
chartVis2        // {rev, cost, p1, p2, p3, acum, net, sp}  bar chart visibility
chatHistory[], chartEvents[]
```

## P object — full field reference
```js
// Fútbol
co, ohd, oc, pr, rc

// Escuelita
st, fe, rs, stRatio, stSalArs

// Paddle
po, pn, pod, pa, pp, rp

// Adicionales
buOn, evOn, toOn, pInvM

// Empleados
eBlanco, eCarg, indemFam   // indemFam = % staff familia/amigos (lower dismissal risk)

// Parámetros generales
fx, meses, negro, obra, seas, startMonth, startYear

// Impuestos
ibRate, chequeRate, ganRate

// Financiamiento
finCOn, finCCuotas, finCTasa, finPdOn, finPdCuotas, finPdTasa

// Socios
pn1, pn2, pn3              // names: "Maxi", "Pato", "Julio"
sp1                        // legacy (overridden by waterfall)

// Waterfall (NEW: independent per-partner splits)
trig1Pct                   // % of totalInv Maxi must accumulate → enters Phase 1
trig1Sp1, trig1Sp2         // Maxi %, Pato % in Phase 1 (Julio gets remainder)
trig2Pct                   // % of totalInv → enters Phase 2 (permanent)
trig2Sp1, trig2Sp2         // Maxi %, Pato % in Phase 2 (Julio gets remainder)

// Devaluación
devalOn, devalMonthly
```

## Waterfall dividend logic (calc())
- **Phase 0:** 100% → Maxi until `maxiCumDiv ≥ trig1Pct/100 × totalInv`
- **Phase 1:** `trig1Sp1 / trig1Sp2 / (100-trig1Sp1-trig1Sp2)` until `maxiCumDiv ≥ trig2Pct/100 × totalInv`
- **Phase 2:** `trig2Sp1 / trig2Sp2 / (100-trig2Sp1-trig2Sp2)`, permanent
- Pato/Julio = $0 in Phase 0. Both triggers are **cumulative from day 1**, not incremental.

## Devaluation model
- Smooth crawl: monthly FX = `P.fx × (1 + devalMonthly/100)^m`
- Crisis events (`DEVAL_EVENTS`): each event has `{month, jump%, postRate%}` — FX jumps at that month then continues at the new rate
- `fxRatio = P.fx / fxM` applied to ARS-denominated costs; USD revenues unaffected

## calc() key return values
```js
pn1D, pn2D, pn3D           // monthly dividends per partner
pn1CumD, pn2CumD, pn3CumD // cumulative per partner
grossD, costsD, netD        // monthly revenue, costs, net
pn1Cum, pn2Cum, pn3Cum     // final cumulative totals
```

## Persistence
- `saveLS()` → localStorage + Supabase (1.5s debounce via `sbSave`) + `_writeFile()`
- Load order: localStorage → Supabase → `scenarios.json` fallback
- Anon key embedded in file

## Deploy
```bash
cd F:\code\canchitas
git add index.html HANDOFF.md
git commit -m "..."
git push origin main
# Pages live in ~2 min
```

---

## Partner cards UX
- Click a card → toggles that partner's data on/off in the chart (`partVis.p1/p2/p3`)
- To edit split %: open the **🏦 Distribución de ganancias** accordion in the left panel
- No inline edit button on cards (removed — was overlapping/broken)

## Scenario management
- Click a slot pill → load it (active = blue border)
- `×` on a pill → first click turns pill red + shows **¿Borrar? ✓ ✗** (two-step, no accidental deletes)
- Double-click slot name → rename inline
- `+ nuevo` button → saves current params as a new slot
- Max 4 slots

## Completed items (as of 2026-05-18)
- ✅ Consolidated to one chart (`ch2` bar + line overlays; `ch` line chart dropped)
- ✅ Chart.js legend removed — partner cards are the sole visibility toggle
- ✅ Acumulado lines react to partner checkboxes (`partVis`)
- ✅ Independent Pato/Julio splits (`trig1Sp2`, `trig2Sp2`)
- ✅ Dark/light theme toggle
- ✅ DEVAL_EVENTS crisis model
- ✅ Full tax model (IB, cheque, ganancias)
- ✅ Teacher salary model (stRatio, stSalArs, profN computed)
- ✅ Indem provision (indemFam)
- ✅ Seasonality (seas), startMonth/startYear
- ✅ Financing section
- ✅ Removed dead startEditP1/confirmP1/editP1 code
- ✅ Two-step confirm on scenario delete (`pendingDelete` state, ¿Borrar? ✓ ✗)

## Next items / open questions
1. **Waterfall UX clarity** — trigger labels still confusing; show all 3 partners' % in accordion header
2. **Reality check section** — verify calcs with new independent splits (trig1Sp2/trig2Sp2)
3. **Supabase schema** — verify trig1Sp2/trig2Sp2 round-trip through saved slots

---

## UI sections (left panel accordions)
⚽ Canchas fútbol · 🎓 Escuelita · 🏓 Paddle · 🍔 Otros ingresos · 💰 Inversión · 💳 Financiamiento · 🔧 CAPEX · 👥 Empleados · 📊 Gastos fijos · ⚙️ Parámetros · 📉 Devaluación · 🏦 Waterfall dividendos · 🔍 Reality Check

## UI sections (right panel)
KPI strip (kRow2 + kRow) · Seasonality strip · OCC draggable curve · Partner checkboxes + cards · Bar chart toggles · `ch2` (stacked bar + line overlays)
