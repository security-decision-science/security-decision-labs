# FAIR Risk Quantification Tool

A cyber risk quantification tool based on the FAIR (Factor Analysis of Information Risk) methodology.

- React + TypeScript frontend (Vite)
- Python FastAPI backend for Monte Carlo simulations
- IRIS 2025 benchmarks for LEF (frequency) and LM (loss magnitude)
- UI designed in Figma, implemented with shadcn/ui components

See 
- [docs/MODEL_REFERENCE.md](docs/MODEL_REFERENCE.md) for the math details.  
- [docs/LIMITATIONS.md](docs/LIMITATIONS.md) for scope and limitations.

[![License: CC BY-NC-SA 4.0](https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by-nc-sa/4.0/)
[![FAIR Methodology](https://img.shields.io/badge/Methodology-FAIR-blue.svg)](https://www.fairinstitute.org/)

**Important:** This is an independent implementation of the FAIR methodology. See [FAIR_NOTICE.md](FAIR_NOTICE.md) for trademark information, data sources, and attributions.

**Non-Commercial License:** This tool is for educational and research purposes only.
See [LICENSE](LICENSE) for details.

---

## Getting Started

For detailed setup instructions, see **[SETUP.md](SETUP.md)**.

**Quick overview:**
- **Prerequisites:** Node.js 18+, Python 3.9+
- **Backend:** `cd risk_service && pip install -r requirements.txt && python fair_risk_engine.py`
- **Frontend:** `npm install && npm start`
- **Backend URL:** `http://localhost:8000` (configure via `.env` if needed)
- **Frontend URL:** `http://localhost:5173`

---
## Project Structure

```text
/
├── risk_service/               # Python FastAPI backend
│   ├── fair_risk_engine.py     # Main API server
│   ├── fair_calculator.py      # Monte Carlo engine
│   ├── fair_aggregation.py     # Portfolio aggregation
│   ├── fair_distributions.py   # Statistical distributions & fitting
│   ├── benchmark_library.py    # IRIS 2025 benchmark helpers
│   └── requirements.txt        # Python dependencies
│
├── docs/                       # Model reference and limitations
├── src/                        # Frontend source (Vite)
│   ├── main.tsx                # React entry point
│   ├── App.tsx                 # Main app component
│   ├── components/             # UI components
│   │   ├── Dashboard.tsx
│   │   ├── ScenarioCreation.tsx
│   │   ├── ScenarioDetail.tsx
│   │   ├── ScenarioList.tsx
│   │   ├── SensitivityAnalysis.tsx
│   │   └── ui/…                # Shared UI primitives
│   ├── hooks/
│   │   └── useFairCalculation.ts
│   ├── utils/
│   │   └── fairApi.ts          # Typed client for FastAPI backend
│   ├── config/
│   │   └── iris2025_benchmarks.ts
│   ├── styles/
│   │   └── globals.css         # Tailwind v4 base + theme tokens
│   └── vite-env.d.ts           # Vite/TS environment typing
│
├── shared/
│   └── iris2025_benchmarks.json
│
├── start.sh                    # Convenience script (optional)
├── package.json
├── vite.config.ts
├── tsconfig.json
├── tsconfig.node.json
├── README.md
└── SETUP.md
```

(Some filenames may differ slightly depending on the version and local changes, but the overall layout should be as above.)

---

## Key Features

### FAIR Methodology

- **LEF** = Threat Event Frequency (TEF) × Susceptibility
- **LM** (Loss Magnitude) uses the **6 FAIR loss forms**:
  - Productivity
  - Response
  - Replacement
  - Fines & Judgments
  - Competitive Advantage
  - Reputation
- **ALE** (Annual Loss Exposure) = LEF × LM

### Monte Carlo Engine (Backend)

- 100k+ simulations per scenario
- Poisson / zero-inflated Poisson / lognormal for frequency
- Beta-PERT for probabilities (Susceptibility & SLEF)
- Lognormal + zero-inflated modeling for loss forms where P10 = 0
- Sensitivity analysis and portfolio aggregation

### Benchmarks

- IRIS 2025 LEF benchmarks (annual probability of loss event):
  - Endpoint: `GET /api/benchmarks/lef?industry=…&revenue=…`
- IRIS 2025 LM benchmarks (loss magnitude):
  - Endpoint: `GET /api/benchmarks/lm?industry=…&revenue=…`

These are surfaced in the UI as hints when you pick an industry + revenue tier in the scenario wizard.

---

## Backend API Overview

### Health

- `GET /`  
  Basic health check.

### Risk Calculation

- `POST /calculate`  
  Run Monte Carlo simulation for a single scenario.  
  Returns ALE, LEF, LM and loss-form breakdowns.

- `POST /sensitivity`  
  One-factor-at-a-time (±% around baseline) sensitivity on ALE.

- `POST /aggregate`  
  Aggregate multiple scenarios into a portfolio, with optional correlation.

- `POST /portfolio/metrics`  
  Portfolio metrics via linearity of expectation (total ALE, weighted LM, etc.).

- `POST /validate`  
  Validate FAIR inputs without running the full simulation.

### Benchmarks (IRIS 2025)

- `GET /api/benchmarks/lef`  
  Query params: `industry`, `revenue` (both optional).

- `GET /api/benchmarks/lm`  
  Query params: `industry`, `revenue` (both optional).

Full interactive docs (with schemas and try-it-out): `http://localhost:8000/docs`.

---

## Development Commands

From the project root:

```bash
# Start frontend dev server (Vite, port 5173)
npm start

# Build frontend for production
npm run build

# Preview production build
npm run preview
```

Backend:

```bash
cd risk_service

# Start risk_service (simple)
python fair_risk_engine.py

# or with uvicorn (recommended for dev)
uvicorn api:app --reload --host 0.0.0.0 --port 8000
```

---

## Production Deployment

This tool is not intended for commercial or production use. See [docs/LIMITATIONS.md](docs/LIMITATIONS.md) for details.

For theoretical production deployment guidance, see the **Production Deployment** section in [SETUP.md](SETUP.md).

---

## Troubleshooting

**Frontend can't reach backend:**
- Confirm backend is running: `curl http://localhost:8000/`
- Check `.env` contains a valid `VITE_API_URL`
- Restart Vite after changing `.env`

**CORS errors:**
- Backend includes permissive CORS for dev. For production, narrow `allow_origins` to your real frontend origin(s).

For more troubleshooting help, see the **Common Issues & Fixes** section in [SETUP.md](SETUP.md).
