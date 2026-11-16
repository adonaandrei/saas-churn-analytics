```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
```


```python
df = pd.read_csv('../data/raw/customer_subscription_churn_usage_patterns.csv')
```


```python
# Preview ng data
df.head()
```


```python
# function para makita yung data types columns and non-null counts
df.info()

#  exact number ng missing values kada column
df.isnull().sum()
```


```python
# summary statistics
df.describe()

```


```python
# Convert signup_date s proper datetime format
df['signup_date'] = pd.to_datetime(df['signup_date'])

# kaya nag print para makita if naconvert ba signup_date  from 'object' (string) to 'datetime64'
print(df[['signup_date']].dtypes)
```


```python
# save dataframe sa clean data folder
df.to_csv('../data/clean/clean_saas_data.csv', index=False)

print("Clean dataset successfully exported to /data/clean/clean_saas_data.csv!")
```

## Exploratory Data Analysis (EDA)
Investigating behavioral, financial, and support-related drivers behind customer churn across 2,800 subscriber records.


```python
# Set default styling for all charts
sns.set_theme(style="whitegrid")
```


```python
# Plot overall churn count
plt.figure(figsize=(6, 4))
sns.countplot(data=df, x='churn', hue='churn', palette='Set2', legend=False)
plt.title('Total Customer Churn Distribution')
plt.xlabel('Churn Status')
plt.ylabel('Number of Customers')
plt.show()

# Print natin yung exact numbers and percentages
churn_counts = df['churn'].value_counts()
churn_percentage = df['churn'].value_counts(normalize=True) * 100
print("--- Churn Breakdown ---")
print(churn_counts)
print("\n--- Churn Percentage ---")
print(churn_percentage)
```

### Key Takeaway: Baseline Churn Rate
* **Observation:** Out of 2,800 total subscribers, **1,605 customers (57.32%)** have canceled their subscriptions, compared to **1,195 active users (42.68%)**.
* **Business Impact:** Losing more than half of the customer base indicates a severe retention bottleneck. This **57.32%** figure serves as our benchmark subsequent analysis will isolate which pricing tiers, support friction levels, and usage behaviors push churn above this baseline.


```python
# Plot churn volume by subscription plan
plt.figure(figsize=(8, 5))
sns.countplot(data=df, x='plan_type', hue='churn', palette='Set2')
plt.title('Churn Volume by Subscription Plan')
plt.xlabel('Subscription Plan Type')
plt.ylabel('Number of Customers')
plt.legend(title='Churned')
plt.show()

# Here we added pd.crosstab to print exact churn percentage within each plan tier
plan_churn_rate = pd.crosstab(df['plan_type'], df['churn'], normalize='index') * 100
print("--- Churn Rate (%) by Subscription Plan ---")
print(plan_churn_rate.round(2))
```

### Key Takeaway: Churn by Subscription Plan
* **Observation:** Churn rates are nearly identical across all tiers Premium leads slightly at 58.05%, followed by Basic at 57.85% and Standard at 56.06%.
* **So What:** Because every tier sits right near our 57.32% overall average, plan pricing alone isn't driving customers away. However, losing Premium subscribers hurts revenue the most, so retaining high-tier users should be a priority once we uncover the behavioral triggers.


```python
# Boxplot comparing support tickets for churned vs active users
plt.figure(figsize=(8, 5))
sns.boxplot(data=df, x='churn', y='support_tickets', hue='churn', palette='Set2', legend=False)
plt.title('Impact of Support Tickets on Customer Churn')
plt.xlabel('Churn Status')
plt.ylabel('Number of Support Tickets')
plt.show()

# Print summary stats to get exact median and mean ticket counts
print(df.groupby('churn')['support_tickets'].describe())

```

### Key Takeaway: Support Friction vs. Churn
* **Observation:** Churned customers have a median of 5 support tickets (mean: 4.22), compared to a median of 3 tickets (mean: 3.44) for active users.
* **So What:** High support volume is a clear red flag for cancellation. Flagging accounts for proactive outreach as soon as they log their 4th support ticket could help resolve issues before users decide to leave.


```python
# Boxplot comparing weekly usage hours for churned vs active users
plt.figure(figsize=(8, 5))
sns.boxplot(data=df, x='churn', y='avg_weekly_usage_hours', hue='churn', palette='Set2', legend=False)
plt.title('Average Weekly Usage Hours vs Churn')
plt.xlabel('Churn Status')
plt.ylabel('Average Weekly Usage Hours')
plt.show()

# Print summary stats for usage hours
print(df.groupby('churn')['avg_weekly_usage_hours'].describe())
```

### Key Takeaway: Product Engagement vs. Churn
* **Observation:** Churned users average 12.25 weekly usage hours (median: 12.1), down from 13.75 hours (median: 13.9) for active subscribers. Additionally, 25% of churned users log 5.1 hours or less per week.
* **So What:** Usage dropping below 8 hours per week and especially under 5 hours signals low product adoption. Triggering automated onboarding tips or check-in emails when weekly usage dips below 6 hours can help re-engage at-risk accounts.


```python
# Calculate correlation matrix for numerical business metrics (excluding user_id)
plt.figure(figsize=(10, 6))
numeric_df = df.drop(columns=['user_id']).select_dtypes(include=[np.number])
sns.heatmap(numeric_df.corr(), annot=True, cmap='coolwarm', fmt=".2f", linewidths=0.5)
plt.title('Feature Correlation Heatmap')
plt.show()
```

### Key Takeaway: Feature Correlation Heatmap
* **Observation:** All numerical predictors show near-zero correlation with one another, ranging from -0.06 (support tickets vs. last login days) to 0.03 (weekly usage hours vs. support tickets).
* **So What:** The lack of correlation shows that our behavioral and billing metrics are independent low engagement, support friction, and payment issues do not overlap into a single group. Because each issue affects different segments of users, we need to track all of them separately in our Power BI dashboard rather than relying on a single metric.


```python
# Compare median and mean across remaining numerical metrics by churn status
remaining_metrics = ['payment_failures', 'tenure_months', 'last_login_days_ago']
print("--- Median Values by Churn Status ---")
print(df.groupby('churn')[remaining_metrics].median())

print("\n--- Mean Values by Churn Status ---")
print(df.groupby('churn')[remaining_metrics].mean().round(2))
```

### Key Takeaway: Billing Friction, Inactivity, and Tenure
* **Observation:** Churned users average 34.0 median days since their last login (vs. 22.0 days for active users) and 3.0 median payment failures (vs. 2.0 for active users). Meanwhile, median tenure sits at exactly 18.0 months for both groups.
* **So What:** Long-term customers are just as likely to cancel as newer ones. Real retention risks come from inactivity past 30 days and repeated billing failures—meaning automated re-engagement emails at day 25 of inactivity and better failed-payment recovery after the 2nd failure can directly reduce churn.

## Phase 3 Summary & Next Steps for Power BI
* **Baseline Churn:** 57.32% (1,605 of 2,800 subscribers).
* **Non-Drivers:** Subscription tier (`plan_type`) and account age (`tenure_months`) show nearly identical rates across active and churned groups.
* **Top 4 Churn Drivers:**
  1. **Account Inactivity:** Last login exceeding 30 days (median 34 days for churned users).
  2. **Support Friction:** Reaching 4 to 5+ support tickets (median 5 tickets vs. 3 for active).
  3. **Billing Failures:** Hitting 3+ payment failures (median 3 vs. 2 for active).
  4. **Low Weekly Usage:** Dropping below 8 hours/week (25% of churned users log 5.1 hours or less).
* **Next Phase:** Build an interactive Power BI dashboard using `clean_saas_data.csv` to monitor these four risk thresholds across customer segments and prioritize high-revenue Premium accounts.
