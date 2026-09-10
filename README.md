# Predict-Daily-Sales
A client-side ML dashboard that forecasts daily sales using Linear Regression and a from-scratch Random Forest, with trend analysis and PDF report export — all running in the browser, no backend required.
# Predict Daily Sales

A browser-based sales forecasting dashboard that trains machine learning models **entirely client-side** — no backend, no external API calls — to predict daily sales, analyze historical trends, and generate exportable PDF reports.

The app loads a historical sales CSV, engineers time-series features, trains a Linear Regression model and a Random Forest from scratch in TypeScript, and picks whichever performs best on a held-out test set.

## Features

- **Dashboard** — At-a-glance KPIs (total sales, average daily sales, customers, promotions/holidays) plus trend and category charts for the full dataset.
- **Daily Sales Predictor** — Enter a date, product category, price, quantity, promotion/holiday flags, and season to get a predicted sales figure with a confidence interval, plus a rolling 7-day forecast.
- **Sales Analysis** — Daily/monthly trends, category and seasonal breakdowns, promotion vs. non-promotion and holiday vs. non-holiday comparisons, and weekday performance.
- **Model Performance** — Side-by-side comparison of Linear Regression vs. Random Forest: MAE, MSE, RMSE, R², actual-vs-predicted charts, residual plots, and feature importance.
- **PDF Report Export** — One-click export of stats, model metrics, and current predictions/forecasts to a downloadable PDF (via `jspdf` and `html2canvas`).
- **Auto model selection** — The pipeline automatically picks the model with the lowest test-set RMSE for live predictions.

## How the ML pipeline works

Everything below runs in the browser, in plain TypeScript — there is no Python backend and no server-side training.

1. **Preprocessing** (`src/ml/preprocessing.ts`) — Parses the raw CSV (via PapaParse), drops invalid/empty rows, coerces types, and normalizes boolean fields like `promotion` and `holiday`.
2. **Feature engineering**:
   - **Date features** (`dateFeatures.ts`) — year, month, day, day of week, weekend flag, quarter, week of year.
   - **Lag features** (`lagFeatures.ts`) — sales from 1, 7, and 30 records prior, computed strictly from past observations to avoid leakage.
   - **Rolling features** (`rollingFeatures.ts`) — 7/14/30-period trailing moving averages of sales.
   - **Categorical encoding** (`encoding.ts`) — one-hot encoding for `product_category` and `season`.
3. **Chronological train/test split** — an 80/20 split preserving time order (no shuffling), so the model is always tested on data that comes after what it trained on.
4. **Model training** (`modelTraining.ts`):
   - **Linear Regression** (`linearRegression.ts`) — solved via Gauss-Jordan elimination with partial pivoting on standardized features, with light regularization for numerical stability.
   - **Random Forest** (`randomForest.ts`) — an ensemble of 25 regression trees (max depth 7, min samples split 4) built from scratch, with variance-reduction splits and aggregated feature importance.
5. **Evaluation** (`utils/metrics.ts`) — MAE, MSE, RMSE, R², and residual statistics on the held-out test set for both models.
6. **Inference** — for a new prediction, lag/rolling features are recomputed dynamically from the historical dataset, and the prediction interval is derived from the selected model's test-set RMSE/MAE.

## Dataset

Bundled at `public/data/daily_sales.csv`: **608 daily records** spanning **2024-01-01 to 2025-08-31**.

| Column | Description |
|---|---|
| `sales_id` | Unique record identifier |
| `date` | Calendar date of the record |
| `product_category` | One of: Apparel, Electronics, Health & Beauty, Home & Kitchen, Sports & Outdoors |
| `quantity` | Units sold |
| `price` | Unit price |
| `previous_sales` | Sales figure from the prior record |
| `promotion` | Whether a promotion was running (Yes/No) |
| `holiday` | Whether the date was a holiday (Yes/No) |
| `customers` | Number of customers |
| `season` | Winter, Spring, Summer, or Fall |
| `sales` | Target variable — total sales for the day |

You can swap in your own CSV as long as it includes at least `date` and `sales` columns; the loader (`src/services/datasetService.ts`) validates and cleans the data on load.

## Tech stack

- **React 19** + **TypeScript**
- **Vite 6** — build tooling and dev server
- **Tailwind CSS 4** — styling
- **Recharts** — charts and data visualization
- **PapaParse** — CSV parsing
- **jsPDF** + **html2canvas** — PDF report generation
- **lucide-react** — icons
- **motion** — animations

> Note: the project scaffold includes a `@google/genai` dependency and `GEMINI_API_KEY` environment variable (left over from its AI Studio template), but the current feature set — data loading, feature engineering, model training, and predictions — runs entirely client-side and does **not** call the Gemini API. No API key is required to run the app as-is.

## Project structure

```
├── public/
│   └── data/daily_sales.csv     # Default sales dataset
├── src/
│   ├── ml/                      # Feature engineering & models
│   │   ├── preprocessing.ts
│   │   ├── dateFeatures.ts
│   │   ├── lagFeatures.ts
│   │   ├── rollingFeatures.ts
│   │   ├── encoding.ts
│   │   ├── linearRegression.ts
│   │   ├── randomForest.ts
│   │   └── modelTraining.ts
│   ├── services/datasetService.ts
│   ├── utils/                   # Metrics, forecasting, insights, PDF export
│   ├── components/              # Charts, cards, forms, tables
│   ├── pages/                   # Dashboard, Predictor, Analysis, Model Performance
│   ├── types.ts
│   └── App.tsx
└── package.json
```

## Getting started

**Prerequisites:** Node.js (18+ recommended)

```bash
# Install dependencies
npm install

# Start the dev server (http://localhost:3000)
npm run dev

# Type-check
npm run lint

# Build for production
npm run build

# Preview the production build
npm run preview
```

## License

No license file is currently included — add one (e.g. MIT) if you plan to share or accept contributions on this repo.
