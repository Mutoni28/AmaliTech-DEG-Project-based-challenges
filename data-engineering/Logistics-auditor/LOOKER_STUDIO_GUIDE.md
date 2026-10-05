# Looker Studio Dashboard: Build Guide

The dashboard uses `looker_data.csv`, a smaller copy of the notebook output with one row per delivered order (10 columns, without the written review comments). This guide turns it into a public dashboard in about 45 minutes.

## 1. Connect the data

1. Go to [lookerstudio.google.com](https://lookerstudio.google.com) → **Create → Report**.
2. Upload `looker_data.csv` to Google Drive, open it with Google Sheets, then in Data Studio add it with the **Google Sheets** connector. (The CSV File Upload connector does not load data for viewers who are not logged in, so Google Sheets is used instead.)
3. Check these field types in the data source:
   - `order_purchase_timestamp` → **Date & Time**
   - `state_iso` → **Geo → Country subdivision (1st level)** (values look like `BR-SP`)
   - `Is_Late`, `review_score`, `Days_Difference`, `km` → **Number**

## 2. Calculated fields

| Field name | Formula | Format |
|---|---|---|
| Late Rate | `AVG(Is_Late)` | Percent |
| Avg Review | `AVG(review_score)` | Number, 2 decimals |
| Orders | `COUNT(order_id)` | Number |

Every row in the file is a delivered order, so the late rate is simply the average of `Is_Late`.

## 3. Filters (top of the page)

Add drop-down controls for `region` and `customer_state`, and a date range control on `order_purchase_timestamp`.

## 4. Charts

**Overview:**
- Scorecards for Orders, Late Rate and Avg Review.
- A donut chart of `Delivery_Status` by record count.

**Where are deliveries late? (Story 3)**
- **Filled map:** Location = `state_iso`, Color = Late Rate, region = Brazil.
- **Bar chart:** Dimension = `customer_state`, Metric = Late Rate, sorted descending.
- **Bar chart:** Dimension = `region`, Metric = Late Rate.

**Do late deliveries cause bad reviews? (Story 4)**
- **Column chart:** Dimension = `Delivery_Status`, Metric = Avg Review.
- **Line chart:** Dimension = `Days_Difference` (set to Number and sort ascending; add a filter to keep values between −30 and 30), Metric = Avg Review.

**Who caused the delay? (Candidate's Choice)**
- **Pie chart:** Dimension = `Delay_Owner`, Metric = Record Count, filter `Is_Late = 1`.
- **Stacked bar:** Dimension = `customer_state`, Breakdown = `Delay_Owner`, filter `Is_Late = 1`.

## 5. Publish publicly

1. **Share → Manage access → Link settings → Unlisted** (anyone with the link can view) or **Public**.
2. Open the link in an incognito window. It must load without logging in.
3. Paste the link into the README and the submission form.
