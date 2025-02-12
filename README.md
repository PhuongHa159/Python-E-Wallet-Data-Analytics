# E-WALLET DATA ANALYTICS

## I. Introduction

This project analyzes payment and transaction data from an e-wallet company to understand payment trends, product performance, and user behavior. The goal is to identify key insights that improve transaction efficiency, detect anomalies, and enhance customer experience.

## II. Dataset

This dataset contains transactional and payment data from an e-wallet company, capturing key details about monthly payment volumes, product performance, and user transactions. It consists of three main files:

- **payment_report.csv** – Contains monthly payment volumes for different products, tracking transaction amounts and trends.
- **product.csv** – Stores product information, including product id, categories, and team own.
- **transactions.csv** – Records individual transactions, including transaction IDs, timestamps, amounts, and statuses (e.g., successful, failed, refunded).

## III. Explore data

### Part 1: EDA

EDA is a crucial step to assess data quality before performing in-depth analysis. This process includes checking for missing values, handling duplicates, correcting data types, and identifying potential anomalies in the dataset. The key steps involve:

- Merging **payment_report.csv** with **product.csv** to create `payment_enriched` for a more comprehensive analysis.
- Checking for missing values and deciding whether to remove or impute them using relevant metrics.
- Identifying and eliminating duplicate records to ensure data accuracy.
- Correcting incorrect data types to maintain consistency across numerical and categorical fields.
- Detecting and handling incorrect or outlier values to improve data reliability.

The following analysis examines both the `payment_enriched` and `transactions` datasets to highlight potential data quality issues and recommend appropriate cleaning actions.

**Actions:**

```
df_pay.info()
```

<img width="181" alt="Image" src="https://github.com/user-attachments/assets/51f691f0-4679-4fdc-982b-9e66b305e299" />

```
df_trans.info()
```
<img width="199" alt="Image" src="https://github.com/user-attachments/assets/d901ab47-460b-4c2e-905f-74886df20e74" />


**EDA check:**
- Missing data:
  * 22 rows in column df_pay["category"]
  * 22 rows in column df_pay["team_own"]
  -> Delete rows

  * 49059 rows in df_trans["sender_id"]
  * 164795 rows in df_trans["receiver_id"]
  * 1317907 rows in df_trans["extra_info"]
  -> Delete rows in sender_id, delete columns extra_info, imputing data in receiver_id by mode

- Duplicates:
  * df_pay: 0 -> No action
  * df_trans: 27 -> Delete row

- Incorrect data types:
  * df_pay["report_month"] -> Change to datetime
  * df_trans["sender_id","receiver_id"] -> Change to int
  
- Incorrect values:
  * df_pay:0 -> No action
  * df_trans: 0-> No action


**Check each column: missing data? duplicates? incorrect data types?**

**Payment_enriched**
```
#dropping missing value
threshold_pay=len(df_pay) *0.05
cols_to_drop=df_pay.columns[(df_pay.isna().sum() > 0) & (df_pay.isna().sum() < threshold_pay)]
df_pay.dropna(subset=cols_to_drop,inplace=True)
print(df_pay.isna().sum())

# duplicate?
duplicates = df_pay.duplicated().sum()
print("Duplicate in Payment enriched:",duplicates)

#incorrect data types?
print(df_pay.dtypes)
df_pay["report_month"] = pd.to_datetime(df_pay['report_month'], format='%Y-%m')
print(df_pay.dtypes)

#check outliner
def find_outliers_iqr(df_pay):
  outliers = pd.DataFrame()
for column in df_pay.select_dtypes(include=['int']).columns:
        Q1 = df_pay[column].quantile(0.25)
        Q3 = df_pay[column].quantile(0.75)
        IQR = Q3 - Q1
      
        outliers_column = df_pay[(df_pay[column] < (Q1 - 1.5 * IQR)) | (df_pay[column] > (Q3 + 1.5 * IQR))]
      
outliers = find_outliers_iqr(df_pay)
print("Outlier in Payment_enriched:",outliers)
```

**transactions**

```
  #check missing value
print(df_trans.isna().sum())
threshold_trans=len(df_trans)*0.05

    #dropping missing value
transactions_drop=df_trans.columns[(df_trans.isna().sum() > 0) & (df_trans.isna().sum() <= threshold_trans)]
df_trans.dropna(subset=transactions_drop,inplace=True)
df_trans.drop(columns=['extra_info'], inplace=True)
print(df_trans.isna().sum())

    #Imputing data
cols_to_impute=df_trans.columns[df_trans.isna().sum() > threshold_trans]
for col in cols_to_impute:
  df_trans[col] = df_trans[col].fillna(df_trans[col].mode()[0])
print(df_trans.isna().sum())

    # duplicate?
duplicates = df_trans.duplicated().sum()
print("Duplicates in transactions:",duplicates)
df_trans.drop_duplicates(keep='last')

    #incorrect data types?
print(df_trans.dtypes)
df_trans["sender_id"] = df_trans["sender_id"].astype(int)
df_trans["receiver_id"] = df_trans["receiver_id"].astype(int)
print(df_trans.dtypes)

    #check outliner
def find_outliers_iqr(df_trans):
    outliers = pd.DataFrame()
for column in df_trans.select_dtypes(include=['int']).columns:
        Q1 = df_trans[column].quantile(0.25)
        Q3 = df_trans[column].quantile(0.75)
        IQR = Q3 - Q1
      # Filter outlier and add to Dataframe
        outliers_column = df_trans[(df_trans[column] < (Q1 - 1.5 * IQR)) | (df_trans[column] > (Q3 + 1.5 * IQR))]
      # Find outlier
outliers = find_outliers_iqr(df_trans)
print("Outlier in transactions:",outliers)

```

## Part II: Data Wrangling

**1. Top 3 product_ids with the highest volume**

```
top_three_product=payment_report.groupby("product_id")["volume"].sum().nlargest(3)

print("Top 3 product_ids with the highest volume:", top_three_product)
```

<img width="251" alt="Image" src="https://github.com/user-attachments/assets/d653dac5-bbe4-4fa2-8201-f2e38366a06c" />


**Key observations**: The top 3 products have high transaction volumes, with Product ID 1976 leading at over 61 billion units. Product IDs 429 and 372 follow with significant volumes. These products could be the company's core offerings, and focusing on enhancing and developing them further, along with deeper analysis, could provide valuable insights into customer preferences and sales trends, helping to identify other potential high-performing products.


**2. Given that 1 product_id is only owed by 1 team, are there any abnormal products against this rule?**

```
  # Find 1 product_id is only owed by 1 team
abnormal_product = product.groupby("product_id")["team_own"].nunique()

  # Find abnormal products against this rule
abnormal_product = abnormal_product[abnormal_product > 1]

print("Abnormal products against this rule:",abnormal_product)
```

**3. Find the team has had the lowest performance (lowest volume) since Q2.2023. Find the category that contributes the least to that team**

```
  # Find data since Q2.2023
payment_q2_2023 = payment_enriched[payment_enriched["report_month"].isin(['2023-04', '2023-05', '2023-06'])]

  # Find the team has had the lowest performance (lowest volume) since Q2.2023
lowest_team=payment_q2_2023.groupby("team_own")["volume"].sum().idxmin()

  # Find the category that contributes the least to that team.
team_low = payment_q2_2023[payment_q2_2023["team_own"] == lowest_team]
least_category= team_low.groupby("category")["volume"].sum().idxmin()

print("The team has had the lowest volume since Q2.2023:", lowest_team)
print("The category that contributes the least:", least_category)
```
<img width="332" alt="Image" src="https://github.com/user-attachments/assets/02c4e137-5379-4884-bef1-fe77a27e1001" />


**Key observations**: 
The team with the lowest performance (lowest volume) since Q2.2023 is **APS**. Within this team, the category contributing the least to its volume is **PXXXXXE**. This suggests that APS may need to assess its performance drivers and consider strategies to improve its volume, particularly in the PXXXXXE category.


**4. Find the contribution of source_ids of refund transactions (payment_group = ‘refund’), what is the source_id with the highest contribution?**

```
  #Find refund transactions
refund_transactions=payment_report[payment_report["payment_group"]=="refund"]

  #Find the contribution of source_ids of refund transactions
source_id_contribution=refund_transactions.groupby("source_id")["volume"].sum()

  #Find the source_id with the highest contribution
highest_contribution=source_id_contribution.idxmax()

print("The contribution of source_ids of refund transactions:",source_id_contribution)
print("The source_id with the highest contribution:",highest_contribution)
```
<img width="405" alt="Image" src="https://github.com/user-attachments/assets/b4d61c51-7bef-4501-86ee-0487f6834aba" />

**Key observations**: `Source_id` **38** has the highest number of refund transactions and needs further analysis to understand the root cause. This can help identify potential issues and develop solutions to mitigate them or improve the product.


**5. Define type of transactions (‘transaction_type’) for each row, given:**

- **transType = 2 & merchant_id = 1205: Bank Transfer Transaction**
- **transType = 2 & merchant_id = 2260: Withdraw Money Transaction**
- **transType = 2 & merchant_id = 2270: Top Up Money Transaction**
- **transType = 2 & others merchant_id: Payment Transaction**
- **transType = 8, merchant_id = 2250: Transfer Money Transaction**
- **transType = 8 & others merchant_id: Split Bill Transaction**
    
    **Remained cases are invalid transactions**
    

```
def type_transactions(row):
  if row["transType"]==2 :
    if row["merchant_id"] == 1205 :
      return "Bank Transfer Transaction"
    elif row["merchant_id"] == 2260 :
      return "Withdraw Money Transaction"
    elif row["merchant_id"] == 2270 :
      return "Top Up Money Transaction"
    else:
      return "Payment Transaction"
  elif row["transType"] == 8 :
    if row["merchant_id"] == 2250 :
      return "Transfer Money Transaction"
    else :
      return "Split Bill Transaction"
  else :
    return "Invalid transactions"

    #Apply transaction_type in transactions
transactions['transaction_type'] = transactions.apply(type_transactions, axis=1)
print("Transaction_type in transactions:",transactions.head())
```

**6. Of each transaction type (excluding invalid transactions): find the number of transactions, volume, senders and receivers.**

```
# Find valid_transaction(excluding invalid transactions)
valid_transaction=transactions[transactions['transaction_type'] != 'Invalid transactions']

# Find the number of transactions, volume, senders and receivers
valid_transaction_describe = valid_transaction.groupby("transaction_type").agg(
    num_transaction=("transaction_id","count"),
    sum_volume=("volume","sum"),
    num_sender=("sender_id","nunique"),
    num_receiver=("receiver_id","nunique")
)
print("The number of transactions, volume, senders and receivers:",valid_transaction_describe)
```

**Key observations**:

- **Payment Transactions Dominate in Volume and Engagement**
    - **Largest number of transactions (398,677)** and **widest reach (113,298 receivers)** indicate that this transaction type is the most frequently used.
    - Despite having the second-highest total volume (71.85 billion), its high transaction count suggests that **average transaction values might be smaller**, pointing to frequent but lower-value payments.
- **Top Up Money Transactions Contribute the Highest Volume**
    - **Total volume (108.6 billion) is the highest**, even though it has fewer transactions (290,502) than Payment Transactions.
    - This suggests **higher-value transactions per instance**, likely due to users loading significant amounts into their accounts at once.
    - The number of senders and receivers is equal (110,409), indicating **one-to-one transactions, likely between users and the platform.**
- **Transfer Money Transactions Are Frequent but Lower in Volume**
    - **341,177 transactions** (second highest after Payment Transactions), but the total volume (37 billion) is considerably lower.
    - This suggests **smaller peer-to-peer (P2P) transfers**, possibly for personal transactions rather than large payments.
    - The number of senders (39,021) and receivers (34,585) is relatively close, indicating a balanced ecosystem of money transfers.
- **Bank Transfers Are Used Less Frequently but Involve Large-Scale Transactions**
    - **37,879 transactions, 50.6 billion in volume** → While fewer in number, these transactions carry relatively higher values on average.
    - The **high sender-to-receiver ratio (23,156 senders vs. 9,271 receivers)** suggests a concentration of funds being sent to a smaller group, possibly businesses or financial institutions.
- **Withdraw Money Transactions Indicate a High Cash-Out Volume**
    - **33,725 transactions with a total volume of 23.4 billion** suggests that users are withdrawing significant funds, likely converting digital balance to physical cash or bank accounts.
    - The **sender and receiver counts (24,814 each) are identical**, confirming that these transactions are straightforward cash-out operations.
- **Split Bill Transactions Are Niche But Show Social Payment Behavior**
    - **1,376 transactions, 4.9 million in volume** → The lowest among all types, but its presence indicates a small but engaged user base using this feature.
    - The **sender count (1,323) is close to the transaction count**, suggesting **individual-driven group payments**, rather than system-automated ones.
    
    ## V. Recomendations:
    
    ### **1. Improve performance of the APS team (Lowest-performing team)**
    
    - **Recommendation:**
        - Conduct an in-depth analysis to identify the reasons behind the **APS** team's low performance. This could be related to factors such as marketing strategy, product portfolio management, or work processes.
        - Improve management and marketing strategies for the **PXXXXXE** product portfolio. Consider implementing retraining programs for the team or improving production and distribution processes.
        - Strengthen support from other teams or explore collaboration opportunities to enhance overall results.
    
    ### **2. Analyze causes and improve refund transaction rates (Refund Transactions)**
    
    - **Recommendation:**
        - **The product with `source_id` 38 has a high refund rate**, so a deeper analysis is needed to understand the causes. There could be issues related to product quality, customer service, or unclear refund policies.
        - **Solution:** Improve refund policies and customer service quality to reduce refund rates. Improving the process and creating more transparency in refund policies can help minimize unnecessary refunds.
    
    ### **3. Optimize key transactions (Payment, Top-Up)**
    
    - **Recommendation:**
        - **Payment** and **Top-Up** transactions have high transaction volumes and are important, so they should continue to be maintained and optimized. Consider implementing strategies to increase transaction value, such as offering promotional packages for users performing multiple transactions.
        - Invest in optimizing payment processes, reducing transaction fees, and improving user experience to retain loyal customers.
    
    ### **4. Promote less common but potential transactions (Split Bill Transactions)**
    
    - **Recommendation:** Although **split bill** transactions have low volume, they have potential for future growth. It is important to strengthen marketing strategies and educate users about this feature, especially among younger user groups or in scenarios involving shared costs.
    
    ### **5. Scale up production and distribution of core products**
    
    - **Recommendation:** Core products with the highest transaction volumes, like **Product ID 1976**, should be maintained and further developed. Investing in production, improving quality, and expanding distribution channels will help ensure these products can meet the growing market demand.
    - **Improve marketing strategies:** Strengthen marketing campaigns and promotional programs for these core products to boost revenue and maintain market leadership.
    - **Customer behavior analysis:** Conduct deep analysis of user behavior to understand why these products are popular, then make improvements or develop similar products that could capture market share.
    
    ### **6. Improve banking and withdrawal transaction services**
    
    - **Recommendation:** **Bank Transfer** and **Withdraw Money** transactions have large financial volumes but low transaction counts. These services should be optimized by reducing processing times and enhancing security to increase convenience and customer trust when performing these transactions.
    
    ### **Conclusion:**
    
    Overall, to optimize business performance and customer experience, the company should continue to monitor core transactions and products while focusing on improving factors that could help reduce refund rates and drive growth in less common but promising transactions. Developing products and implementing effective marketing strategies will help the company maintain competitiveness and achieve sustainable growth in the mar
