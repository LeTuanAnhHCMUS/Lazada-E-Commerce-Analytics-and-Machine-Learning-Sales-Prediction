# Lazada Marketplace Analytics

### Pricing Strategy · Seller Performance · Sales Forecasting

[![Python](https://img.shields.io/badge/Python-3.x-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![Pandas](https://img.shields.io/badge/Pandas-data%20analysis-150458?logo=pandas&logoColor=white)](https://pandas.pydata.org/)
[![Scikit--learn](https://img.shields.io/badge/scikit--learn-modeling-F7931E?logo=scikitlearn&logoColor=white)](https://scikit-learn.org/)
[![License](https://img.shields.io/badge/License-not%20specified-lightgrey)](#license)

This notebook-based project explores Lazada product and seller listings to support practical decisions on discounting, category strategy, and marketplace performance. It combines scraped product/shop data and MongoDB aggregations with exploratory visual analysis and a Random Forest sales-volume model. The focus is not only on predicting sales, but on translating observed patterns into pricing and seller-management insights.

> **Scope note:** Findings below are descriptive analyses of the collected marketplace snapshot. They do not establish that changing a discount, rating, or seller attribute will cause the reported sales or revenue change.

## Key Business Insights

### Pricing and discounting

The pricing analysis in [`tesstloc.ipynb`](notebooks/tesstloc.ipynb) groups products into five discount bands and calculates mean estimated revenue as `price × units sold`.

| Discount band | Product count | Mean estimated revenue (notebook-reported VND) |
|---|---:|---:|
| 0–10% | 2,296 | 263,974 |
| 10–20% | 631 | 146,417 |
| 20–30% | 1,042 | 101,281 |
| 30–50% | 2,086 | 88,920 |
| >50% | 966 | 56,328 |

- **0–10% is the highest-revenue band in this snapshot**, with mean estimated revenue of 263,974 VND per product. The notebook describes this as its discount “sweet spot”; it is an observational comparison, not a causal optimum.
- **Deep discounts performed poorly for the premium subset.** For products with an original price above 500,000 VND, the notebook compares discounts above 30% with discounts of 30% or less. The deep-discount group had 1,044 products and mean estimated revenue of 137,738 VND, versus 1,196 products and 562,539 VND for the lower-discount group — about **75.51% lower mean revenue**.
- In that same premium subset, the >30% discount group contained fewer products in the analyzed sample than the ≤30% group (1,044 versus 1,196). These values are cohort product counts, not units sold.
- The discount/units-sold correlation reported in the notebook is **0.0136**, effectively close to zero in this dataset; it should not be interpreted as evidence of a reliable sales lift from discounts.

The deep-discount result is specific to the notebook’s **original-price >500,000 VND** cohort and should not be generalized to every product or category.

### Category trends and price sensitivity

The dataset covers nine product categories. One saved category-count output ranks **Men’s Apparel (2,043 products)** and **Women’s Apparel (1,889)** as the largest groups, followed by Skin Care (669), Laptops (477), and Beauty Tools (433). Another notebook reports 7,014 products, while the category-count output sums to 7,021; treat category and product totals as snapshot-specific until the source data is refreshed and reconciled.

[`Loi.ipynb`](notebooks/Loi.ipynb) compares average units sold at **0% discount** and **20–30% discount**:

- **Makeup** and **Men’s Apparel** show the strongest reported responses to 20–30% discounts: +516 (+269%) and +186 (+137%), respectively.
- **Skin Care** increases by +612 (+84%) in the same comparison.
- **Beauty Tools** moves in the opposite direction: the notebook reports about 2,100 average units sold without a discount, then a decline of 1,525 (-64%) in the 20–30% group.
- **Perfume** is nearly unchanged (-8 units, -2%) in the comparison.

These are group-level comparisons, not controlled experiments; differences in product mix and seller composition may also contribute.

### Ratings, customer trust, and seller tiers

The customer-review analysis in [`Bao.ipynb`](notebooks/Bao.ipynb) reports:

- Products with **more than 1,000 reviews account for nearly 40% of analyzed revenue**; products with **200 or more reviews account for 65%**.
- Average revenue and the review-to-purchase ratio peak in the **4.8–4.9 rating band** in the notebook’s analysis. The notebook also reports lower average revenue below 4.5 stars. These are associations in the analyzed sample, not proof that soliciting reviews alone increases revenue.

The seller analysis in [`TAnh.ipynb`](notebooks/TAnh.ipynb) segments shops by follower count. The saved output shows a strongly skewed distribution:

| Seller follower tier | Mean revenue (notebook units) | Median revenue (notebook units) |
|---|---:|---:|
| <1k followers | 73,819 | 7,115 |
| 1k–10k followers | 232,055 | 23,380 |
| >10k followers | 2,787,784 | 100,543 |

The largest tier’s mean is far above its median, indicating that a small number of high-revenue observations influence the average. In a separate category comparison, median revenue is higher for LazMall in **Skin Care** (481,850 vs. 13,721 for regular shops) and **Laptops** (194,332 vs. 86,250), while regular shops have higher medians in **Men’s Apparel** (14,700 vs. 12,036) and **Women’s Apparel** (8,888 vs. 7,434). The category-level sample sizes differ substantially, so interpret these as exploratory comparisons, not a like-for-like seller effect.

## Data Pipeline

```text
Lazada listing pages
        │
        ▼
Selenium + Undetected Chromedriver crawl notebooks
        │
        ├── Raw product CSVs ──┐
        └── Raw shop CSVs ─────┤
                               ▼
                    Cleaning and feature engineering
                               │
                               ├── Cleaned CSV snapshots
                               └── MongoDB Atlas collections
                                    (fact_products, dim_shops)
                                               │
                                               ▼
                              EDA, aggregation, and charts
                                               │
                                               ▼
                              Sales model and what-if analysis
```

1. **Ingestion:** Four notebooks in [`notebooks/crawl_data/`](notebooks/crawl_data/) crawl men’s apparel, women’s apparel, electronics, and beauty listings with Selenium and Undetected Chromedriver. No separate API-based ingestion script is present in the repository.
2. **Preprocessing:** The notebooks in [`notebooks/preprocessing/`](notebooks/preprocessing/) and [`notebooks/preprocess and eda/`](notebooks/preprocess%20and%20eda/) clean product and shop fields, handle missing values, derive numeric features, and create analysis-ready CSVs. The modeling pipeline uses `SimpleImputer`, `OneHotEncoder`, and `StandardScaler`.
3. **Storage and aggregation:** Analysis notebooks query MongoDB Atlas collections named `fact_products` and `dim_shops`; local raw and cleaned CSV snapshots are also included.
4. **EDA:** Product and shop notebooks examine distributions, category composition, reviews, discounts, seller characteristics, and revenue using statistical summaries and visualizations.
5. **Modeling:** A held-out test set is used to compare a linear baseline with a tuned Random Forest, followed by a discount what-if simulation.

## Machine Learning and Evaluation

The model notebook predicts product **units sold** (`Da_ban`) using `Gia_goc`, `Giam_gia`, `Danh_muc`, `Diem_danh_gia`, `Loai_cua_hang`, `Followers`, `Danh_gia_nguoi_ban`, `So_nam_tham_gia`, `Ty_le_phan_hoi_chat`, and `Giao_dung_hen`. It uses an 80/20 train-test split, a `log1p` target transformation, and `RandomizedSearchCV` to tune a `RandomForestRegressor`; a multiple linear regression model is also evaluated.

| Model | R² (log target) | R² (original scale) | MAE | RMSE |
|---|---:|---:|---:|---:|
| Tuned Random Forest (log target) | 0.5626 | 0.0131 | 594.84 | 5,960.26 |
| Multiple Linear Regression (log target) | 0.2067 | -0.0077 | 636.81 | 6,022.81 |

The Random Forest performs better than the linear baseline, but its **original-scale R² of 0.0131 is weak**. The log-scale R² is not a substitute for original-scale fit; the notebook attributes the gap to highly skewed sales and large outliers. The reported absolute-error distribution is **41.72% below 20 units**, **30.41% from 20 to 100 units**, and **27.87% above 100 units**. Thus, 72.13% of the evaluated predictions have absolute error below 100 units; this is not a per-category guarantee.

### What-if scenario

The notebook simulates a **5-percentage-point increase in the discount feature** by adding `0.05` to the current discount rate (not a 5% decrease in selling price). Its saved aggregate prediction is:

| Scenario | Predicted total units sold |
|---|---:|
| Current inputs | 804,384 |
| Discount increased by 5 percentage points | 802,747 |
| Difference | -1,636 (-0.20%) |

The notebook calculates the difference from unrounded predictions and rounds the totals independently, so the displayed totals differ by one unit from the reported rounded delta.

The scenario is model-based and inherits the model’s limitations; it is not a measured experiment or a revenue forecast.

## Technology Stack

| Area | Technologies |
|---|---|
| Language and notebooks | Python, Jupyter |
| Data processing | Pandas, NumPy |
| Machine learning | Scikit-learn (pipelines, preprocessing, Random Forest, Linear Regression, `RandomizedSearchCV`), SciPy |
| Visualization | Matplotlib, Seaborn |
| Collection and storage | Selenium, Undetected Chromedriver, Beautiful Soup, WebDriver Manager, PyMongo, MongoDB Atlas |

## Repository Structure

```text
Lab1_DV/
├── README.md
├── requirements.txt
├── .gitignore
├── docs/
│   └── Lab 1 .pdf
├── data/
│   ├── raw/
│   │   ├── product/
│   │   │   ├── lazada_trang_phuc_nam.csv
│   │   │   ├── lazada_trang_phuc_nu.csv
│   │   │   ├── lazada_thietbidientu.csv
│   │   │   └── lazada_suc_khoe_lam_dep.csv
│   │   └── shop/
│   │       ├── lazada_shop_profiles_tbdtu.csv
│   │       ├── lazada_shop_profiles_final_tpnu.csv
│   │       ├── lazada_shop_profiles_final_tpnam.csv
│   │       └── lazada_shop_profiles_final_skldep.csv
│   ├── preprocessed/
│   │   ├── database.ipynb
│   │   ├── sanpham_cleaned.csv
│   │   └── shop_cleaned.csv
│   └── preprocess and eda/
│       ├── sanpham_cleaned.csv
│       └── shop_cleaned.csv
└── notebooks/
    ├── Bao.ipynb
    ├── Loi.ipynb
    ├── TAnh.ipynb
    ├── locc.ipynb
    ├── tesstloc.ipynb
    ├── preprocessing/
    │   ├── preprocess_sanpham.ipynb
    │   └── preprocess_shop.ipynb
    ├── preprocess and eda/
    │   ├── preprocess_eda_product.ipynb
    │   └── preprocess_eda_shop.ipynb
    └── crawl_data/
        ├── 01_Crawl_Lazada_TrangPhucNam.ipynb
        ├── 02_Crawl_Lazada_TrangPhucNu.ipynb
        ├── 03_Crawl_Lazada_ThietBiDienTu.ipynb
        └── 04_Crawl_Lazada_MyPham.ipynb
```

The repository is notebook-first; no standalone `.py` analysis scripts were found. Charts and some result tables are embedded in notebook outputs rather than saved as separate image files.

## Reproduction Guide

### 1. Clone and enter the project

```bash
git clone <repository-url>
cd <repository-directory>
```

If the repository contains this project in a `Lab1_DV/` subfolder, change into that folder before continuing.

### 2. Create an environment and install dependencies

Python 3.10 or later is recommended. Most dependencies are unpinned; Scikit-learn is set to version 1.4 or later because the notebooks use `root_mean_squared_error`.

```bash
python -m venv .venv
```

Activate it:

```bash
# Windows PowerShell
.\.venv\Scripts\Activate.ps1

# macOS/Linux
source .venv/bin/activate
```

Install the packages:

```bash
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

### 3. Prepare data access

- Existing CSV snapshots are under [`data/`](data/). Use these for local inspection and to avoid recrawling.
- Several analysis notebooks connect to MongoDB Atlas and expect the `ecom_data_lab` database with `fact_products` and `dim_shops` collections. Configure an authorized connection string before running those cells; do not commit credentials.
- Crawling requires Chrome/Chromium and a compatible driver. Respect Lazada’s terms, robots guidance, and rate limits; the crawler notebooks may encounter CAPTCHAs or layout changes.

### 4. Launch Jupyter and run notebooks

```bash
jupyter lab
```

Suggested workflow:

1. For fresh collection, run the relevant notebook in [`notebooks/crawl_data/`](notebooks/crawl_data/).
2. For cleaning, run the product/shop notebooks in [`notebooks/preprocessing/`](notebooks/preprocessing/) or [`notebooks/preprocess and eda/`](notebooks/preprocess%20and%20eda/).
3. Explore the analysis and visualizations in [`Bao.ipynb`](notebooks/Bao.ipynb), [`Loi.ipynb`](notebooks/Loi.ipynb), [`TAnh.ipynb`](notebooks/TAnh.ipynb), [`locc.ipynb`](notebooks/locc.ipynb), and [`tesstloc.ipynb`](notebooks/tesstloc.ipynb).

Notebook outputs are saved in the `.ipynb` files, but rerunning the MongoDB-backed cells requires a working database connection and compatible source data. There are no standalone script entry points or automated test suite in the current tree.

## License

No `LICENSE` file or license declaration was found in the repository. Unless a license is added, reuse and redistribution permissions are not specified.
