# LaborLens Analytics

Independent research project analyzing the labor market impact of AI adoption
within NAICS 54 (Professional, Scientific, and Technical Services) using
IPUMS CPS ASEC microdata (2020–2025) and Census BTOS subsector data.

This project provides granular subsector evidence linking firm-level AI adoption
rates to worker wage outcomes in NAICS 54, with equity analysis across gender
and race — a combination not present in existing public research.

---

## Research Questions

### Core question
> To what extent does a sector's AI exposure intensity correlate with shifts in
> its employment growth, entry-level wages, and wage equity within NAICS 54
> over the past five years?

---

### Theme 1 — Post-2022 DiD: Did generative AI restructure wages in NAICS 54?

Most DiD studies on AI and wages use broader industry cuts or pre-2022 data.
This project tests the wage impact of the ChatGPT-era AI adoption surge
specifically within professional services.

1. Did the wage premium for AI-exposed occupations in NAICS 54 accelerate
   following the mass adoption of generative AI tools post-2022, relative to
   non-AI occupations in the same sector?

2. Does the post-2022 wage divergence between AI-exposed and non-AI workers
   in NAICS 54 exceed what would be predicted by pre-existing human capital
   differences alone?

3. Has the employment share of AI-exposed occupations within NAICS 54 grown
   disproportionately in the post-ChatGPT period compared to the 2020–2022
   baseline?

---

### Theme 2 — Equity: Does AI exposure narrow or widen wage gaps?

Gender and race wage gaps *inside* AI-exposed roles are underexplored.
Most equity research examines tech broadly — not professional services specifically.

1. Do gender wage gaps within AI-exposed occupations in NAICS 54 differ
   significantly from gender wage gaps in non-AI occupations in the same
   sector — and has this gap narrowed post-2022?

2. Does AI occupational exposure provide differential wage benefits across
   racial groups within NAICS 54, and if so, which groups see the largest
   earnings lift?

3. To what extent does AI exposure moderate the traditional education-wage
   relationship differently across gender and racial groups in professional
   services?

---

### Theme 3 — Entry-level wages: Does AI accelerate early-career earnings?

The experience-wage curve assumes earnings rise gradually with tenure.
AI exposure may compress or extend this curve for young workers — a question
with direct implications for workforce development and education policy.

1. Do entry-level workers (age ≤30) in AI-exposed occupations in NAICS 54
   earn significantly more than their non-AI peers at the same career stage?

2. Has the entry-level AI wage premium grown post-2022, suggesting generative
   AI tools are differentially rewarding younger workers earlier in their careers?

3. Does the entry-level AI wage premium compound with education — do young
   workers with college degrees in AI roles earn disproportionately more than
   young workers without?

---

## Data Sources

| Source | Coverage | Key Variables |
|--------|----------|---------------|
| IPUMS CPS ASEC | 2020–2025, NAICS 54 | EARNWEEK, OCC, IND, AGE, SEX, RACE, EDUC, WTFINL |
| Census BTOS Subsector | Mid-2025–Mid-2026 | AI adoption % by subsector (Q7: current use, Q24: planned) |

NAICS 54 subsectors covered (IPUMS IND codes 7270–7490):

| IND | NAICS | Subsector |
|-----|-------|-----------|
| 7270 | 5411 | Legal services |
| 7280 | 5412 | Accounting/tax |
| 7290 | 5413 | Architecture/engineering |
| 7370 | 5414 | Specialized design |
| 7380 | 5415 | Computer systems/IT |
| 7390 | 5416 | Management consulting |
| 7460 | 5417 | Scientific R&D |
| 7470 | 5418 | Advertising/marketing |
| 7480 | 5419 | Other professional services |

---

## Methods

| Model | Question | Specification |
|-------|----------|---------------|
| Mincer OLS | AI wage premium controlling for human capital | `ln(EARNWEEK) ~ AI_FLAG + EDUC_YRS + AGE + AGE² + C(YEAR)` |
| Difference-in-Differences | Post-2022 wage restructuring | `ln(EARNWEEK) ~ TREAT + POST + TREAT×POST + controls` |
| Quantile regression | Does AI benefit high vs low earners? | Q10–Q90 AI coefficients |
| Employment share trend | Are AI roles growing in NAICS 54? | Weighted headcount by year |
| Equity analysis | Gender/race wage gaps within AI roles | Grouped median earnings + OLS interactions |
| Entry-level analysis | Early-career AI wage premium | Age ≤30 subsample + DiD |

All regressions use HC3 heteroskedasticity-robust standard errors.
Person weights (ASECWT) used for population-representative employment counts.

---

## Occupational Exposure Groups

AI-exposed (Stanford AI Index high-intensity):

- Software Developers, Engineers, QA (OCC 1006, 1007, 1022)
- Computer Programmers (OCC 1021)
- Web Developers (OCC 1010)
- Cybersecurity Analysts (OCC 1031)
- Database Administrators (OCC 1065)
- Network/Systems Analysts (OCC 1032, 1105, 1108)
- Computer/IS Researchers (OCC 1005)
- IT Support Specialists (OCC 1050)

Comparison group (NAICS 54, lower AI exposure):

- Lawyers, Paralegals (OCC 2100, 2145)
- Accountants/Auditors (OCC 710)
- Architects, Civil Engineers (OCC 1305, 1360)
- Chief Executives, IS Managers (OCC 10, 110)

---

## Key Findings (preliminary)

- AI-exposed workers earn **+13.2% wage premium** in NAICS 54 (p<0.001)
- AI-exposed occupation share grew **48.4% → 54.3%** of NAICS 54 (2020–2023)
- Post-2022 DiD estimate: **-5.5%** — inconclusive (p=0.42, ns) — needs longer post window
- Gender gap **narrows inside AI roles**: $100/wk vs $404/wk in non-AI roles
- Black workers see **+$207/wk** lift from AI exposure — largest racial benefit
- Asian workers command highest AI-exposed median: **$1,683/wk**

---

## Repo Structure

```
LaborLens/
├── README.md
├── requirements.txt
├── src/
│   ├── ipums_pipeline.py      # IPUMS CPS ingestion, cleaning, OCC mapping
│   ├── btos_pipeline.py       # BTOS AI adoption pipeline
│   └── econometrics.py        # LaborLensEconometrics — 4 econometric models
├── notebooks/
│   └── labor_lens_analysis.py # Orchestration + full analysis
├── outputs/                   # Generated plots
└── data/
    └── .gitkeep               # Raw data not committed
```

---

## Setup

```bash
pip install -r requirements.txt
```

Add to `data/`:
- `CPS - 2020-2025.gz` — IPUMS CPS ASEC extract (ipums.org)
- `Subsector.xlsx` — Census BTOS subsector data (census.gov/hfp/btos)

---

## Run

```python
from src.ipums_pipeline import IPUMSPipeline
from src.btos_pipeline import BTOSPipeline
from src.econometrics import LaborLensEconometrics

cps = IPUMSPipeline("data/CPS - 2020-2025.gz")
cps.run()

btos = BTOSPipeline("data/Subsector.xlsx")
btos.run()

econ = LaborLensEconometrics(cps.df_occ)
econ.run_all()
econ.plot_all(save_path="outputs/econometric_summary.png")
```

---

## Limitations

- CPS ASEC sample size after NAICS 54 + occupation filter: n≈964 — limits statistical power
- No geographic variables in current extract — state/metro analysis pending
- BTOS × CPS cross-reference pending — firm AI adoption rate not yet linked to worker outcomes
- TFP analysis requires external BLS/BEA productivity data — not in current pipeline
- Post-2022 DiD window short — 2024/2025 ASEC releases will strengthen identification

---

## Author

Lead Data Scientist — LaborLens Analytics
