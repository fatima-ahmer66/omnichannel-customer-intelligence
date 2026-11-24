# **OMNICHANNEL CUSTOMER RISK, CHURN & BASKET INTELLIGENCE PLATFORM**

**End-to-End Retail & E-Commerce Data Warehousing and Data Mining**

# **PROJECT REPORT**

| SUBMITTED TO                          | SUBMITTED BY                                |
| ------------------------------------- | ------------------------------------------- |
| Sir Mohammad Farooq                   | Group Members: Fatima Ahmer, Mahnoor Nadeem |
| Beaconhouse National University (BNU) |                                             |

**Course:** Data Warehousing and Data Mining\
**Submission Date:** 25 November 2026\
**Project Domain:** Retail Analytics | E-Commerce | Customer Intelligence

# **EXECUTIVE SUMMARY**

Retail and e-commerce businesses generate substantial transaction data, but raw records alone do not provide an integrated view of customer value, purchasing behaviour, order returns, or product demand. This project develops an analytical workflow around an omnichannel retail transaction dataset, combining data preparation, dimensional modelling, customer segmentation, return-risk classification, and market-basket analysis.

The notebook begins with 34,500 transaction-line records and 17 attributes. After data quality checks and standardization, the working dataset contains 34,500 rows and 21 columns. A star-schema style warehouse model is prepared with one sales fact table and four dimension tables: customer, product, channel/region, and date. The five tables are exported as CSV files for loading into Snowflake.

Customer features are engineered using Recency, Frequency, Monetary Value (RFM), average order value, basket size, discount ratio, and return ratio. K-Means identifies two customer groups: 4,814 customers classified as At-Risk / Inactive and 3,089 as High-Value Loyal. Four classification models predict order-return risk. Logistic Regression has the highest F1-score (0.1205), while Random Forest has the highest accuracy (0.7044). These results must be interpreted in light of the imbalanced target, where returned orders represent only 5.52% of the dataset.

Apriori and FP-Growth each identify seven frequent single-category itemsets. No association rules meet the selected thresholds because the transaction baskets in the available data contain only one distinct category per order in the observed sample. Consequently, the market-basket component demonstrates the algorithms and their outputs, but does not produce multi-category cross-selling rules.

An important implementation note is that the submitted notebook constructs and exports warehouse tables locally; it does not contain a live Snowflake connection or SQL DDL/load execution. Snowflake is therefore the intended cloud warehouse destination, while the verified implementation in the notebook is the dimensional model and CSV export stage.

# **REPORT CONTENTS**

-

1. Introduction and Project Planning

1)

2. Dataset Collection and Understanding

1)

3. Data Cleaning and Preprocessing

1)

4. Data Warehouse Design and ETL Preparation

1)

5. Exploratory Analysis and Feature Engineering

1)

6. Customer Segmentation

1)

7. Predictive Classification: Order Return Risk

1)

8. Market-Basket Analysis

1)

9. Results and Discussion

1)

10. Limitations and Challenges

1)

11. Conclusion and Recommendations

1)

12. References and Appendices

# **1. INTRODUCTION AND PROJECT PLANNING**

## **1.1 Introduction**

Retail and e-commerce operations record information about orders, customers, products, prices, discounts, payment methods, delivery, regions, and returns. When this information is stored as raw transactional rows, it can be difficult to compare performance across customers, products, dates, and locations. A data warehouse organizes these records into a consistent analytical structure, while data mining techniques help identify patterns and estimate potential risks.

The Omnichannel Customer Risk, Churn & Basket Intelligence Platform uses Python-based data preparation and mining to convert transaction records into structured warehouse tables and customer/order-level analytical outputs.

## **1.2 Problem Statement**

The project addresses the challenge of turning raw retail transaction data into reliable and useful business intelligence. Inconsistent text values, missing or invalid fields, duplicate order-product records, and a lack of dimensional organization can affect reporting and analysis. Businesses also need ways to distinguish customer groups, identify orders with a higher likelihood of being returned, and understand product-category purchasing patterns.

## **1.3 Project Objectives**

1. Load and examine the retail transaction dataset and understand its fields and data types.
2. Clean and standardize identifiers, text fields, dates, and numeric measures.
3. Create derived financial measures and prepare dimension and fact tables.
4. Develop a dimensional warehouse structure suitable for analytical querying and Snowflake loading.
5. Engineer customer-level RFM and behavioural features.
6. Apply K-Means and compare its clustering quality with hierarchical clustering.
7. Train and evaluate classification models for order-return risk.
8. Apply Apriori and FP-Growth to identify frequent product-category itemsets and association rules.
9. Export the warehouse tables and analytical results for reuse.

## **1.4 Project Scope**

- Dataset loading and profiling
- Data cleaning, validation, and feature derivation
- Customer, product, channel/region, and date dimensions
- Fact sales table construction
- CSV export of warehouse tables for Snowflake ingestion
- Customer segmentation and persona profiling
- Order-return risk prediction
- Global and segment-level market-basket analysis
- Model evaluation and business interpretation

## **1.5 Tools and Technologies**

| Technology                      | Purpose in the project                                                              |
| ------------------------------- | ----------------------------------------------------------------------------------- |
| Python                          | Main programming language for data preparation and analysis                         |
| Pandas / NumPy                  | Tabular transformations, aggregation, numerical operations, and feature engineering |
| Scikit-learn                    | Scaling, clustering, classification pipelines, and evaluation metrics               |
| mlxtend                         | Transaction encoding, Apriori, FP-Growth, and association-rule generation           |
| Matplotlib / Seaborn            | Exploratory plots and model/cluster visualizations                                  |
| Jupyter Notebook / Google Colab | Notebook-based development and execution                                            |
| Snowflake (intended target)     | Cloud warehouse destination for exported fact and dimension CSVs                    |

## **1.6 Project Workflow**

1. Load and profile the source transaction data.
2. Clean, standardize, validate, and derive analytical fields.
3. Build dimension tables and the sales fact table.
4. Export warehouse tables for Snowflake ingestion.
5. Create customer behavioural features and scale selected variables.
6. Perform customer clustering and persona interpretation.
7. Train return-risk classification models and compare metrics.
8. Generate frequent itemsets and attempt association-rule generation.
9. Export customer segments, model results, predictions, and association-rule outputs.

# **2. DATASET COLLECTION AND UNDERSTANDING**

## **2.1 Dataset Overview**

The submitted notebook loads a single retail/e-commerce transaction dataset from a CSV file. The exact public dataset name, provider, and source URL are not recorded in the notebook or the supplied README, so they should be added by the project team if available.

| Dataset attribute                   | Recorded details                         |
| ----------------------------------- | ---------------------------------------- |
| Initial shape                       | 34,500 rows × 17 columns                 |
| Working granularity                 | Transaction/order-product line           |
| File format                         | CSV                                      |
| Key identifiers                     | order_id, customer_id, product_id        |
| Date field                          | order_date                               |
| Target field used in classification | returned / derived return-risk indicator |
| Source citation                     | Not specified in supplied project files  |

## **2.2 Dataset Attributes**

| Attribute                     | Description                         |
| ----------------------------- | ----------------------------------- |
| order_id                      | Order identifier                    |
| customer_id                   | Customer identifier                 |
| product_id                    | Product identifier                  |
| category                      | Product category                    |
| price, quantity               | Unit price and purchased quantity   |
| discount                      | Discount percentage                 |
| payment_method                | Payment method used                 |
| order_date                    | Order date                          |
| delivery_time_days            | Delivery duration in days           |
| region                        | Customer/order region               |
| returned                      | Return status                       |
| total_amount, shipping_cost   | Order financial and shipping values |
| profit_margin                 | Recorded profit-margin measure      |
| customer_age, customer_gender | Available customer attributes       |

## **2.3 Initial Data Quality Profile**

The notebook reports no missing values and no exact duplicate rows in the initial 34,500-row dataset. The cleaning stage nevertheless applies explicit checks for essential identifiers, dates, exact duplicates, and duplicate order-product combinations, then standardizes and validates the fields.

| Check                | Initial notebook finding                  |
| -------------------- | ----------------------------------------- |
| Missing values       | 0 reported across the 17 original columns |
| Exact duplicate rows | 0 reported                                |
| Initial records      | 34,500                                    |
| Initial attributes   | 17                                        |

# **3. DATA CLEANING AND PREPROCESSING**

## **3.1 Handling Missing and Duplicate Records**

Rows missing essential order, customer, product, or date identifiers are excluded. Exact duplicate rows are removed, followed by a duplicate check at the order_id and product_id combination level. The initial data-quality output showed zero nulls and zero exact duplicates, and the cleaned dataset retained 34,500 rows.

## **3.2 Text Standardization**

Text-based fields including order, customer, and product identifiers, category, payment method, region, and return status—are converted to strings and stripped of leading or trailing spaces. Blank or null-like category values are standardized to an Unknown label. Return-status values are normalized to support consistent downstream processing.

## **3.3 Data Type Conversion and Imputation**

The order_date field is converted to a datetime value, and records with invalid dates are removed. Numeric fields are converted to numeric types. The notebook fills missing discount and shipping-cost values with zero, and uses the median for missing customer age and delivery-time values.

## **3.4 Business-Rule Validation**

- Customer age is restricted to the range 18–70.
- Quantity must be greater than zero.
- Price must be greater than zero.
- Discount must fall between 0 and 100.
- Delivery time must be zero or greater.

## **3.5 Derived Financial Measures**

The notebook creates gross_amount as price multiplied by quantity; discount_amount as gross amount multiplied by the discount percentage; and net_sales_before_shipping as gross amount less discount amount. It also derives a binary is_returned indicator for modelling.

After preprocessing, the working dataset contains 34,500 rows and 21 columns, with zero remaining null values reported in the notebook.

| Stage                 | Rows   | Columns | Nulls reported |
| --------------------- | ------ | ------- | -------------- |
| Raw dataset           | 34,500 | 17      | 0              |
| Clean working dataset | 34,500 | 21      | 0              |

# **4. DATA WAREHOUSE DESIGN AND ETL PREPARATION**

## **4.1 Dimensional Modelling**

The notebook creates a dimensional structure consisting of a central Fact_Sales table and four supporting dimensions. Surrogate integer keys are generated for the dimensions and then joined to the transaction records to form the fact table.

| Warehouse table    | Purpose                                                   | Exported size |
| ------------------ | --------------------------------------------------------- | ------------- |
| Dim_Customer       | Customer attributes, including age, gender, and age group | 7,903 × 5     |
| Dim_Product        | Product category and standard/median price attributes     | 24,912 × 4    |
| Dim_Channel_Region | Distinct region and payment-method combinations           | 30 × 4        |
| Dim_Date           | Calendar date, date key, and calendar attributes          | 731 × 9       |
| Fact_Sales         | Transaction-level measures and foreign keys to dimensions | 34,500 × 17   |

## **4.2 Fact Table**

Fact_Sales is built by joining the cleaned transaction data to customer, product, channel/region, and date dimension keys. It retains the transaction-level grain and provides measures and keys for sales reporting and analysis.

## **4.3 ETL Workflow**

| ETL stage        | Implementation described in the notebook                                                                        |
| ---------------- | --------------------------------------------------------------------------------------------------------------- |
| Extract          | Read the source CSV into a Pandas DataFrame.                                                                    |
| Transform        | Clean and standardize fields, validate business rules, derive financial measures, and construct dimension keys. |
| Load preparation | Export Dim_Customer, Dim_Product, Dim_Channel_Region, Dim_Date, and Fact_Sales as separate CSV files.           |

The notebook labels these CSV exports as files for Snowflake. However, the supplied code does not establish a Snowflake connection, create Snowflake database/schema/table objects, or execute a COPY/load command. Therefore, this report describes the implemented warehouse preparation and export process, not a verified live Snowflake deployment.

## **4.4 Warehouse Model**

| Central fact | Dimension relationships                 |
| ------------ | --------------------------------------- |
| FACT_SALES   | customer_key → DIM_CUSTOMER             |
| FACT_SALES   | product_key → DIM_PRODUCT               |
| FACT_SALES   | channel_region_key → DIM_CHANNEL_REGION |
| FACT_SALES   | date_key → DIM_DATE                     |

# **5. EXPLORATORY ANALYSIS AND FEATURE ENGINEERING**

## **5.1 Customer Behavioural Features**

Customer-level features are calculated by grouping transaction records by customer. The reference date for recency is taken from the latest order date in the dataset.

| Feature                    | Meaning                                                   |
| -------------------------- | --------------------------------------------------------- |
| Recency                    | Number of days since the customer’s most recent order     |
| Frequency                  | Number of orders placed by the customer                   |
| Monetary                   | Total monetary value associated with the customer         |
| Average Order Value        | Average value per customer order                          |
| Average Basket Size        | Average number of items per order                         |
| Total Items / Total Orders | Customer-level item and order counts                      |
| Discount Ratio             | Discount amount relative to the relevant sales amount     |
| Return Ratio               | Share of the customer’s orders/items recorded as returned |

## **5.2 Feature Preparation**

The notebook prepares customer behavioural variables for clustering, applies feature encoding/scaling where required, and performs correlation analysis and feature selection. A separate classification feature set is assembled at order level for predicting return risk.

## **5.3 Key Dataset Characteristics**

The customer feature table contains 7,903 unique customers, matching the number of rows in the customer dimension. The classification dataset contains 34,500 records and seven predictor columns. The return-risk target is imbalanced: 32,597 records (94.48%) are labelled non-returned and 1,903 (5.52%) are labelled returned.

# **6. CUSTOMER SEGMENTATION**

## **6.1 Clustering Approach**

K-Means clustering is applied to scaled customer behavioural features. The notebook evaluates candidate cluster counts from two through ten using inertia and silhouette score. The selected solution uses two clusters. Hierarchical (agglomerative) clustering is also run on a sample for comparison.

## **6.2 Selecting the Number of Clusters**

| Number of clusters | Inertia   | Silhouette score |
| ------------------ | --------- | ---------------- |
| 2                  | 65,224.48 | 0.170969         |
| 3                  | 59,162.00 | 0.146122         |
| 4                  | 54,235.19 | 0.161729         |
| 5                  | 50,442.53 | 0.137536         |
| 6                  | 47,524.07 | 0.133274         |
| 7                  | 44,940.11 | 0.134559         |
| 8                  | 42,746.05 | 0.125048         |
| 9                  | 41,157.11 | 0.116777         |
| 10                 | 39,827.57 | 0.116747         |

Two clusters were selected because this candidate produced the highest silhouette score in the tested range (approximately 0.171). The score is relatively low, so the separation between customer groups should be interpreted as modest rather than strong.

## **6.3 Customer Segment Profiles**

| Measure             | Cluster 0                    | Cluster 1                  |
| ------------------- | ---------------------------- | -------------------------- |
| Customer count      | 4,814                        | 3,089                      |
| Recency (days)      | 197.22                       | 96.89                      |
| Frequency           | 3.28                         | 6.06                       |
| Monetary value      | 365.59                       | 1,329.03                   |
| Average order value | 118.57                       | 248.39                     |
| Average basket size | 1.34                         | 1.72                       |
| Return ratio        | 0.05                         | 0.06                       |
| Persona assigned    | At-Risk / Inactive Customers | High-Value Loyal Customers |

Cluster 0 contains more customers and has higher average recency, lower order frequency, and lower monetary value. It is labelled At-Risk / Inactive Customers. Cluster 1 has lower recency, higher purchase frequency, higher monetary value, and a larger average order value, and is labelled High-Value Loyal Customers.

## **6.4 Hierarchical Clustering Comparison**

| Algorithm               | Silhouette score             |
| ----------------------- | ---------------------------- |
| K-Means                 | 0.158449 (comparison sample) |
| Hierarchical Clustering | 0.158195                     |

The comparison-sample silhouette scores are very close. K-Means is marginally higher in the reported comparison. The notebook also reports a PCA visualization in which the first two components explain 42.71% of variance.

## **6.5 Business Interpretation**

- At-Risk / Inactive customers may be candidates for re-engagement campaigns, subject to validating the segment with business stakeholders.
- High-Value Loyal customers may be suitable for loyalty benefits, retention initiatives, and personalized offers.
- Because the silhouette scores are low, segment labels should be treated as descriptive profiles rather than definitive customer types.

# **7. PREDICTIVE CLASSIFICATION: ORDER RETURN RISK**

## **7.1 Prediction Objective**

The classification task predicts whether an order is returned. Although the broader project title refers to customer risk and churn, the implemented target in the notebook is order return status, not customer churn. The report therefore uses the more precise term order-return risk.

## **7.2 Target Distribution and Features**

The classification dataset has 34,500 rows and seven predictor variables. The target contains 32,597 non-returned orders (94.48%) and 1,903 returned orders (5.52%). Class weights are calculated to address this imbalance in selected model pipelines.

## **7.3 Models and Evaluation**

Four classifiers are trained: Logistic Regression, Decision Tree, Random Forest, and Gradient Boosting. The notebook evaluates Accuracy, Precision, Recall, F1-score, and ROC-AUC. An 80/20 train-test split is used with a fixed random seed of 42.

| Model               | Accuracy | Precision | Recall | F1-score | ROC-AUC |
| ------------------- | -------- | --------- | ------ | -------- | ------- |
| Logistic Regression | 0.5510   | 0.0673    | 0.5741 | 0.1205   | 0.5817  |
| Decision Tree       | 0.5735   | 0.0656    | 0.5256 | 0.1166   | 0.5671  |
| Random Forest       | 0.7044   | 0.0635    | 0.3288 | 0.1065   | 0.5568  |
| Gradient Boosting   | 0.5643   | 0.0656    | 0.5391 | 0.1170   | 0.5717  |

## **7.4 Model Interpretation**

Logistic Regression is selected as the best model by F1-score (0.1205), while Random Forest achieves the highest accuracy (0.7044). Accuracy alone is not a sufficient selection criterion because the non-returned class dominates the dataset. Precision is low for all models, indicating that many orders predicted as returns are not actual returns. Logistic Regression has the highest recall among the reported models (0.5741), meaning it identifies a larger share of returned orders than the other tested classifiers.

The ROC-AUC values range from approximately 0.557 to 0.582. These values indicate limited discrimination, although they are above 0.5. The results suggest that return-risk prediction remains difficult using the selected features and available dataset.

## **7.5 Prediction Outputs and Error Analysis**

The notebook generates a prediction-results table containing actual return-risk labels, predicted labels, and predicted probabilities. It also creates a high-risk order subset, a confusion matrix, a classification report, and an error analysis distinguishing correct predictions, false positives, and false negatives. The test split contains 6,926 records; the notebook reports 3,165 as predicted high return risk and 3,761 as predicted low return risk.

## **7.6 Practical Use**

- Flag orders for additional review when their predicted return-risk probability is high.
- Review false positives to avoid unnecessary interventions for customers whose orders would not be returned.
- Monitor recall and precision together when deciding how the model should be used operationally.

# **8. MARKET-BASKET ANALYSIS**

## **8.1 Transaction Basket Preparation**

The market-basket analysis groups product categories by order_id and removes duplicate order-category pairs. It prepares 34,500 transaction baskets. The displayed sample baskets each contain one category, such as Home, Grocery, or Electronics.

## **8.2 Apriori and FP-Growth**

The notebook applies Apriori and FP-Growth with a minimum support threshold of 0.02. Both algorithms return seven frequent itemsets, all of which are single-category itemsets.

| Category    | Support  |
| ----------- | -------- |
| Fashion     | 0.181275 |
| Electronics | 0.179130 |
| Home        | 0.159043 |
| Toys        | 0.123101 |
| Sports      | 0.120899 |
| Beauty      | 0.118928 |
| Grocery     | 0.117623 |

## **8.3 Association Rules**

Association rules are generated using a minimum confidence threshold of 0.30. Both Apriori and FP-Growth return empty rule tables. The notebook subsequently applies stronger validation filters for support, confidence, and lift, but no qualifying global or segment-specific rules are reported.

This outcome is consistent with the observed basket structure: the transaction baskets shown in the notebook contain one distinct category per order. With no co-occurring categories in a basket, the current dataset representation cannot produce meaningful multi-category antecedent-consequent rules. The project successfully demonstrates frequent-itemset generation, but cross-category product recommendations remain unachieved in this run.

## **8.4 Future Improvement for Basket Analysis**

If the original data contains multiple product lines per order, the order-level basket construction should retain all distinct categories within each order. If each order genuinely contains only one category, a richer order-line dataset with multiple products per basket will be required before cross-selling rules can be discovered.

# **9. OVERALL RESULTS AND DISCUSSION**

| Project area               | Verified output                                             |
| -------------------------- | ----------------------------------------------------------- |
| Data preparation           | 34,500 rows; 21 columns after cleaning; zero nulls reported |
| Warehouse preparation      | 1 fact table and 4 dimension tables exported as CSV         |
| Customer features          | 7,903 customer profiles                                     |
| Customer segmentation      | 2 K-Means clusters; 4,814 and 3,089 customers               |
| Clustering quality         | Best tested silhouette score 0.170969 for k=2               |
| Return-risk classification | Logistic Regression highest F1-score: 0.1205                |
| Highest accuracy           | Random Forest: 0.704447                                     |
| Market-basket itemsets     | 7 itemsets each from Apriori and FP-Growth                  |
| Association rules          | No qualifying global rules generated                        |

The project demonstrates a complete analytical preparation workflow from raw transaction data through dimensional table construction and downstream data mining. The most useful implemented outputs are the fact/dimension CSVs, customer segments and profiles, model comparison, order-level return-risk predictions, and association-rule export files.

The two principal analytical cautions are the weak/modest clustering separation and the limited classification precision/ROC-AUC. In addition, the market-basket analysis does not identify multi-category rules in the current data representation. These results are important findings and should be reported transparently rather than overstated.

# **10. PROJECT LIMITATIONS AND CHALLENGES**

## **10.1 Limitations**

1. The dataset source, provider, and exact collection methodology are not identified in the submitted notebook or README.
2. The notebook prepares Snowflake-ready CSV tables but does not show a live Snowflake connection, SQL table creation, or data loading execution.
3. The classification target is order return status, not actual customer churn.
4. The target is imbalanced, with returned orders representing 5.52% of the dataset; this affects model performance and interpretation.
5. Classification precision and F1-scores are low, and ROC-AUC values indicate limited predictive discrimination.
6. K-Means silhouette scores are low, indicating modest cluster separation.
7. Association-rule generation returns no rules because the observed order baskets contain only one distinct category.
8. Association rules, even when available, describe associations and do not establish causation.
9. Results depend on the quality, coverage, and representativeness of the available transaction data.

## **10.2 Challenges**

- Ensuring consistent data types and standard formats across customer, product, category, payment, region, and date fields.
- Applying business-rule validation without unnecessarily removing valid transactions.
- Creating surrogate keys and joining transaction rows to multiple dimension tables.
- Selecting customer behavioural variables and scaling them appropriately for clustering.
- Handling an imbalanced return-risk target and selecting evaluation metrics beyond accuracy.
- Interpreting the lack of association rules and identifying the basket-granularity limitation.

# **11. CONCLUSION AND RECOMMENDATIONS**

## **11.1 Conclusion**

The Omnichannel Customer Risk, Churn & Basket Intelligence Platform demonstrates how retail transaction data can be prepared for analytical warehousing and data mining. The project cleans and standardizes 34,500 transaction records, derives financial measures, constructs customer/product/channel-region/date dimensions, and builds a sales fact table. These five warehouse tables are exported in CSV format for a planned Snowflake loading stage.

The data-mining workflow develops customer-level RFM and behavioural features, identifies two customer segments through K-Means, and compares clustering with hierarchical clustering. It also trains four classifiers to predict order-return risk. Logistic Regression provides the best F1-score, while Random Forest provides the highest accuracy. Apriori and FP-Growth each identify seven frequent category itemsets, but no association rules are generated under the current basket structure.

Overall, the project provides a practical foundation for retail analytics and demonstrates both the value and limitations of the implemented methods. The results support further development of the warehouse deployment, predictive features, and transaction basket representation.

## **11.2 Recommendations**

1. Complete the Snowflake implementation by creating the database, schema, fact/dimension tables, and controlled CSV load process; validate row counts and key relationships after loading.
2. Record the dataset name, provider, URL, license, and extraction date in the project documentation.
3. Use a time-aware validation strategy where possible to test whether customer and return-risk patterns generalize to later transactions.
4. Investigate additional return-related predictors and evaluate threshold tuning, class-weight strategies, and precision-recall curves.
5. Report confusion matrices and class-specific metrics alongside accuracy.
6. Review the number of clusters and feature selection with business interpretability in mind; consider alternative clustering methods and outlier treatment.
7. Retain multiple product lines per order for market-basket analysis, or obtain a dataset with multi-product baskets so that cross-category rules can be generated.
8. Validate the assigned customer personas with business stakeholders before using them for customer targeting.

## **11.3 Future Enhancements**

- Implement and document a live Snowflake ingestion and SQL-based analytical layer.
- Build BI dashboards in Power BI or Tableau using the fact and dimension tables.
- Develop customer churn labels from an explicit inactivity definition if churn prediction is required.
- Improve return-risk prediction with richer behavioural and operational data.
- Generate segment-specific product recommendations after obtaining suitable multi-item baskets.
- Automate scheduled data refreshes, data-quality checks, and model monitoring.

# **12. REFERENCES AND APPENDICES**

## **12.1 Project Materials**

**Primary implementation source:** DATAWAREHOUSINGPROJECT Jupyter Notebook supplied with this report.

**Project documentation:** README.DOCX supplied with this report.

**Dataset citation:** Not provided in the supplied files. Insert the actual dataset source and URL before final academic submission.

## **Appendix A: Warehouse Table Summary**

| CSV export             | Purpose                             |
| ---------------------- | ----------------------------------- |
| dim_customer.csv       | Customer dimension                  |
| dim_product.csv        | Product dimension                   |
| dim_channel_region.csv | Region and payment-method dimension |
| dim_date.csv           | Date dimension                      |
| fact_sales.csv         | Sales transaction fact table        |

## **Appendix B: Analytical Output Files**

| Output file                         | Contents                                              |
| ----------------------------------- | ----------------------------------------------------- |
| customer_segments.csv               | Customer cluster, persona, and behavioural features   |
| cluster_profiles.csv                | Average feature profile by cluster                    |
| classification_model_comparison.csv | Classification model metrics                          |
| return_risk_predictions.csv         | Actual/predicted return-risk labels and probabilities |
| global_association_rules.csv        | Global association-rule output                        |
| segment_association_rules.csv       | Segment-specific association-rule output              |
| validated_global_rules.csv          | Global rules passing validation filters               |
| validated_segment_rules.csv         | Segment rules passing validation filters              |

## **Appendix C: Group Members**

| Group member   | Student ID |
| -------------- | ---------- |
| Fatima Ahmer   | f2023-300  |
| Mahnoor Nadeem | f2023-164  |

**Course:** Data Warehousing and Data Mining\
**Instructor:** Sir Mohammad Farooq\
**University:** Beaconhouse National University (BNU)
