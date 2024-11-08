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

# Quartile for R_score
r1, r2, r3, r4 = df_user['Recency'].quantile([0.2, 0.4, 0.6, 0.8])
df_user['R_score'] = df_user['Recency'].apply(lambda x: 5 if x < r1
                                                               else 4 if (r1 <= x) & (x < r2)
                                                               else 3 if (r2 <= x) & (x < r3)
                                                               else 2 if (r3 <= x) & (x < r4)
                                                               else 1)

# Quartile for F_score
f1, f2, f3, f4 = df_user['Frequency'].quantile([0.2, 0.4, 0.6, 0.8])
df_user['F_score'] = df_user['Frequency'].apply(lambda x: 1 if x < f1
                                                               else 2 if (f1 <= x) & (x < f2)
                                                               else 3 if (f2 <= x) & (x < f3)
                                                               else 4 if (f3 <= x) & (x < f4)
                                                               else 5)

# Quartile for M_score
m1, m2, m3, m4 = df_user['Monetary'].quantile([0.2, 0.4, 0.6, 0.8])
df_user['M_score'] = df_user['Monetary'].apply(lambda x: 1 if x < m1
                                                               else 2 if (m1 <= x) & (x < m2)
                                                               else 3 if (m2 <= x) & (x < m3)
                                                               else 4 if (m3 <= x) & (x < m4)
                                                               else 5)

# Create RFM_score
df_user['RFM_score'] = df_user['R_score'].astype(str) + df_user['F_score'].astype(str) + df_user['M_score'].astype(str)

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
df_user['Segment'] = df_user['RFM_score'].map(map_segment)

```

# **3. Data Visualization**

**RFM Distribution**

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

<img src="">

**Segment Distribution**

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

<img src="">

# **4. Recommendations**
