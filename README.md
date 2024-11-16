# **1. Overview**
This project aims to develop a customer segmentation model using the RFM analysis for SuperStore, a global retail company. With a vast customer base, the marketing department is looking to implement targeted marketing campaigns to appreciate loyal customers during the upcoming holiday season and identify potential high-value customers.

Given the company's rapid growth, the marketing team has been challenged with manually segmenting the increasing volume of customer data. To address this, the data analytics team has been tasked with creating a Python-based solution to automate the customer segmentation process using the RFM model.

The insights derived from this analysis will enable the marketing team to tailor their campaigns effectively and improve customer retention.

**RFM** is a popular customer segmentation model used in data analysis. It stands for Recency, Frequency, and Monetary.\
  &nbsp;- **Recency**: Measures how recently a customer made a purchase.\
  &nbsp;- **Frequency**: Measures how often a customer makes purchases.\
  &nbsp;- **Monetary**: Measures the total amount a customer spends.

# **2. Tools and Technologies**
   - Python programming language
   - Global Superstore Transaction Data

This is a transnational data set which contains all the transactions occurring between _01/12/2010_ and _09/12/2011_ for a _UK-based_ and registered non-store online retail. The company mainly sells unique all-occasion gifts. Many customers of the company are wholesalers.

|Field | Explaintion|
|----|----|
|InvoiceNo | Invoice number. Nominal, a 6-digit integral number uniquely assigned to each transaction. If this code starts with letter 'C', it indicates a cancellation.|
|StockCode | Product (item) code. Nominal, a 5-digit integral number uniquely assigned to each distinct product.|
|Description | Product (item) name. Nominal.|
|Quantity | The quantities of each product (item) per transaction. Numeric.|
|InvoiceDate | Invoice Date and time. Numeric, the day and time when each transaction was generated.|
|UnitPrice | Unit price. Numeric, Product price per unit in sterling.|
|CustomerID | Customer number. Nominal, a 5-digit integral number uniquely assigned to each customer.|
|Country | Country name. Nominal, the name of the country where each customer resides.|

# **3. Data Preparation**
**Data Cleaning**

```php

# Check for Datatype
df_ecom.dtypes

# Check for Missing data
df_ecom.isna().mean().sort_values()

# Check for Duplicate data
df_ecom.duplicated().sum()

# Check for Incorrect values
display(df_ecom.describe())
display(df_ecom.describe(exclude="number"))

```

**R-F-M Calculation**

```php

# Create dataframe User
df_user = pd.DataFrame(df_ecom['CustomerID'].unique())
df_user.rename(columns={0: 'CustomerID'}, inplace=True)

# Calculate Recency
df_recency = df_ecom.groupby('CustomerID')['InvoiceDate'].max().reset_index()
df_recency['Recency'] = df_recency['InvoiceDate'].apply(lambda x: (dt.date(2011, 12, 31) - x.date()).days)

df_user = df_user.merge(df_recency, on='CustomerID', how='left')
df_user.drop(columns='InvoiceDate', inplace=True)

# Calculate Frequency
df_frequency = df_ecom.groupby('CustomerID')['InvoiceNo'].count().reset_index()

df_user = df_user.merge(df_frequency, on='CustomerID', how='left')
df_user.rename(columns={'InvoiceNo': 'Frequency'}, inplace=True)

# Calculate Monetary
df_ecom['TotalPay'] = df_ecom['Quantity'] * df_ecom['UnitPrice']
df_monetary = df_ecom.groupby('CustomerID')['TotalPay'].sum().reset_index()

df_user = df_user.merge(df_monetary, on='CustomerID', how='left')
df_user.rename(columns={'TotalPay': 'Monetary'}, inplace=True)

```
*Note: The reference date for calculating Recency is December 31, 2017.*

**Create RFM Score**

```php

# Calculate quintiles for each RFM metric
df_user['R_Score'] = pd.qcut(df_user['Recency'], q=5, labels=[5,4,3,2,1])
df_user['F_Score'] = pd.qcut(df_user['Frequency'], q=5, labels= [1,2,3,4,5])
df_user['M_Score'] = pd.qcut(df_user['Monetary'], q=5, labels= [1,2,3,4,5])

# Combine RFM scores into a single RFM_Score
df_user['RFM_Score'] = df_user['R_Score'].astype(str) + df_user['F_Score'].astype(str) + df_user['M_Score'].astype(str)

```

**Segmentation**

```php

# Create segmentation function
def map_segment(score):
    if score in [555, 554, 544, 545, 454, 455, 445]:
        segment = 'Champions'
    elif score in [543, 444, 435, 355, 354, 345, 344, 335]:
        segment = 'Loyal'
    elif score in [553, 551, 552, 541, 542, 533, 532, 531, 452, 451, 442, 441, 431, 453, 433, 432, 423, 353, 352, 351, 342, 341, 333, 323]:
        segment = 'Potential Loyalist'
    elif score in [512, 511, 422, 421, 412, 411, 311]:
        segment = 'New Customers'
    elif score in [525, 524, 523, 522, 521, 515, 514, 513, 425,424, 413,414,415, 315, 314, 313]:
        segment = 'Promising'
    elif score in [535, 534, 443, 434, 343, 334, 325, 324]:
        segment = 'Need Attention'
    elif score in [331, 321, 312, 221, 213, 231, 241, 251]:
        segment = 'About To Sleep'
    elif score in [255, 254, 245, 244, 253, 252, 243, 242, 235, 234, 225, 224, 153, 152, 145, 143, 142, 135, 134, 133, 125, 124]:
        segment = 'At Risk'
    elif score in [155, 154, 144, 214,215,115, 114, 113]:
        segment = 'Cannot Lose Them'
    elif score in [332, 322, 233, 232, 223, 222, 132, 123, 122, 212, 211]:
        segment = 'Hibernating Customers'
    else:
        segment = 'Lost customers'
    return segment

# Apply the segmentation function to create 'Segment' column
df_user['RFM_Score'] = df_user['RFM_Score'].astype(int)
df_user['Segment'] = df_user['RFM_Score'].map(map_segment)

```

# **3. Data Visualization**

## **RFM Distribution**

```php

# Distribution of Recency, Frequency, Monetary
fig, ax = plt.subplots(figsize=(15,5), ncols=3)
sns.histplot(data=df_user, x='Recency', bins=30, ax=ax[0])
sns.histplot(data=df_user, x='Frequency', bins=30, ax=ax[1])
sns.histplot(data=df_user, x='Monetary', bins=30, ax=ax[2])
ax[0].set_title('Recency Distribution')
ax[1].set_title('Frequency Distribution')
ax[2].set_title('Monetary Distribution')
plt.show()

```

<img src="https://i.imgur.com/bRoiQzl.png">

**Recency**:

* The peak of the distribution is around 0-100 days, indicating that many customers have made recent purchases within this timeframe.
* The long tail suggests that there is a segment of customers who have not purchased for a longer period. These customers might be at risk of churn.

**Frequency**:

* The peak of the distribution is around 0-250 purchases, indicating that most customers make a few purchases.
* The long tail suggests that there is a segment of high-value customers who make frequent purchases. These customers could be valuable for loyalty programs and personalized offers.

**Monetary**:

* The peak of the distribution is around 0-5000, indicating that most customers make smaller purchases.
* The long tail suggests that there is a segment of high-spending customers who make larger purchases. These customers could be targeted for premium products or exclusive offers.

```php

fig, ax = plt.subplots(figsize=(15,5), ncols=3)
sns.scatterplot(df_user, x='Recency', y='Frequency', ax=ax[0])
sns.scatterplot(df_user, x='Recency', y='Monetary', ax=ax[1])
sns.scatterplot(df_user, x='Frequency', y='Monetary', ax=ax[2])
ax[0].set_title('Recency vs Frequency')
ax[1].set_title('Recency vs Monetary')
ax[2].set_title('Frequency vs Monetary')
plt.show()

```

<img src="https://i.imgur.com/tbAlXEq.png">

**Recency vs. Frequency**:

* The majority of customers who have made recent purchases (lower Recency values) tend to have lower purchase frequencies.
* A few customers with higher Recency values (longer time since last purchase) also have high purchase frequencies, suggesting that they might be occasional high-value customers.

**Recency vs. Monetary**:

* The majority of customers who have made recent purchases (lower Recency values) tend to have lower average purchase amounts.
* A few customers with higher Recency values (longer time since last purchase) also have high average purchase amounts, suggesting that they might be occasional high-spending customers.

**Frequency vs. Monetary**:

* The majority of customers with lower purchase frequencies tend to have lower average purchase amounts.
* As purchase frequency increases, the average purchase amount also tends to increase, indicating a _positive relationship_ between the two.

## **Segment Distribution**

```php

# Distribution of Segment
segment_counts = df_user['Segment'].value_counts().reset_index()
segment_counts.columns = ['Segment', 'Count']

plt.figure(figsize=(15,5))
ax = sns.barplot(segment_counts, x='Count', y='Segment', errorbar=None)
total = segment_counts['Count'].sum()
for p in ax.patches:
    percentage = '{:.1f}%'.format(100 * p.get_width() / total)
    x = p.get_x() + p.get_width()
    y = p.get_y() + p.get_height() / 2
    ax.annotate(percentage, (x, y), ha='right', va='center', color='white')
plt.show()

```

<img src="https://i.imgur.com/sIh1QJ2.png">

# **4. Recommendations**

|Segment | Characteristics| Recommendations|
|--|--|--|
|Hibernating Customers | Haven't made a purchase recently but have a history of activity| **Personalized Email Campaigns**: Tailor email content based on past purchases, preferences, and offer time-limited discounts or exclusive deals.<br>**Loyalty Programs and Rewards**: Implement tiered reward systems and exclusive perks to incentivize repeat purchases and engagement.<br>**Social Media Engagement**: Use targeted ads, interactive content, and customer surveys to rekindle interest and gather feedback.|
|Champions | High purchase frequency and spend, and it's crucial to retain them| **Exclusive Perks and Benefits**: Offer them early access to new products, exclusive discounts, or personalized VIP treatment.<br>**Loyalty Programs**: Design a tiered loyalty program with higher rewards and benefits for Champions.<br>**Referral Programs**: Implement a referral program that offers incentives for referring new customers.|
|Potential Loyalists | Have a history of purchases but haven't reached the level of Champions yet| **Personalized Loyalty Programs**: Create tailored loyalty programs with exclusive perks and rewards to incentivize repeat purchases and increase engagement.<br>**Targeted Communication**: Implement personalized marketing campaigns, including email and SMS, to keep them informed about new products, promotions, and special offers.<br>**Enhanced Customer Experience**: Provide exceptional customer service and support to build strong relationships and foster loyalty.|
|Lost Customers | Haven't made a purchase for a long time| Analyze the reasons for their inactivity and implement targeted win-back campaigns|
|At Risk | Risk of churning. They may have decreased purchase frequency or spend| Implement targeted marketing campaigns, personalized offers, or customer support to retain these customers|
|Loyal Customers | Make regular purchases| Personalized offers, exclusive access to new products, or early access to sales|
|New Customers | Recently made their first purchase| Providing a positive customer experience to encourage repeat purchases|
|Need Attention | Further analysis to understand their behavior and needs| Segment them further based on other factors like purchase history or demographics|
|About To Sleep | Similar to At Risk, but they may be further along the path to churn| Motivate them to make another purchase by offering discounts|
|Promising | Made a few purchases and have the potential to become loyal customers| Implement targeted marketing campaigns to encourage repeat purchases|
|Cannot Lose Them | Small but crucial group of high-value customers| Prioritize their retention and satisfaction with personalized offers, exclusive benefits, and exceptional customer service|
