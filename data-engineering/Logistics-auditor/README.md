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

