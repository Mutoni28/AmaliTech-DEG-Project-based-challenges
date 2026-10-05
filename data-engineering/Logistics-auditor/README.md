# The "Last Mile" Logistics Auditor

## A. Executive Summary

About 7 out of every 100 delivered orders (6.8%) arrived later than promised, and late orders get much worse reviews: 4.29 stars when on time, 2.99 when up to 5 days late, and only 1.74 when more than 5 days late. The problem is not the same everywhere. The North-East is late 12.7% of the time, about twice as often as the South and South-East, and Alagoas (21%), Maranhão (17%) and Rio de Janeiro (12%) are among the worst. Distance from the seller is not the reason. The real reason is that some regions get a much smaller safety margin in their delivery estimates. Most late orders (72%) left the seller on time, so the delay happens during shipping. The first things to fix are the delivery estimates and the carriers in the North-East and Rio.

## B. Project Links

| What | Link |
|---|---|
| Notebook (Google Colab) | https://colab.research.google.com/drive/1Xhn9CpqoYDHuFfHYPIuzodWXiwTwPTNy?usp=sharing |
| Dashboard (Looker Studio / Data Studio) | https://datastudio.google.com/reporting/a00dd5c9-fb87-48ff-a723-1734748dcabd/page/XmVAG |
| Presentation | https://drive.google.com/file/d/13pSOZ19j99Nn7AYbyueSC40jt0w1z6Jg/view?usp=sharing |

This repo also has the notebook file (`last_mile_logistics_auditor.ipynb`), a PDF copy of it with all the charts (`last_mile_logistics_auditor.pdf`), and the slides in the `presentation` folder.

## C. Technical Explanation

### How I cleaned the data

**No duplicate orders.** Some orders have two or three reviews. If I joined the tables as they are, those orders would show up more than once. So I kept only the newest review for each order before joining. After joining, the notebook checks for duplicates, and there are none.

**How late was each order?** I subtracted the actual delivery date from the promised date to get `Days_Difference`. A positive number means the order came early, and a negative number means it came late. I only compared the dates, not the times. Otherwise, a parcel that arrived in the evening of the promised day would wrongly count as one day late.

Each order then gets a status:
- **On Time:** arrived on or before the promised day
- **Late:** 1 to 5 days late
- **Super Late:** more than 5 days late

**Orders that never arrived.** Some orders were canceled, unavailable or still on the way, so there's no delivery date to measure. I marked these 2,971 orders as "Unknown" and left them out of the delay results.

**Product categories.** Categories are in Portuguese, so I translated them using the translation file that comes with the data. Two categories were missing from that file, so I added them myself. I only compared categories with at least 500 orders, so very small categories don't give misleading results.

**Remote states.** I didn't want to guess which states are "remote", so I measured the distance between each seller and customer. It turned out that distance doesn't explain late deliveries. The far-North states (Amazonas, Amapá, Rondônia, Acre) are the furthest away but are rarely late, because their orders usually arrive about 20 days before the promised date. North-East states only get about 9 to 11 days of extra time, so any delay makes them late.

### My extra analysis (Candidate's Choice): was the seller or the carrier late?

A delivery has two parts. First, the seller hands the parcel to the carrier, and the seller has a deadline for this. Then the carrier brings it to the customer. For every late order, I checked whether the seller missed its deadline. If it did, I counted the delay as the seller's. If not, the carrier caused it.

**Why this matters:** knowing *where* orders are late is useful, but Veridi also needs to know *who* to fix, because each problem needs a different solution. The result: 72% of late orders were handed to the carrier on time, and only 28% started with a late seller. So the biggest improvement will come from better carriers and routes in the worst regions, with stricter seller deadlines as a second step.

### Bonus: English categories

All categories are shown in English. The type of product matters much less than where it's going. The ten worst categories are all between 7% and 8% late, close to the 6.8% average, but states range from 3% to 21%. Furniture is not harder to deliver on time than electronics (Furniture Decor 7.1%, Electronics 7.6%).

## What's in this repo

```
├── README.md
├── last_mile_logistics_auditor.ipynb      # the notebook
├── last_mile_logistics_auditor.pdf        # PDF copy of the notebook with all charts
├── presentation/
│   ├── last_mile_audit_presentation.pptx
│   └── last_mile_audit_presentation.pdf
├── LOOKER_STUDIO_GUIDE.md                 # how I built the dashboard
├── requirements.txt
└── .gitignore                             # keeps the raw data files out of the repo
```

## How to run it

1. Download the [Olist dataset from Kaggle](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce).
2. Put the CSV files in the same folder as the notebook. In Colab, upload them using the Files panel on the left.
3. Run the notebook. It creates `logistics_master.csv` (the full cleaned data) and `looker_data.csv` (a smaller copy used for the dashboard, loaded into a Google Sheet).

The raw data files are not uploaded to this repo.

---

# Project Brief: The "Last Mile" Logistics Auditor

**Client:** Veridi Logistics (Global E-Commerce Aggregator)
**Deliverable:** Public Dashboard, Code Notebook & Insight Presentation

---

## 1. Business Context

**Veridi Logistics** manages shipping for thousands of online sellers. Recently, the CEO has noticed a spike in negative customer reviews. She has a "gut feeling" that the problem isn't just that packages are late, but that the estimated delivery dates provided to customers are wildly inaccurate (i.e., we are over-promising and under-delivering).

She needs you to audit the delivery data to find the root cause. She specifically wants to know: **"Are we failing specific regions, or is this a nationwide problem?"**

Your job is to build a "Delivery Performance" audit tool that connects the dots between **Logistics Data** (when a package arrived) and **Customer Sentiment** (how they rated the experience).

## 2. The Data

You will use the **Olist E-Commerce Dataset**, a real commercial dataset from a Brazilian marketplace. This is a relational database dump, meaning the data is split across multiple CSV files.

- **Source:** [Kaggle - Olist Brazilian E-Commerce Dataset](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)
- **Key Files to Use:**
  - `olist_orders_dataset.csv` (The central table)
  - `olist_order_reviews_dataset.csv` (Sentiment)
  - `olist_customers_dataset.csv` (Location)
  - `olist_products_dataset.csv` (Categories)

## 3. Tooling Requirements

You have the flexibility to choose your development environment:

- **Option A (Recommended):** Use a cloud-hosted notebook like **Google Colab**, or **Deepnote**, etc.
- **Option B:** Use a local **Jupyter Notebook** or **VS Code**.
  - _Condition:_ If you choose this, you must ensure your code is reproducible. Do not reference local file paths (e.g., `C:/Downloads/...`). Assume the dataset is in the same folder as your notebook.
- **Dashboarding:** The final output must be a **publicly accessible link** (e.g., Tableau Public, Google Looker Studio, Streamlit Cloud, or PowerBI Web, etc.).

---

## 4. User Stories & Acceptance Criteria

### Story 1: The Schema Builder

**As a** Data Engineer,
**I want** to join the Orders, Reviews, and Customers tables into a single master dataset,
**So that** I can analyze a customer's location and their review score in the same row.

- **Acceptance Criteria:**
  - Load the raw CSVs into your notebook.
  - Perform the correct joins (e.g., join Reviews to Orders on `order_id`, join Customers to Orders on `customer_id`).
  - **Check:** Ensure you don't accidentally duplicate rows (a common error with 1-to-many joins).

### Story 2: The "Real" Delay Calculator

**As a** Logistics Manager,
**I want** to know the difference between the "Estimated Delivery Date" and the "Actual Delivery Date,"
**So that** I can see how often we are lying to customers.

- **Acceptance Criteria:**
  - Create a new calculated column: `Days_Difference` = `order_estimated_delivery_date` - `order_delivered_customer_date`.
  - Classify orders into statuses: "On Time", "Late", and "Super Late" (> 5 days late).
  - Handle missing values: Some orders were never delivered (`order_status` = 'canceled' or 'unavailable'). These should be excluded or flagged separately.

### Story 3: The Geographic Heatmap

**As a** Regional Director,
**I want** to see which specific States (`customer_state`) have the highest percentage of late deliveries,
**So that** I can focus my repair efforts on the worst regions.

- **Acceptance Criteria:**
  - Calculate the % of late orders per State.
  - Visualize this on a map or a bar chart.
  - **Insight:** Identify if "Remote" states (far from the distribution center) are disproportionately affected.

### Story 4: The Sentiment Correlation

**As a** Customer Success Lead,
**I want** to see if late deliveries actually cause bad reviews,
**So that** I can prove to the CEO that logistics is the problem.

- **Acceptance Criteria:**
  - Create a visualization comparing "Delivery Delay (Days)" vs "Average Review Score (1-5)".
  - Show the average review score for "On Time" orders vs. "Late" orders.

---

## 5. Bonus User Story: The "Translation" Challenge

**As a** Global Analyst,
**I want** to see product categories in **English**, not Portuguese,
**So that** I can understand if "Furniture" is harder to ship than "Electronics".

- **Acceptance Criteria:**
  - The `product_category_name` is in Portuguese (e.g., `cama_mesa_banho`).
  - Use the `product_category_name_translation.csv` file included in the dataset (or create your own mapping) to translate these into English for your final dashboard.

---

## 6. The "Candidate's Choice" Challenge

**As a** Creative Problem Solver,
**I want** to include one extra feature or analysis that adds specific business value,
**So that** I can demonstrate my ability to think beyond the basic requirements.

- **Instructions:**
  - Add one more metric, chart, or drill-down.
  - **Requirement:** You must justify _why_ this feature matters to the business in your README.

---

## 7. Submission Guidelines

Please edit this `README.md` file in your forked repository to include the following three sections at the top:

### A. The Executive Summary

- A 3-5 sentence summary of your findings.

### B. Project Links

- **Link to Notebook:** (e.g., Google Colab, etc.). _Ensure sharing permissions are set to "Anyone with the link can view"._
- **Link to Dashboard:** (e.g., Tableau Public, etc.).
- **Link to Presentation:** A link to a short slide deck (PDF/PPT) AND (Optional) a 2-minute video walkthrough (YouTube) explaining your results.

### C. Technical Explanation

- Briefly explain how you handled the "Data Cleaning".
- Explain your "Candidate's Choice" addition.

**Important Note on Code Submission:**

- Upload your `.ipynb` notebook file to the repo.
- **Crucial:** Also upload an **HTML or PDF export** of your notebook so we can see your charts even if GitHub fails to render the notebook code.
- Once you are ready, please fill out the [Official Submission Form Here](https://forms.cloud.microsoft/e/CeQN2mCyUr) with your links

---

## 🛑 CRITICAL: Pre-Submission Checklist

**Before you submit your form, you MUST complete this checklist.**

> ⚠️ **WARNING:** If you miss any of these items, your submission will be flagged as "Incomplete" and you will **NOT** be invited to an interview.
>
> **We do not accept "permission error" excuses. Test your links in Incognito Mode.**

### 1. Repository & Code Checks

- [ ] **My GitHub Repo is Public.** (Open the link in a Private/Incognito window to verify).
- [ ] **I have uploaded the `.ipynb` notebook file.**
- [ ] **I have ALSO uploaded an HTML or PDF export** of the notebook.
- [ ] **I have NOT uploaded the massive raw dataset.** (Use `.gitignore` or just don't commit the CSV).
- [ ] **My code uses Relative Paths.**

### 2. Deliverable Checks

- [ ] **My Dashboard link is publicly accessible.** (No login required).
- [ ] **My Presentation link is publicly accessible.** (Permissions set to "Anyone with the link can view").
- [ ] **I have updated this `README.md` file** with my Executive Summary and technical notes.

### 3. Completeness

- [ ] I have completed **User Stories 1-4**.
- [ ] I have completed the **"Candidate's Choice"** challenge and explained it in the README.

**✅ Only when you have checked every box above, proceed to the submission form.**

---
