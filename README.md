# Mutual Fund Portfolio Analyzer API

A production-style REST API that analyzes mutual fund portfolios, computes diversification scores, detects overlap, and recommends smart fund switches — built with FastAPI, MySQL, and JWT authentication.

## Problem

Most retail investors in India hold 3–5 mutual funds believing they're diversified, but many funds hold the same underlying stocks. This API takes a list of funds, X-rays the true portfolio exposure, scores diversification on a 0–100 scale, and recommends swaps to reduce redundancy.

## Features

- **Portfolio X-Ray** — True sector exposure, duplicate holdings, and market cap breakdown across multiple funds
- **Diversification Score (0–100)** — Custom algorithm penalizing overlap, concentration, and sector dominance while rewarding category diversity
- **Smart Switch Recommendations** — Identifies the weakest fund in a portfolio and recommends better replacements
- **Risk-Based Fund Allocation** — Monthly SIP allocation split across funds by risk profile (low/medium/high)
- **Fund Overlap Analysis** — Weighted overlap between any two funds using the MIN-formula
- **Most Held Stocks & Sector Analysis** — Systemic concentration across the entire fund universe
- **JWT Authentication** — Signup, login, and bcrypt-hashed passwords
- **Saved Portfolios (Protected Routes)** — Users can save, retrieve, and delete portfolios

## Tech Stack

- **Backend:** FastAPI (Python 3.12)
- **Database:** MySQL 8.0 with SQLAlchemy ORM
- **Auth:** JWT tokens (python-jose), bcrypt password hashing
- **Validation:** Pydantic schemas
- **Docs:** Auto-generated interactive Swagger UI

## Data

45 top Indian equity mutual funds containing **3,421 stock holdings across 1,041 unique stocks** — spanning Large Cap, Mid Cap, Small Cap, Flexi Cap, and Index categories.

The dataset was scraped from Moneycontrol.com as part of a separate project: [Mutual Fund Portfolio Overlap Analyzer](https://github.com/DakshMalhotra256/Mutual-Fund-Portfolio-Overlap-Analyzer). That project uses Python + BeautifulSoup to extract holdings, perform data cleaning (handling hidden `display:none` rows, stripping `-\n` and `#\n` prefixes from stock names), and runs 19 analytical SQL queries on the resulting dataset.

## API Endpoints

### Funds
- `GET /api/funds/` — List all funds (filter by category, search by name)
- `GET /api/funds/{fund_id}` — Get fund details with all holdings

### Analysis
- `GET /api/analysis/overlap?fund1_id=X&fund2_id=Y` — Weighted overlap between two funds
- `GET /api/analysis/most-held-stocks` — Most widely held stocks across all funds
- `GET /api/analysis/sectors` — Sector concentration statistics

### Portfolio
- `POST /api/portfolio/xray` — Deep analysis of a multi-fund portfolio
- `POST /api/portfolio/score` — 0–100 diversification score with breakdown
- `POST /api/portfolio/smart-switch` — Swap recommendations to improve diversification
- `POST /api/portfolio/recommend` — Fund allocation by risk level
- `POST /api/portfolio/save` — Save a portfolio (protected)
- `GET /api/portfolio/my` — Retrieve saved portfolios (protected)
- `DELETE /api/portfolio/{portfolio_id}` — Delete a saved portfolio (protected)

### Auth
- `POST /api/auth/signup` — Create account, returns JWT token
- `POST /api/auth/login` — Login, returns JWT token

## Diversification Score Algorithm

The score starts at 100, with penalties and bonuses applied:

- **Overlap Penalty (0–30):** Average pairwise overlap across all funds
- **Concentration Penalty (0–25):** Top 5 stocks' combined weight in portfolio
- **Sector Penalty (0–25):** Most dominant sector's weight
- **Category Bonus (0–20):** Rewards spreading across Large / Mid / Small / Flexi Cap / Index

The final score is clamped between 0 and 100. A portfolio of 3 Large Cap funds typically scores in the 40s, while a mix of 1 Large + 1 Mid + 1 Small Cap scores in the 80s.

## Project Structure
mutual-fund-api/
├── app/
│   ├── main.py              # FastAPI app entry point
│   ├── database.py          # SQLAlchemy engine & session
│   ├── models.py            # ORM models (5 tables, with relationships)
│   ├── schemas.py           # Pydantic response schemas
│   ├── auth.py              # JWT utilities + password hashing
│   └── routers/
│       ├── funds.py         # Fund listing/search
│       ├── analysis.py      # Overlap & sector analysis
│       ├── portfolio.py     # X-Ray, Score, Smart Switch, Saved Portfolios
│       └── auth.py          # Signup/Login
├── data/
│   ├── holdings.csv         # Mutual fund holdings dataset
│   └── seed.py              # CSV → MySQL loader
├── requirements.txt
└── README.md

## Local Setup

```bash
# Clone the repo
git clone https://github.com/DakshMalhotra256/mutual-fund-api.git
cd mutual-fund-api

# Install dependencies
pip install -r requirements.txt

# Create MySQL database
mysql -u root -p
CREATE DATABASE mutual_fund_db;

# Update DATABASE_URL in app/database.py with your MySQL credentials

# Seed the database
python data/seed.py

# Run the server
python -m uvicorn app.main:app --reload --port 8080
```

Open `http://127.0.0.1:8080/docs` for the interactive API documentation.

## Author

**Daksh Malhotra** — B.Tech Engineering Physics, Delhi Technological University ('27)
[GitHub](https://github.com/DakshMalhotra256) · [LinkedIn](https://www.linkedin.com/in/daksh-malhotra-176a94274/) · [LeetCode](https://leetcode.com/u/Daksh_Malhotra/)