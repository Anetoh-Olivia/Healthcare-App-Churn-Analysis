# 🏥 Healthcare App Churn Analysis (SQL Project)

A full SQL analysis of 240,000 health app users, built to find out why 1 in 5 of them leave within six months.

---

Out of every 5 people who sign up for this app, 1 is gone within six months. Users who leave their auto-renew switched off are 54% more likely to churn than those who leave it on. And oddly, the users who log in the most are not the ones who stay. This project digs into 240,000 real user records using nothing but SQL to find out exactly who is leaving, and why.

---

## Table of Contents
1. [Business Problem and Project Overview](#1-business-problem-and-project-overview)
2. [Dataset Description](#2-dataset-description)
3. [Data Model and Table Relationships](#3-data-model-and-table-relationships)
4. [Tools and Technologies](#4-tools-and-technologies)
5. [Skills Explored](#5-skills-explored)
6. [Data Cleaning Process](#6-data-cleaning-process)
7. [Analysis Results (Q1 to Q7)](#7-analysis-results-q1-to-q7)
8. [Key Findings](#8-key-findings)
9. [Summary and Conclusion](#9-summary-and-conclusion)
10. [Recommendations](#10-recommendations)
11. [Caveats and Limitations](#11-caveats-and-limitations)
12. [How to Explore This Project](#12-how-to-explore-this-project)
13. [About Me and Contact](#13-about-me-and-contact)

---

## 1. Business Problem and Project Overview

A digital health and fitness app lets users track workouts, log their diet, monitor sleep, and chat with coaches, on a free, basic, or premium plan. Like any subscription business, the app only makes money while a user stays. The problem is that a large number of users stop using the app within six months of signing up, and before this project, nobody could say exactly who was leaving or why.

This project answers that with data. Using SQL alone, 240,000 user records were loaded, cleaned, organized into a proper database, and analyzed to answer seven specific questions:

1. How many users are we losing overall?
2. Does the subscription plan affect whether a user leaves?
3. Do some regions lose more users than others?
4. Does turning on auto-renewal make a user more likely to stay?
5. Are the users who leave less active in the app than the ones who stay?
6. Does the device a user is on (iPhone, Android, or web browser) make a difference?
7. Do users who finish more of their workouts stick around longer?

Every step, from loading the raw file to answering these seven questions, is documented below with real numbers and screenshots.

---

## 2. Dataset Description

The data came as one file, `health_app_churn_data.csv`, with **240,000 rows** (one row per user) and **22 columns**. Every user's demographics, subscription details, app activity, and outcome all sit together in this one wide table. This is called a **flat table**, and it's the starting point before any cleaning or organizing happens.



![Figure 1: The raw flat table](screenshots/figure-01-flat-table-preview.png)


*Figure 1: The raw data as it looked right after loading, with 240,000 rows confirmed.*

**Data Dictionary**

| Column | What it means |
|---|---|
| `user_id` | A unique number identifying each user |
| `age` | The user's age |
| `gender` | Male, Female, or Other |
| `region` | Where the user is based: US, EU, LATAM, SEA, or KR |
| `bmi` | The user's Body Mass Index |
| `plan_type` | The subscription tier: free, basic, or premium |
| `monthly_fee` | What the user pays each month |
| `discount_rate` | Any discount applied to the fee |
| `tenure_days` | How many days the account has existed |
| `auto_renew` | Whether the subscription automatically renews (yes/no) |
| `last_payment_success` | Whether the most recent payment went through (yes/no) |
| `weekly_sessions` | How many times a week the user opens the app |
| `avg_session_minutes` | The average length of each visit, in minutes |
| `workout_completion_rate` | The share of started workouts the user actually finished |
| `diet_log_adherence` | The share of days the user logged their food |
| `sleep_tracking_usage` | The share of nights the user tracked their sleep |
| `coaching_messages_per_week` | How many messages the user exchanged with a coach weekly |
| `community_posts_per_month` | How many times the user posted in the community each month |
| `device_type` | iOS, Android, or web only |
| `wearable_connected` | Whether the user has a wearable device linked (yes/no) |
| `push_enabled` | Whether the user has push notifications on (yes/no) |
| `churn_within_6m` | **The outcome:** did the user leave within 6 months? (yes/no) |

---

## 3. Data Model and Table Relationships

The flat table above was split into **four smaller, linked tables**, each covering one topic, like organizing one big messy folder into four clearly labeled ones:

| Table | What it holds |
|---|---|
| `users` | Who the user is: age, gender, region, BMI |
| `subscription` | Their plan, fees, discount, tenure, and auto-renew status |
| `engagement` | Their app activity: sessions, workouts, diet, sleep, device settings |
| `churn_status` | Whether they left (the outcome we care about) |

All four tables are linked by `user_id`, so a user's activity, subscription, and outcome can always be traced back to the same person. `users` sits at the center, and the other three tables connect to it.



![Figure 2: Data model diagram](screenshots/figure-02-data-model.png)


*Figure 2: How the four tables connect through `user_id`.*

Splitting the data this way keeps each fact in one clear place, prevents a record from ever pointing to a user that doesn't exist, and makes the whole database easier to work with and grow later.

---

## 4. Tools and Technologies

- **PostgreSQL** – the database where all the data lives and all the work was done
- **pgAdmin** – the tool used to write and run SQL queries
- **GitHub** – used to document and share this project

This is a **fully SQL project**. No Excel, no Python, no BI tool was used to produce any result in this report.

---

## 5. Skills Explored

| Skill | What it looked like in this project |
|---|---|
| **Data Exploration** | Counting users, checking uniqueness, and looking at how users are spread across plans, regions, devices, and outcomes |
| **Data Cleaning** | Finding missing values, filling them in properly, removing risk of duplicates, and checking that every column holds sensible values |
| **Data Transformation** | Splitting one flat table into four organized, linked tables with proper keys |
| **Data Extraction & Analysis** | Combining the four tables to answer real business questions and calculate churn rates |
| **Data Validation** | Double-checking that every table lined up correctly before trusting any result |
| **Query Optimization** | Speeding up how the database searches and joins large tables |

---

## 6. Data Cleaning Process

Raw data is never ready to analyze straight away. Before answering any business question, the data had to be set up, explored, cleaned, and reorganized. Skipping any of these steps would mean building answers on data that might be wrong, incomplete, or duplicated, so every step below exists for a reason.

### 6.1 Setting Up a Clean Workspace

A dedicated database was created for this project, and the raw file was loaded into a separate table called `health_app_raw`. This table was left completely untouched as a backup of the original data, so if anything ever went wrong later, the source data would still be safe to go back to.



![Figure 3: Database and raw table created](screenshots/figure-03-database-and-raw-table.png)


*Figure 3: The project database and the raw table, set up before any changes were made.*

### 6.2 Data Exploration

Exploration comes before cleaning, and it means **looking at the data to understand what it contains, without changing anything yet.** This step tells you what you're working with, so you know what to check more closely later.

**Confirming the size and uniqueness of the data.** The total number of rows was counted, and checked against the number of unique `user_id` values, to confirm that every row represents exactly one real user with no one counted twice.



![Figure 4: User count and uniqueness check](screenshots/figure-04-user-count-uniqueness.png)


*Figure 4: 240,000 rows and 240,000 unique user IDs, confirming one row per person.*

**Looking at how users are spread out.** Users were grouped by plan, region, and device type, to get a first picture of who makes up the user base.



![Figure 5: User breakdown by plan, region, and device](screenshots/figure-05-user-breakdown.png)


*Figure 5: How the 240,000 users split across subscription plans, regions, and devices.*

**Looking at the outcome we care about.** Users were also grouped by whether they churned or not, giving a first look at the overall churn split before any deeper analysis.



![Figure 6: Churn split](screenshots/figure-06-churn-split.png)


*Figure 6: How many users churned versus stayed, before any further digging.*

**Checking the range of the numbers.** Key numeric columns, like age, BMI, and weekly sessions, were checked for their minimum, maximum, and average values. This is what first revealed that some users had unrealistically high weekly session counts, which was investigated further during cleaning.



![Figure 7: Range of key numeric columns](screenshots/figure-07-numeric-ranges.png)


*Figure 7: Minimum, maximum, and average values for key numeric columns.*

Exploration on its own doesn't fix anything. It simply tells us what to pay attention to next, which is where cleaning comes in.

### 6.3 Data Cleaning

Cleaning means **acting on what exploration uncovered:** fixing what's broken, filling in what's missing, and confirming what's already fine.

**Checking for missing information.** All 22 columns were checked at once for blank values, because a blank left unnoticed can quietly throw off an average or a total.



![Figure 8: Missing value audit](screenshots/figure-08-missing-value-audit.png)


*Figure 8: The count of blanks found in every column.*

This turned up two columns with gaps: `bmi` was missing for 36,215 users (about 15%), and `community_posts_per_month` was missing for 13,958 users (about 6%). Every other column, including the churn outcome, was complete.

**Checking for duplicate users.** If the same user appeared twice in the data, they would be counted twice in every result, which would quietly inflate the numbers. A check confirmed there were **zero duplicate users**.



![Figure 9: Duplicate check](screenshots/figure-09-duplicate-check.png)


*Figure 9: No duplicate user IDs found.*

**Filling in the missing values.** Rather than deleting 15% of users and losing that information, the missing BMI and community post values were filled in with the **average value** of that column. This is a simple, honest way to handle gaps in numeric data, and neither of these two columns is used to answer any of the seven business questions, so this choice does not affect any finding in this report. After filling them in, a re-check confirmed zero blanks remained.



![Figure 10: Filling in missing values](screenshots/figure-10-fill-missing-values.png)


*Figure 10: Blank counts before filling and after, now at zero.*

**Making sure values made sense.** Every category column (like gender, region, and plan) was checked for typos or inconsistent spelling, every yes/no column was checked to make sure it only held yes or no, and every percentage column was checked to make sure it stayed within a realistic range. Everything came back clean.



![Figure 11: Consistency checks](screenshots/figure-11-consistency-checks.png)


*Figure 11: Categories, yes/no fields, and value ranges all confirmed valid.*

**Flagging unusual activity.** Following up on what exploration flagged in Figure 7, a closer look confirmed about 2,470 users (roughly 1% of everyone) with more than 21 app sessions in a single week, some over 200. That is not realistic for a real person opening an app. These users were flagged rather than deleted, since they turned out to matter later, in Question 5.



![Figure 12: Unusual session counts](screenshots/figure-12-unusual-values.png)


*Figure 12: Users with implausibly high weekly session counts.*

### 6.4 Splitting the Data Into Four Tables

Once the data was clean, it was moved out of the raw table and into the four organized tables described in Section 3 (`users`, `subscription`, `engagement`, `churn_status`). Each new table record received its own auto-generated ID, so nothing had to be typed by hand and no ID could ever be duplicated.



![Figure 13: Creating and populating the four tables](screenshots/figure-13-create-and-load-tables.png)


*Figure 13: The four tables created and filled with the cleaned data.*

### 6.5 Final Quality Checks

Before trusting any result, two things were confirmed: every one of the four tables held exactly 240,000 rows, and no record existed without a matching user. Both checks passed.



![Figure 14: Final validation](screenshots/figure-14-final-checks.png)


*Figure 14: All four tables matched at 240,000 rows each, with zero orphaned records.*

### 6.6 Query Optimization

With four tables holding 240,000 rows each, queries that join them together can slow down as the data grows. To keep things fast, indexes were added on the `user_id` column in each table, which works like a table of contents so the database can find matching records without scanning every row.



![Figure 15: Indexes created](screenshots/figure-15-indexes-created.png)


*Figure 15: Indexes added on `user_id` across the tables.*

To confirm the improvement, a query's execution plan was checked before and after adding the indexes.



![Figure 16: Query performance before and after indexing](screenshots/figure-16-query-optimization.png)


*Figure 16: Query execution time compared before and after the indexes were added.*

> *If this step wasn't part of the final project, Section 6.6 and Figures 15–16 can simply be removed.*

At this point, the data was clean, verified, properly organized, and optimized, ready to answer the client's seven questions with confidence.

---

## 7. Analysis Results (Q1 to Q7)

To answer each question, the four cleaned tables were joined back together using `user_id`, so a user's plan, activity, and outcome could all be looked at side by side. **Churn rate** always means the same thing throughout this section: *of the users in a given group, what percentage left within six months?* The number to compare every group against is the overall churn rate of **21.08%**.

---

### Question 1: How many users are we losing overall?



![Figure 17: Overall churn](screenshots/figure-17-q1-overall-churn.png)


*Figure 17: Total, churned, and retained users.*

**Finding:** Out of 240,000 users, **50,593 left within six months**, which is a churn rate of **21.08%**.
**In plain terms:** roughly **1 out of every 5 users** who sign up is gone within half a year.

---

### Question 2: Does the subscription plan affect churn?



![Figure 18: Churn by plan](screenshots/figure-18-q2-churn-by-plan.png)


*Figure 18: Churn rate by plan.*

| Plan | Users | Churn rate |
|---|---:|---:|
| Free | 69,562 | 21.81% |
| Premium | 92,206 | 21.14% |
| Basic | 78,232 | 20.36% |

**Finding:** The churn rate barely moves between plans, a spread of just **1.45 percentage points** from the lowest (Basic, 20.36%) to the highest (Free, 21.81%).
**In plain terms:** it doesn't matter much which plan a user is on, they leave at almost the same rate either way. Pricing is not the problem.

---

### Question 3: Do some regions lose more users than others?



![Figure 19: Churn by region](screenshots/figure-19-q3-churn-by-region.png)


*Figure 19: Churn rate by region.*

| Region | Users | Churn rate |
|---|---:|---:|
| US | 34,354 | 22.00% |
| EU | 48,043 | 21.88% |
| LATAM | 58,439 | 21.47% |
| SEA | 39,807 | 21.21% |
| KR | 59,357 | 19.43% |

**Finding:** Churn ranges from **19.43% in KR** (the lowest) to **22.00% in the US** (the highest), a gap of about 2.6 percentage points.
**In plain terms:** no region is in serious trouble. KR keeps its users slightly better than everywhere else, and might be worth studying to see what it's doing right.

---

### Question 4: Does auto-renewal affect whether a user stays?



![Figure 20: Churn by auto-renew status](screenshots/figure-20-q4-auto-renew.png)


*Figure 20: Churn for users with and without auto-renewal switched on.*

| Auto-renew | Users | Churn rate |
|---|---:|---:|
| Off | 121,168 | **25.48%** |
| On | 118,832 | **16.60%** |

**Finding:** Users who leave auto-renew switched off churn at **25.48%**, compared with **16.60%** for users who have it switched on, a gap of **8.88 percentage points**. That means a user without auto-renew is about **54% more likely** to leave than one with it. Users without auto-renew make up roughly half the users overall, yet they account for **61% of everyone who churned**.
**In plain terms:** this is by far the strongest pattern in the whole dataset. Turning auto-renew on is closely tied to a user staying.

---

### Question 5: Are the users who leave less active in the app than the ones who stay?



![Figure 21: Engagement of churned vs. retained users](screenshots/figure-21-q5-engagement.png)


*Figure 21: Average activity for users who stayed (`false`) versus users who left (`true`).*

| Measure | Stayed | Left |
|---|---:|---:|
| Weekly sessions | 2.19 | 3.77 |
| Minutes per session | 23.46 | 24.18 |
| Workout completion | 0.56 | 0.57 |
| Diet logging | 0.59 | 0.58 |
| Sleep tracking | 0.53 | 0.58 |
| Coaching messages / week | 0.71 | 0.80 |

**Finding:** Every measure except weekly sessions is almost identical between the two groups, within a couple of points of each other. The weekly-session gap (3.77 vs. 2.19) looked meaningful at first, but it turned out to come almost entirely from the roughly 2,470 users flagged earlier for unrealistic activity (Figure 12), who churn at **37.5%**. Take those users out, and both groups average close to **1 session a week**.
**In plain terms:** contrary to what you'd expect, users who leave are **not** less active than users who stay. Activity level alone does not predict who is about to churn.

---

### Question 6: Does the device a user is on make a difference?



![Figure 22: Churn by device type](screenshots/figure-22-q6-churn-by-device.png)


*Figure 22: Churn rate by device.*

| Device | Users | Churn rate |
|---|---:|---:|
| Web only | 63,910 | **23.71%** |
| iOS | 80,094 | 20.23% |
| Android | 95,996 | 20.04% |

**Finding:** Users on the web-only version churn at **23.71%**, about **3.5 to 3.7 percentage points higher** than users on iOS or Android.
**In plain terms:** users who only access the app through a browser are more likely to leave than users on the mobile app. The web experience may be weaker, or web-only users may simply be less committed to begin with.

---

### Question 7: Do users who finish more of their workouts stick around longer?



![Figure 23: Churn by workout completion](screenshots/figure-23-q7-workout-completion.png)


*Figure 23: Churn rate by workout-completion group. Low is under 50% of workouts completed, Mid is 50–74%, High is 75% and above.*

| Workout completion | Users | Churn rate |
|---|---:|---:|
| Low (under 50%) | 87,402 | 20.79% |
| Mid (50–74%) | 103,857 | 21.07% |
| High (75%+) | 48,741 | 21.63% |

**Finding:** Churn rate barely changes across the three groups, staying within a **0.84 percentage point** range. If anything, the users who complete the most workouts churn slightly more, not less.
**In plain terms:** finishing more workouts does not make a user more likely to stay. This challenges the natural assumption that more engaged users are safer users.

---

## 8. Key Findings

| # | Question | Answer, in numbers |
|---|---|---|
| 1 | Overall churn | **21.08%** of all 240,000 users left within 6 months |
| 2 | Plan | Barely matters: 20.36% to 21.81% across all three plans |
| 3 | Region | Fairly even: 19.43% (KR) to 22.00% (US) |
| 4 | Auto-renew | **The biggest driver:** 25.48% without it vs. 16.60% with it |
| 5 | Engagement | No real difference once unusual accounts are excluded |
| 6 | Device | Web-only users churn more: 23.71% vs. about 20% on mobile |
| 7 | Workout completion | No effect: churn stays between 20.79% and 21.63% |

---

## 9. Summary and Conclusion

Out of 240,000 users, just over 1 in 5 (21.08%) leave the app within six months. Most of the factors you'd normally expect to explain that, subscription plan, region, and even how active a user is, turn out to make very little difference. The one factor that stands out clearly is **auto-renewal**: users without it are close to 9 percentage points more likely to churn, and they account for 61% of everyone who leaves. The device a user is on also matters, with web-only users churning noticeably more than mobile users. These two findings give a clear, evidence-based place to start fixing the churn problem.

---

## 10. Recommendations

1. **Increase auto-renewal sign-ups.** This is the single strongest lever found in the data. Prompt users to turn it on during signup and at payment, and consider testing a small incentive, like a short discount, for enabling it.
2. **Improve the web-only experience**, or actively encourage web-only users to move to the mobile app, since they churn about 3.5 points more than mobile users.
3. **Don't spend retention budget on plan pricing, region-specific campaigns, or pushing workout completion.** None of these move the churn rate in any meaningful way, based on this data.
4. **Investigate the flagged high-activity accounts** (about 2,470 users with unrealistic session counts) to confirm whether they're real users, test accounts, bots, or a tracking issue.
5. **Explore other possible drivers** not covered by the original seven questions, such as payment success and account tenure, since the data for both is already available.

---

## 11. Caveats and Limitations

- **This shows patterns, not proof.** The data shows which groups churn more, not that any one factor directly causes users to leave. The auto-renew and device findings should be tested (for example with an A/B test) before being treated as guaranteed fixes.
- **Missing values were filled with column averages.** This is a standard, simple approach, and it does not affect any of the seven findings above, since the two affected columns weren't used to answer them.
- **A small group of users show unrealistic activity levels.** About 1% of users log far more app sessions than seems possible for a real person, and this affected the early read on Question 5 until it was investigated.
- **The churn window is fixed at six months.** The data tells us whether a user left within that window, but not exactly when, so we can't see how quickly users tend to leave.
- **Each question looks at one factor at a time.** Combinations, like a web-only user who also has auto-renew off, were not explored in this round of analysis.

---

## 12. How to Explore This Project
