# Canchitas — Handoff

**Live:** https://brainpulp.github.io/canchitas/  
**Repo:** https://github.com/brainpulp/canchitas  
**Local:** `F:\codetests\cleanup\canchitas\index.html` (~2164 lines, single file, no build step)

---

## What it is
Financial feasibility model for an Argentine sports complex (fútbol + paddle + escuelita) in zona norte GBA. All logic in one `index.html` — Chart.js 4.4.4 + Supabase JS v2, both via CDN.

---

## Stack
| Thing | Detail |
|---|---|
| Hosting | GitHub Pages (main branch → auto-deploy ~2 min) |
| Persistence | Supabase project `kbatdnrxfrltcmqvsmyy`, table `canchitas_scenarios` — 4 slots, anon RLS |
| Charts | Chart.js 4.4.4 — `ch` (line) + `ch2` (stacked bar) |
| Auth | None — shared anonymous state, same URL for everyone |

---

## Key globals
```js
P            // all parameters (see P object section below)
EMP[]        // employees: {n, q, s, m}  (name, qty, ARS salary, start month)
CUSTOM_GASTOS[], CUSTOM_INGRESOS[], CUSTOM_CAPEX[]
OCC_PTS[]    // draggable occupancy curve (quarterly multipliers, 100 = P.oc base)
slots[4]     // scenario slots; aSlot = active index
partVis      // {p1, p2, p3}  partner checkbox state
chartVis2    // {rev, cost, p1, p2, p3, acum}  bar chart visibility
```

## Key P fields (additions beyond the original)
```js
pn1, pn2, pn3          // partner names: "Maxi", "Pato", "Julio"
trig1Pct, trig1Sp1     // Trigger 1: when Maxi cum divs = X% of inv → T1 split (Maxi gets trig1Sp1%)
trig2Pct, trig2Sp1     // Trigger 2: when cum = Y% of inv → T2 split (permanent)
devalOn, devalMonthly  // ARS devaluation: monthly FX crawl %, affects ARS-denominated costs
```

## Waterfall dividend logic (calc())
- **Phase 0:** 100% of net profit → Maxi until `maxiCumDiv ≥ trig1Pct/100 × totalInv`
- **Phase 1:** split `trig1Sp1 / ((1-trig1Sp1)/2) / ((1-trig1Sp1)/2)` until `maxiCumDiv ≥ trig2Pct/100 × totalInv`
- **Phase 2:** `trig2Sp1` split, permanent
- Pato/Julio = $0 during Phase 0. Both triggers are **cumulative from day 1**, not incremental.

## calc() returns (key additions)
```js
pn1D, pn2D, pn3D       // monthly dividend arrays per partner
pn1CumD, pn2CumD, pn3CumD  // cumulative per partner
grossD, costsD          // monthly revenue and total costs arrays
```

## Devaluation model
Monthly FX = `P.fx × (1 + devalMonthly/100)^m`. ARS costs get cheaper in USD as peso weakens. USD revenues unaffected. `fxRatio = P.fx / fxM` applied to wages, rent, etc.

## Supabase
- Auto-saves on every `saveLS()` call with 1.5s debounce (`sbSave`)
- Load priority on startup: **localStorage → Supabase → scenarios.json fallback**
- Anon key embedded in file

## Deploy
```bash
cd F:\codetests\cleanup\canchitas
git add index.html
git commit -m "..."
git push origin main
# Pages live in ~2 min
```

---

## Next items (agreed / in progress)
1. **Consolidate to one chart** — drop `ch` (line chart), fold ganancia mensual + S&P + acumulado as optional line datasets onto `ch2` (bar chart) with visibility toggles. User confirmed one chart is enough.
2. **Acumulado lines react to partner checkboxes** — show only visible partners.
3. UX: waterfall trigger labels could be clearer (both say "% inversión que recibe Maxi" — user was confused).

---

## UI sections (left panel accordions)
⚽ Canchas fútbol · 🎓 Escuelita · 🏓 Paddle · 🍔 Otros ingresos · 💰 Inversión · 💳 Financiamiento · 🔧 CAPEX · 👥 Empleados · 📊 Gastos fijos · ⚙️ Parámetros · 📉 Devaluación · 🏦 Waterfall dividendos · 🔍 Reality Check

## UI sections (right panel)
KPI pills (kRow2 + kRow) · Line chart (ch) · Seasonality strip · OCC draggable curve · Risk text · Gauge + pie · Partner checkboxes (partVis) · Partner cards (prow) · Bar chart toggles (chartVis2) · Bar chart (ch2)
