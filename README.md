# FlowTwin — business process simulator

**Live demo → https://navidnajmabadi.github.io/FlowTwin/**

FlowTwin is a digital twin of how a business runs. Draw the business as a workflow, n8n-style: work arrives from **inputs**, **departments** do the work, and it ends at **outcomes**. Then press **Run simulation**. A Monte Carlo engine plays out 200 possible years in about a second and shows you:

- which department is the **bottleneck**, and how many leads it loses to missed deadlines
- **who to hire**: the FTE each team needs to run at a healthy 80% load
- **margin, cost and revenue per employee**, each as a P10–P90 range, not a single guess
- **arrival-to-cash cycle time** and queue build-up across the year
- **AI or humans?**: a score for each task that says automate, augment with a copilot, or keep human, and how much capacity that frees
- **scenarios and sensitivity**: +30% demand, recession, supply shock, quality crisis, AI plan or extra hires, compared side by side

Every chart has a `?` explaining what it shows and a one-sentence insight underneath.

## The model

| Element | Attributes |
|---|---|
| **Input** (demand source) | arrivals / month, monthly volatility, seasonality + peak month, annual growth, average contract value + spread |
| **Department** | manager, roles (FTE × salary), hours/day, productive %, max jobs in parallel (WIP), other direct cost / month, weekend work, *external partner* flag (subtrades / suppliers: capacity and per-job cost, no payroll, not counted as employees) |
| **Task** (inside a department) | effort min / most likely / max (hours), waiting time after work, rework %, material cost per job, *invoice on completion* (% of job value — deposits and progress draws), queue deadline, supplier-dependent, AI profile (repetitive, judgement, digital data, client-facing, physical), AI mode |
| **Line** (handoff) | *after task* → *next task*, routing weight %, type (normal / win / loss / rework), priority at destination, handoff cost, transfer delay, business importance |
| **Outcome** | revenue (with % recognised) or lost |

Lines connect a *finished task* to the *next task*, so one department can appear at several stages. For example, Sales qualifies a tender, Engineering costs it, and it returns to Sales to submit and negotiate.

**Engine:** a daily discrete-time simulation over 1,095 days: 730 days of warm-up, so long projects reach steady state, then a 365-day measured year. Arrivals are Poisson, effort is triangular, and contract values are log-normal. Each team's capacity is FTE × hours × productive % × (1 − absenteeism), shared across up to WIP parallel jobs. Queues are served by line priority, then first in, first out.

## Workspaces and sharing

- Each tab is its own workspace. The Pionova and Atlas Cranes samples open side by side as tabs, next to an empty workspace; **+** adds more.
- **Share** copies a link that holds the whole workspace, compressed into the URL fragment (`#m=…`). Opening the link loads it into a new tab. Nothing is uploaded to a server.
- Models can also be exported and imported as JSON.

## Run locally

It is one static file with no build step:

```bash
python3 -m http.server 8910
```

Then open http://localhost:8910. Opening `index.html` directly also works.

## Samples

**Pionova Construction Management (GTA)** is the default sample. It models an Ontario construction manager with a CEO (Nima), 3 project managers, a 2-person social media team, 1 accountant, 2 heads of site, and external subtrades and material suppliers. It runs three streams:
- **Commercial tenders** (Bids&Tenders, MERX, GCs): takeoff → CEO go/no-go → bid → CCDC contract and bonding → pre-con and City permit → materials → mobilization and OHSA safety plan → structure → progress draw #1 → supervision and municipal inspections (with a rework loop) → MEP and finishes → progress draw #2 at substantial performance → deficiencies → holdback release after 60 days (Construction Act).
- **Residential renovations** from social media and referral leads: qualify → site visit → proposal → contract and 25% deposit → permit → materials → reno trades → handover → final invoice.
- **Change orders**: price → CEO approval → invoice.

**Atlas Cranes** is a fictional crane manufacturer sample. It covers tender → engineering validation → negotiation → contract → design → planning → fabrication → assembly and load test → commissioning → invoicing, plus aftermarket service. It opens in its own tab next to Pionova; the house and crane icons on the left rail reopen either sample.

**All figures in both samples are illustrative assumptions for a demo, not real company data.**
