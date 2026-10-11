# Sarthi IAS: Growth and Retention Analytics for a UPSC Edtech Platform

**Excel + Power Query + Power BI (DAX) | Business Analyst portfolio project**

> **Note:** Sarthi IAS is a **fictional company** and all data is **synthetic**. I generated a realistic dataset to practise an end-to-end business analytics workflow: problem definition, data checks, modelling, KPIs, dashboard and recommendations.

---

## 1. Business problem

Sarthi IAS sells online and offline coaching for the UPSC Civil Services Exam (CSE) through nine marketing channels. Management sees a lot of leads, but:

- only about **7 in 100 leads** become students,
- about **a third of closed enrollments** end in dropout or refund,
- discounts are used heavily, but nobody knows if they help or hurt.

**Goal:** find where students are lost, which channels and courses actually pay back, and what to change before the next UPSC cycle.

### Questions answered

1. Which channels give the best conversion, CAC and lifetime value, not just the most leads?
2. Do bigger discounts bring more revenue or just students who drop out?
3. At which point in a course do students drop out, and does delivery mode matter?
4. Do working professionals need a different offer?
5. Does engagement (tests, scores) relate to clearing Prelims?
6. Which support issues are slowest?
7. How much revenue comes from repeat buyers?

---

## 2. Dataset

One Excel workbook, `Sarthi_IAS_Edtech_Dataset.xlsx`, with nine linked tables (period: Jan 2024 to Oct 2026).

| Table | Rows | Description |
|---|---|---|
| Leads | 20,000 | Channel, campaign, state, funnel stage, converted flag |
| Students | 1,374 | Demographics, background, attempt number, optional subject, device |
| Courses | 20 | Foundation, Prelims, Mains, Test Series, Optional and more |
| Instructors | 10 | Specialization, experience, rating |
| Enrollments | 2,151 | Discount, amount paid, refund, status, completion %, dropout week |
| Marketing_Spend | 297 | Monthly spend by channel |
| Engagement | 8,221 | Monthly logins, video hours, tests, scores |
| Support_Tickets | 559 | Category, resolution hours, CSAT |
| Exam_Outcomes | 746 | Prelims and Mains results, 2024 to 2026 cycles |

Built-in business patterns: Prelims-driven seasonality, channel quality differences, discount-dropout relationship, delivery mode effects, refund timing.

---

## 3. Data model (Power BI)

Star schema with **Enrollments** as the central fact table, plus **DimDate** and **DimChannel** created in Power BI.

```
Instructors --< Courses --< Enrollments >-- Students >-- DimChannel
                                 |              |
                      Engagement / Tickets    Leads, Exam_Outcomes
                                 |
                              DimDate  (also linked to Leads, Marketing_Spend, Students, Tickets, Engagement via inactive relationships + USERELATIONSHIP)
```

- One-to-many relationships, single-direction filters
- Foreign keys hidden in report view
- Date table marked as date table
- All measures kept in a separate `_Measures` table

---

## 4. Process

| Step | Tool | What I did |
|---|---|---|
| 1. Define the problem | Docs | Problem statement, 7 business questions, KPI list |
| 2. Data checks | Excel / Power Query | Duplicate keys, orphan keys, date logic, amount logic, status logic |
| 3. Exploratory analysis | Excel pivots | Funnel by channel, revenue by category, discount vs status, dropout by mode |
| 4. Modelling | Power BI | Star schema, DimDate, DimChannel, relationships |
| 5. KPIs | DAX | Conversion, CAC, ARPU, LTV, dropout %, refund %, CSAT and more |
| 6. Dashboard | Power BI | 5 interactive pages |
| 7. Insights | Power BI / write-up | Findings and recommendations |

### Key DAX measures

```DAX
Lead Conversion % = DIVIDE([Converted Leads], [Total Leads])
CAC = DIVIDE([Marketing Spend], [New Students])
Net Revenue = [Gross Revenue] - [Refunds]
Dropout Rate % =
DIVIDE(
    CALCULATE(COUNTROWS(Enrollments), Enrollments[enrollment_status] = "Dropped"),
    CALCULATE(COUNTROWS(Enrollments),
        Enrollments[enrollment_status] IN {"Completed","Dropped","Refunded"})
)
LTV to CAC = DIVIDE([LTV], [CAC])
```

A cumulative funnel is used (Leads, Demo or beyond, Counselling or beyond, Enrolled) because `funnel_stage` records only the furthest stage reached.

---

## 5. Dashboard pages

| Page | Focus |
|---|---|
| 1. Overview | Students, enrollments, courses, instructors, tickets, enrollment trend |
| 2. Marketing and Funnel | Spend, leads, cost per lead, CAC, funnel, campaign and channel performance |
| 3. Revenue | Gross and net revenue, revenue per enrollment, discount, category revenue, course status, payment plans |
| 4. Exams and Support | Prelims and Mains outcomes, ticket categories, resolution time, optional subject demand |
| 5. Engagement | Video hours, logins, tests, doubt sessions |

Screenshots are in the `05_screenshots/` folder.

---

## 6. Key results

| Metric | Value |
|---|---|
| Leads to students | 20,000 to 1,374 (**6.9%** conversion) |
| Funnel (cumulative) | 20,000 leads, 8,300 demo or beyond, 3,970 counselling or beyond, 1,374 enrolled |
| Net revenue | about **₹3.95 crore** (₹39.5M) |
| Net revenue per enrollment | about ₹18.3K |
| Marketing spend | about ₹45.9 lakh |
| Cost per lead / CAC | about ₹229 / about ₹3,340 |
| Dropout rate (of closed enrollments) | about **34%** |
| Average ticket resolution | 23.2 hours |

---

## 7. Insights and recommendations

1. **Judge channels by cost and conversion together.** Referral, Free Webinar and Toppers Talk Events convert far better than paid social (roughly 11 to 19% vs 2 to 5%), but events cost the most per lead. Shift budget toward referral and webinar programmes and measure LTV to CAC before scaling events.
2. **Deep discounts are linked to higher dropout.** Dropout rises from about 22% with no discount to about 41% at 25%. Cap discounts at 15% for long courses and use scholarships tied to test performance instead.
3. **Unserved optional demand.** Students choose Political Science and IR (about 180) and Anthropology (about 105) as optionals, but no course is offered for either. Launch one as a pilot.
4. **Follow the exam calendar.** Test series and crash courses peak from December to March, while Foundation and Mains peak from June to August. Align campaigns and faculty planning to these windows.
5. **Retention levers.** Recorded courses and working professionals drop out more; add live doubt sessions, weekend batches and early nudges in the first weeks.

*(Confirm every number against your final dashboard before publishing.)*

---

## 8. Limitations

- Data is synthetic; exam outcomes are random draws loosely linked to test scores, not real UPSC statistics.
- Revenue is recognised at enrollment; EMI instalments are not scheduled.
- Monthly CAC is slightly noisy because students sign up up to 20 days after their lead date.
- GST is excluded.

---

## 9. Repository structure

```
Sarthi-IAS-Analytics/
├── 01_data/            Sarthi_IAS_Edtech_Dataset.xlsx
├── 02_excel/           Sarthi_IAS_Excel_Analysis.xlsx
├── 03_powerbi/         Sarthi_IAS_Dashboard.pbix
├── 04_docs/            Project_Case_Study.pdf, Data_Dictionary.md
├── 05_screenshots/     page1_overview.png ... page5_engagement.png
└── README.md
```

## 10. How to use

1. Download the repository.
2. Open `Sarthi_IAS_Dashboard.pbix` in Power BI Desktop. If asked, point the data source to `01_data/Sarthi_IAS_Edtech_Dataset.xlsx`.
3. Use the Year slicer and page navigation to explore.

---

## 11. About me

**Manthan Khairnar**, aspiring Data and Business Analyst (Pune, India)

- Portfolio: https://manthankhairnar.github.io/Manthan_Khairnar_Portfolio/
- GitHub: https://github.com/manthankhairnar
- More projects:
  - [Data Analysis Project](https://github.com/manthankhairnar/Data_Analysis_Project)
  - [Food Delivery E-commerce Analysis](https://github.com/manthankhairnar/Food_Delevary_E-commerce_Analysis)
  - [Pizzahut Sales Analysis (SQL)](https://github.com/manthankhairnar/Pizzahut_sales_analysis_SQL)
  - [Stock Market Analysis (26-06-2026)](https://github.com/manthankhairnar/Stock-market-analysis-on-26-06-2026)
