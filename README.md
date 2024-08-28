# Amazon Sales Report

## Problem Statement
The objective of this project is to analyze sales data to provide actionable insights and recommendations. By understanding sales trends, customer behavior, product preferences, and geographical distribution, the project aims to optimize sales strategies, improve customer satisfaction, and enhance overall business performance.

## Deliverables
1. **Comprehensive Analysis Report:** Summarizes key findings, insights, and recommendations.
2. **Visualizations:** Charts and graphs illustrating various aspects of the data analysis.
3. **Insights:** Insights on product preferences, customer behavior, and geographical sales distribution.
4. **Recommendations:** Strategies for improving sales, inventory management, and customer service.

## 1. Data Overview
- **Check for Missing Values:** Identify any missing or null values in the dataset.

```python
data.isnull().sum()
```
![Screenshot 2024-08-27 172612](https://github.com/user-attachments/assets/24ffdc7c-b5da-4e61-9c63-ae7550dcd7a5)
- **Data Types:** Ensure that each column has the correct data type (e.g., dates as datetime, numerical values as float/int).

```python
data.info()
```
![Screenshot 2024-08-27 172826](https://github.com/user-attachments/assets/c31b3ef4-8d0e-4b3d-822d-e8333ad0acd0)
- **Unique Values:** Analyze the unique values in categorical columns like Status, Fulfilment, Sales Channel, Category, etc.

```python
for column in data:
  print(f'{column}\n{data[column].unique()}\n')
```
![Screenshot 2024-08-27 172937](https://github.com/user-attachments/assets/c16c725f-cb6b-42aa-95ab-35ac735de047)

## 2. Descriptive Statistics
- **Quantitative Analysis:** Calculate summary statistics (mean, median, standard deviation) for numerical columns like Qty, Amount.

```python
data.describe()
```
![Screenshot 2024-08-27 173137](https://github.com/user-attachments/assets/9863bb8c-fcb8-4b57-b317-0655c20b584c)
- **Categorical Analysis:** Analyze the distribution of categorical columns like Status, Fulfilment, Sales Channel, etc.

```python
# Checking for the values distribution of the categorical columns in the data
# for that let's separte all categorical column togther
cat_col = data.select_dtypes(include=['object', 'category']).columns
cat_col
```
![Screenshot 2024-08-27 173240](https://github.com/user-attachments/assets/ef2f1e51-ca08-4a70-8c60-907040d01aa7)

```python
for i in cat_col:
  print(f'\n{data[i].value_counts()}\n')
```
![Screenshot 2024-08-27 173352](https://github.com/user-attachments/assets/a9177e22-53ed-4311-94ba-28f8565d9c55)

- **Data Types:**

Confirm that each column has the correct data type. For instance:
**Dates** should be in **datetime** format.

```python
# Creating a new dataframe same as orginal one
df = data.copy()
```
```python
# Filling the null values of curreny and country beacuse we known this data is all about India
df['currency'] = df['currency'].fillna('INR')
df['ship-country'] = df['ship-country'].fillna('IN')
```
```python
# Removing unwanted columns from the data
df.drop(columns=['New','PendingS'],inplace=True)
```

Quantitative fields like **Qty** and **Amount** should be in **float** or **int** format.

Convert any incorrectly typed columns to their appropriate types.

- **Handling Missing Values :**
Handling missing value is crucial step for further analysis so, filling the missing values with the mean of the specific **ship-city** and **ship-state** wise.

```python
# funstion that will fill the missing values along with each def fill_missing_by_state(state, df):
    # Filter the dataframe for the specified state and where Qty equals 1
    state_df = df[(df['ship-state'] == state) & (df['Qty'] == 1)]

    # Group by the relevant columns and compute the mean for 'Amount' where it's non-zero
    group_means = state_df[state_df['Amount'] != 0].groupby(
        ['Category', 'ship-service-level', 'Size', 'Fulfilment', 'ship-city']
    )['Amount'].mean().reset_index()

    # Merge the means back with the original dataframe on the relevant columns
    df = df.merge(
        group_means, 
        on=['Category', 'ship-service-level', 'Size', 'Fulfilment', 'ship-city'], 
        how='left', 
        suffixes=('', '_mean')
    )

    # Fill missing or zero values in 'Amount' where Qty equals 1
    df.loc[(df['Qty'] == 1) & (df['Amount'].isna() | (df['Amount'] == 0)), 'Amount'] = df['Amount_mean']

    # Drop the temporary 'Amount_mean' column
    df.drop(columns='Amount_mean', inplace=True)

```


## 3. Sales Analysis
- **Total Sales:** Calculate the total sales (sum(Amount)), and analyze by various categories like Sales Channel, Category, ship-country, etc.
```python
# calculating total sales
df['Amount'].sum()
```
![Screenshot 2024-08-27 174235](https://github.com/user-attachments/assets/1e7a369b-9fa6-4655-a535-ae06495f6b3a)

- **Sales by Date:** Plot sales trends over time by grouping by Date. Use line charts to visualize monthly or weekly sales trends.
![Screenshot 2024-08-27 174334](https://github.com/user-attachments/assets/91d560ef-4c51-4438-8221-17589362012a)

`weekly-trend`
![Screenshot 2024-08-27 174404](https://github.com/user-attachments/assets/72d16093-4108-4d8f-a6d4-6330092ad01f)

- **Sales by Fulfillment:** Analyze sales based on Fulfilment and fulfilled-by to see how different fulfillment methods impact sales.

```python
# total sales vs Fulfilment
df.groupby('Fulfilment',as_index=False).sum()[['Amount','Fulfilment']].sort_values(by='Amount',ascending=False)
```
![Screenshot 2024-08-27 174605](https://github.com/user-attachments/assets/4de221c0-6fb8-4574-a2d8-70cfd597bebc)

![Screenshot 2024-08-27 174616](https://github.com/user-attachments/assets/dc5b36cf-05ad-40f0-9be9-1031b5f27385)
## 4. Customer Segmentation
- **Customer Segmentation Analysis:** Segment customers based on their purchasing behavior, geographic location, or order preferences.
`Product vs Customers`
```python
# Aggregate sales by category and taking only top 10 category
Category_sales = df.groupby('Category',as_index=False).sum()[['Amount','Category']].sort_values(by='Amount',ascending=False).head(10)

# Plotting total sales vs fulfilment
plt.figure(figsize=(8, 6))
sns.barplot(x='Category', y='Amount', data=Category_sales, palette='viridis')
plt.title('Total Sales vs Category')
plt.xlabel('Category')
plt.ylabel('Total Sales')
plt.grid(True)
# Rotate the x-labels to be vertical
plt.xticks(rotation=90)
plt.show()
```
![Screenshot 2024-08-27 174816](https://github.com/user-attachments/assets/1a69620a-e68c-4705-8790-cd7d2294a306)

`Geolocation`
```python
# Aggregate sales by ship-states by taking only top 10 states
states_sales = df.groupby('ship-state',as_index=False).sum()[['Amount','ship-state']].sort_values(by='Amount',ascending=False).head(10)

# Plotting total sales vs fulfilment
plt.figure(figsize=(8, 6))
sns.barplot(x='ship-state', y='Amount', data=states_sales, palette='viridis')
plt.title('Total Sales vs States')
plt.xlabel('States')
plt.ylabel('Total Sales')
plt.grid(True)
# Rotate the x-labels to be vertical
plt.xticks(rotation=90)
plt.show()
```
![Screenshot 2024-08-27 175535](https://github.com/user-attachments/assets/a8a1ce93-882b-4a56-81f2-92a0903ec29c)

`ship-service-level`
```python
df['ship-service-level'].value_counts()
```
![Screenshot 2024-08-27 175713](https://github.com/user-attachments/assets/a59e22ee-d81d-4f29-bb39-e4b59ad65a17)


![Screenshot 2024-08-27 175743](https://github.com/user-attachments/assets/87a08176-082f-4a41-a7fc-caeebda361ab)


- **B2B vs. B2C Comparison:** Analyze and compare the purchasing patterns of B2B (B2B == True) and B2C (B2B == False) customers.

```python
df.groupby('B2B',as_index=False).sum()[['Amount','B2B']].sort_values(by='Amount',ascending=False)
```

```python
b2b_sales = df.groupby('B2B',as_index=False).sum()[['Amount','B2B']].sort_values(by='Amount',ascending=False)

# Plotting total sales vs fulfilment
plt.figure(figsize=(8, 6))
sns.barplot(x='B2B', y='Amount', data=b2b_sales)
plt.title('Total Sales vs B2b')
plt.xlabel('B2B')
plt.ylabel('Total Sales')
plt.grid(True)
plt.show()
```
![Screenshot 2024-08-27 180314](https://github.com/user-attachments/assets/4ef4b23b-d352-466d-bc0c-fb9f7139e79d)

`Sales over time when B2B = False`
![Screenshot 2024-08-27 180738](https://github.com/user-attachments/assets/be364c27-456e-4055-94ce-cad89b2f117a)

`Sales over time when B2B = False`
![Screenshot 2024-08-27 180854](https://github.com/user-attachments/assets/4c890015-26f1-4ce1-a03f-fd50a19ae249)


- **New vs. Existing Customers:** Compare the behavior and purchase patterns of new (New == True) versus existing customers.
`Since all the values in the New column is **null** so we can't conclude anything`

## 5. Product and Size Analysis
- **Product Performance:** Analyze sales by product categories such as T-shirts, shirts, blazers, etc.
```python
# total sales vs category
df.groupby('Category',as_index=False).sum()[['Amount','Category']].sort_values(by='Amount',ascending=False)
```
![Screenshot 2024-08-27 181238](https://github.com/user-attachments/assets/38191c6b-74d4-4508-ae48-53fc03b47637)
- **Size Distribution:** Examine sales across different sizes to identify trends in customer preferences.
```python
category_qty_sales = df.groupby(['Category','Size'],as_index=False).sum()[['Category','Size','Qty']]
fig = px.bar(x='Size', y='Qty', data_frame=category_qty_sales,color='Category')
fig.update_layout(
    width=800,  # Set width
    height=600  # Set height
)
fig.show()
```
![Screenshot 2024-08-27 181347](https://github.com/user-attachments/assets/270b5175-be5b-4958-819e-e2295295adc9)

`Quantity sold by each Catgeory`
```python
df.groupby('Category',as_index=False).sum()[['Category','Qty']].sort_values(by='Qty',ascending=False)
```
![Screenshot 2024-08-27 181502](https://github.com/user-attachments/assets/5b12bc0b-c0a9-441b-b475-66d3bdae594d)

## 6. Order Analysis
- **Order Status:** Analyze the distribution of Status to see the proportion of completed, pending, or canceled orders.
![Screenshot 2024-08-27 181600](https://github.com/user-attachments/assets/116f4e9e-2ce2-4b41-8566-f3d523a16bad)

- **Order Quantity:** Check the distribution of Qty to understand the order sizes. Group by Category or Size to see how these factors influence the number of items per order.
```python
df.groupby(['Category','Size'],as_index=False).sum()[['Category','Size','Qty']]
```
![Screenshot 2024-08-27 181724](https://github.com/user-attachments/assets/6414f489-80ab-4cc8-bacb-6ab4eafcfa3c)

- **Pending Orders:** Investigate orders marked as PendingS to identify any potential delays or issues.
`Since the column has all **null** values so, we can't analze it`

## 7. Shipping Analysis
- **Courier Performance:** Evaluate Courier Status to identify any patterns in delays or issues with specific couriers.
```python
df['Courier Status'].value_counts()
```
![Screenshot 2024-08-27 182006](https://github.com/user-attachments/assets/496ccd81-d6f8-4649-8c2b-ca9c8fa5e441)

- **Shipping Service Level:** Analyze how different ship-service-level options affect delivery times and customer satisfaction.
![Screenshot 2024-08-27 182128](https://github.com/user-attachments/assets/27c38a67-20fe-4d94-b2bc-74b3fa243c8d)

## 8. Visualization
- **PowerBI:** Utilize PowerBI for visualizing the data, creating dashboards that track key metrics like sales trends, order fulfillment rates, and geographical distributions.
![Screenshot 2024-08-28 225845](https://github.com/user-attachments/assets/7a0b2fb3-efc4-4a7b-ad06-6e43d1a991bb)


## Final Recommendations and Business Insights

### 1. Sales Performance Optimization
**Observations:**
- Significant sales increase from March to April 2022, followed by a decline.
- T-shirts, shirts, and blazers are the highest performers.
- Maharashtra and Karnataka lead in sales, while Surat shows data quality issues.

**Recommendations:**
- **Promotional Strategies:** Investigate and replicate successful promotions from April to sustain sales momentum.
- **Inventory Management:** Increase stock for high-performing categories to avoid stockouts.
- **Regional Focus:** Target top-performing states with marketing and inventory strategies.

### 2. Data Quality and Order Fulfillment
**Observations:**
- Issues with zero amounts, unfulfilled orders, and discrepancies in order statuses, particularly from Surat.

**Recommendations:**
- **Data Verification:** Conduct comprehensive audits, especially for data from Surat, to correct inconsistencies.
- **Process Improvement:** Refine order fulfillment processes to resolve discrepancies in shipping statuses.

### 3. Customer Preferences and Shipping
**Observations:**
- Expedited shipping is preferred and contributes significantly to revenue.

**Recommendations:**
- **Optimize Logistics:** Align logistics with the demand for expedited shipping.
- **Marketing Opportunities:** Promote expedited shipping in marketing campaigns.

### 4. Product and Size Analysis
**Observations:**
- Blazers and perfumes are popular across various sizes.

**Recommendations:**
- **Inventory Allocation:** Stock high-demand sizes adequately, reduce inventory for less popular sizes.
- **Marketing Focus:** Highlight popular products and sizes in marketing efforts.

### 5. Customer Segmentation and B2B Sales
**Observations:**
- The majority of sales come from non-B2B customers.

**Recommendations:**
- **Customer Segmentation:** Develop targeted strategies for different customer segments.
- **Personalized Marketing:** Use customer data to personalize marketing efforts.

### 6. Data Entry and System Synchronization
**Observations:**
- Discrepancies between order statuses and courier statuses suggest potential system errors.

**Recommendations:**
- **System Audit:** Regularly audit and synchronize systems to ensure accurate order tracking.
- **Training and Guidelines:** Provide staff training to reduce errors and improve data quality.

## Conclusion: Ship-State vs. Sales
**Top Sales States:**
- Maharashtra: Rs.13,340,333.05
- Karnataka: Rs.10,480,694.22
- Telangana: Rs.6,915,018.08
- Uttar Pradesh: Rs.6,823,947.08

**Other Notable States:**
- Tamil Nadu: Rs.6,519,182.30
- Delhi: Rs.4,232,738.97

**Lower Sales States:**
- Lakshadweep: Rs.3175.29
- Ladakh: Rs.38388.43
LAKSHADWEEP and LADAKH show lower sales figures compared to the top states, with amounts of $3175.29 and $38388.43, respectively.

**Summary:** The sales distribution indicates that major states like Maharashtra, Karnataka, and Telangana contribute the highest sales. States with substantial sales should be prioritized for marketing and inventory strategies, while states with lower sales may need targeted initiatives to boost performance.
