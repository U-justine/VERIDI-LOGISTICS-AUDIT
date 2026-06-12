# VERIDI-LOGISTICS-AUDIT
Data analysis project on last-mile logistics performance, focusing on delivery delays, customer behavior, and order fulfillment efficiency

## A. Executive Summary

Analysis of 96,000+ delivered orders reveals that remote Northern states (RR, AM, AP) suffer 22–38% late delivery rates vs. ~5% in São Paulo. Each additional day late reduces average customer review score by approximately 0.4 points. The root cause is not operational failure but **systematic over-promising**: estimated delivery dates for remote regions are 5–7 days too optimistic relative to actual carrier performance. On-time orders average 4.31★ while super-late orders average 2.51★ — a statistically significant difference (Kruskal-Wallis p < 0.001). Certain product categories (office furniture, large appliances) show structurally higher late rates regardless of destination. **Recommendation:** Adjust ETA algorithms to add a +3–5 day buffer for North/Northeast regions, and renegotiate carrier SLAs in the five worst-performing states.

---

## B. Project Links

| Deliverable | Link |
|---|---|
| 📓 Notebook (Google Colab) | *(paste your Colab link here)* |
| 📊 Dashboard | *(paste your Streamlit / Looker Studio / Tableau Public link here)* |
| 🎞 Presentation (Slides) | *(paste your Google Slides / PDF link here)* |
| 🎥 Video Walkthrough (optional) | *(paste YouTube link here)* |


---

## C. Technical Explanation

**Data Cleaning:** Removed canceled/unavailable orders from delay analysis (tagged as "Not Delivered" rather than dropped). Deduped reviews by keeping the most recent per `order_id` to prevent row explosion on join. Filtered out null delivery dates. Used `LEFT JOIN` from the orders table throughout so no orders are silently lost. Confirmed row counts with `assert len(Master) == len(orders)` after every join.

**Sign convention note (Day_Difference):** Computed as `estimated − actual`, so positive = early arrival, negative = late. Classification (`>= 0` → On Time) is correct under this convention. The Kruskal-Wallis test confirms a statistically significant *difference in distributions* between delivery status groups — this shows strong correlation between delay and lower scores; causality should be validated with further controlled analysis.

**Candidate's Choice (DPRS):** A weighted composite score — `(1 − norm_late_rate) × 50 + (1 − norm_severity) × 30 + norm_review_score × 20` — that quantifies "promise reliability" per state on a 0–100 scale. Business value: gives regional directors a single prioritisation metric instead of three separate KPIs. A state with 10% late orders all 20 days overdue scores lower than one with 20% late orders all 1 day overdue, capturing the severity dimension the CEO cares about.

---

### Feature Engineering

| Feature | Formula | Purpose |
|---|---|---|
| `delivery_delay_days` | `estimated_date − actual_delivery_date` | Positive = early, Negative = late |
| `delivery_status` | Rule-based on `delivery_delay_days` | On Time / Late / Super Late / Not Delivered |
| `region_type` | Lookup by `customer_state` | Remote vs Core vs Other |
| `DPRS` (per state) | Weighted composite (see below) | Single KPI for promise reliability |

---

### Candidate's Choice — Delivery Promise Reliability Score (DPRS)

**What it is:** A composite 0–100 score computed per state, measuring how trustworthy Veridi's delivery promises are. It is calculated as:

```
DPRS = (1 − norm_late_rate)    × 50
     + (1 − norm_severity)     × 30
     + norm_avg_review_score   × 20
```

Where all components are MinMax-normalised to [0, 1] across all states.

**Why it matters to the business:**

The CEO's concern is specifically about *promise-keeping* — not just raw lateness. Two states could have identical late rates but very different customer experiences: one might have packages arriving 1 day late, the other 20 days late. DPRS captures this severity dimension. It also incorporates the customer's own perception (review score), making it a leading indicator of churn risk. Regional directors receive a single, comparable number that rolls up frequency of failure, severity of failure, and customer-perceived impact — allowing prioritised resource allocation without reading three separate reports.

---

## D. How to Run the Notebook

### Option A — Google Colab (Recommended)

1. Open the notebook link above
2. Go to **Runtime → Run All**
3. When prompted, upload your `kaggle.json` API token
4. All data downloads automatically; all charts render inline

### Option B — Local Jupyter

```bash
# 1. Clone the repo
git clone https://github.com/YOUR_USERNAME/veridi-logistics-audit.git
cd veridi-logistics-audit

# 2. Install dependencies
pip install -r requirements.txt

# 3. Place Olist CSVs in data/ folder
#    (download from https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)

# 4. Launch notebook
jupyter notebook veridi_logistics_audit.ipynb
```

### requirements.txt
```
pandas>=2.0
numpy>=1.24
matplotlib>=3.7
seaborn>=0.12
plotly>=5.15
kaleido>=0.2
scipy>=1.11
scikit-learn>=1.3
requests>=2.31
```


---

## E. Dataset Description

| File | Rows | Key Columns |
|---|---|---|
| `olist_orders_dataset.csv` | ~99,441 | `order_id`, `order_status`, estimated + actual delivery dates |
| `olist_order_reviews_dataset.csv` | ~100,000 | `order_id`, `review_score`, `review_comment_message` |
| `olist_customers_dataset.csv` | ~99,441 | `customer_id`, `customer_state`, `customer_city` |
| `olist_products_dataset.csv` | ~32,951 | `product_id`, `product_category_name` (Portuguese) |
| `olist_order_items_dataset.csv` | ~112,650 | `order_id`, `product_id`, pricing |
| `product_category_name_translation.csv` | 71 | PT → EN category name mapping |

Source: [Kaggle — Olist Brazilian E-Commerce Dataset](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)

---

## F. Submission Checklist

- [ ] GitHub repo is **Public** (verify in Incognito Mode)
- [x] `.ipynb` notebook uploaded to repo
- [x] HTML export of notebook uploaded (`veridi_notebook_complete.html`)
- [ ] Raw CSV data files **NOT** committed — add `data/` to `.gitignore` (file provided below)
- [x] All file paths in notebook are **relative** (`DATA_DIR = "data"`)
- [ ] Dashboard link is publicly accessible (deploy `veridi_dashboard.html` to Streamlit / Tableau Public)
- [ ] Presentation link is publicly accessible (create 3–5 slide PDF from Key Findings section)
- [x] README updated with Executive Summary, links placeholder, and technical notes
- [x] User Stories 1–4 completed
- [x] Bonus (Translation Challenge) completed
- [x] Candidate's Choice (DPRS) completed and justified in README

### ⚠️ Your 3 remaining actions before submission:
1. **Upload to GitHub** — push `.ipynb` + `veridi_notebook_complete.html` + this `README.md`
2. **Deploy dashboard** — open `veridi_dashboard.html` → upload to [Tableau Public](https://public.tableau.com) or host via [Streamlit Cloud](https://streamlit.io/cloud), paste the public URL above
3. **Create presentation** — 5 slides: Overview → Data & Method → Geographic Findings → Sentiment Findings → Recommendations. Export as PDF, share via Google Drive ("Anyone with link"), paste URL above

---

*Veridi Logistics Last Mile Delivery Audit · Olist Brazilian E-Commerce Dataset · Analysis by Justine Umutoni*
