# Impact of Global Crises on Financial Markets

**DSA 210 — Introduction to Data Science · Sabancı University · Spring 2026**
**Author:** Yağmur Çaloğlu

This project investigates whether six global crises spanning 2000–2024 (9/11, the Iraq War, the 2008 Global Financial Crisis, COVID-19, the Russia–Ukraine invasion, and the Israel–Hamas conflict) produced statistically detectable shifts in financial market returns and sectoral behavior — and whether those shifts are predictable. The analysis combines event-study hypothesis testing with classification and clustering models, and is framed as an operational test of the Efficient Market Hypothesis. Warning: If GitHub fails to render the notebooks, please open the HTML versions included in the repository.

## Key Findings

- **5 of 6 crises produced statistically significant shifts** in S&P 500 returns under Interrupted Time Series (ITS) regression with HAC standard errors. Russia–Ukraine was the only exception — markets had priced in the geopolitical risk before the invasion.
- **Mann–Whitney U and Kruskal–Wallis failed to detect any crisis effect** — but this reflects insufficient statistical power (small windows against fat-tailed daily returns), not absence of effect. ITS, which models the pre-crisis trend, is the more appropriate method for this context.
- **Classification models perform near chance** — Logistic Regression AUC = 0.516, Random Forest AUC = 0.507. The `is_crisis` feature has near-zero importance. This supports the weak-form Efficient Market Hypothesis: crisis information is already absorbed into volatility and sector behavior.
- **K-Means identifies three persistent regimes** (Calm, Turbulent, single-day Crisis outlier). The ±60-day regime distribution around each crisis reveals structural differences: the 2008 GFC has 43% turbulent days clustering *after* the event, while Russia–Ukraine had turbulence elevated *before* the invasion.
- **Sector behavior is more predictable than the aggregate index.** Defense gains in geopolitical events, energy collapses in financial crises, aviation suffers most in health crises, and gold maintains near-zero correlation with the S&P 500 across the full sample.

The complete write-up is in [`DSA210_Final_Report.pdf`](./DSA210_Final_Report.pdf).

## Repository Structure

```
DSA_210_TermProject/
├── data_collection.ipynb           # Step 1: Pull prices from yfinance + macro from FRED
├── eda.ipynb                       # Step 2: Exploratory data analysis, correlations, sector behavior
├── hypothesis_testing.ipynb        # Step 3: Mann–Whitney U, Kruskal–Wallis, ITS regression
├── ml.ipynb                        # Step 4: Logistic Regression, Random Forest, K-Means
├── data/
│   ├── prices.csv                  # Daily closing prices, 10 assets, 2000–2024 (1.2 MB)
│   ├── returns.csv                 # Daily percentage returns (1.3 MB)
│   ├── macro.csv                   # Fed Rate, CPI, Unemployment, M2 (FRED)
│   └── *_(YYYY)_pre.csv,           # Per-crisis ±30-day windows
│       *_(YYYY)_post.csv             (12 files: 6 crises × pre/post)
├── DSA_210_Term_Project_Proposal.pdf
├── DSA210_Final_Report.pdf
├── requirements.txt
└── README.md
```

## Data

The dataset combines two public sources:

- **yfinance (Yahoo Finance API)** — daily closing prices for nine assets: S&P 500, VIX, WTI Crude Oil, Gold, and five sector ETFs (Defense, Energy, Healthcare, Technology, Aviation). The aviation series is a composite of Boeing (BA, pre-2007) and Delta Airlines (DAL, post-2007) because JETS only began trading in 2015.
- **FRED (Federal Reserve Economic Data)** — Federal Funds Rate, CPI, Unemployment Rate, and M2 money supply. MSCI World was added as a global benchmark.

The merged dataset covers **2000-01-03 to 2024-12-31**, yielding **6,292 daily observations across 10 asset series**.

## Reproducing the Analysis

Requires Python 3.9+ and a free FRED API key (https://fred.stlouisfed.org/docs/api/api_key.html).

```bash
# 1. Clone the repository
git clone https://github.com/yagmurcaloglu-design/DSA_210_TermProject.git
cd DSA_210_TermProject

# 2. Create and activate a virtual environment
python -m venv .venv
source .venv/bin/activate          # macOS / Linux
# .venv\Scripts\activate            # Windows

# 3. Install dependencies
pip install -r requirements.txt

# 4. (Optional) Set your FRED API key as an environment variable
export FRED_API_KEY="your_key_here"

# 5. Run the notebooks in order
jupyter notebook
# Open and run: data_collection.ipynb → eda.ipynb → hypothesis_testing.ipynb → ml.ipynb
```

The notebooks should be run in the order listed. `data_collection.ipynb` populates `data/`; the rest read from there.

## Methods at a Glance

| Stage | Notebook | Key techniques |
|---|---|---|
| Data collection | `data_collection.ipynb` | `yfinance` API, FRED API, per-crisis window slicing |
| Exploratory analysis | `eda.ipynb` | Distributional checks, correlation matrix, sector return comparison, MSCI World benchmark |
| Hypothesis testing | `hypothesis_testing.ipynb` | Mann–Whitney U, Kruskal–Wallis, Interrupted Time Series with Newey-West (HAC) standard errors |
| Machine learning | `ml.ipynb` | Logistic Regression, Random Forest Classifier, K-Means Clustering with elbow method, ±60-day regime distribution analysis |

## AI Assistance Disclosure

This project was completed with the assistance of Claude (Anthropic, claude.ai).

**Tool used:** Claude (claude.ai), Anthropic.

**What AI was used for:**
- Coding assistance — debugging the JETS ETF NaN issue (3,858 missing values) and constructing the BA+DAL composite aviation series to recover pre-2015 coverage.
- Methodological guidance — justifying Mann–Whitney U and Interrupted Time Series over t-tests given the fat-tailed return distributions; motivating Newey-West (HAC) standard errors from the Durbin–Watson diagnostics (0.21–0.77 indicating strong autocorrelation).
- Structuring the ML notebook (logistic regression → random forest → K-Means flow), interpreting near-zero `is_crisis` feature importance as evidence for the Efficient Market Hypothesis, and motivating the ±60-day regime-distribution analysis after the K-Means "Crisis" cluster collapsed to a single outlier day (April 20, 2020 oil-futures collapse).
- Writing assistance for the final report.

**Conversation logs (full prompts and outputs):**
- [Data Collection, Hypothesis testing and EDA notebook](https://claude.ai/chat/4e7c06c0-c418-4bde-b165-d27ec41b4162)
- [ML follow-up: ±60-day regime distribution](https://claude.ai/chat/15ecd8dd-b08d-46e5-b692-f588d8788472)
- [Final report & README session](https://claude.ai/chat/9976d698-fce3-477c-a617-5251cacf8ebe)

**Example prompts:**
- "JETS ticker'ında 3858 NaN var, ne yapayım?" → suggested combining BA + DAL to extend historical coverage.
- "MWU testleri hep insignificant çıkıyor, hipotezim yanlış mı?" → diagnosed as a power problem, recommended ITS with HAC standard errors.
- "K-Means crisis cluster'ında tek gün var, ne demek bu?" → identified as the April 20, 2020 oil-futures anomaly; motivated the ±60-day regime-distribution analysis as a more robust view.

All analytical decisions, data interpretation, model selection, and final conclusions are the author's own. AI was used as a methodological consultant and coding assistant, not as a substitute for original analysis.

## References

- Yahoo Finance API via the `yfinance` Python package
- Federal Reserve Economic Data (FRED), St. Louis Fed
- Course materials: DSA 210, Sabancı University, Spring 2026

