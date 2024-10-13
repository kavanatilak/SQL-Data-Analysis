# **Case Study: Analyzing Customer Churn in a Telecom Company Using SQL**

1. # **Introduction**

   ## **Objective:**

   Analyze the customer churn data of a telecom company to identify key factors influencing customer retention and provide actionable insights to reduce churn rates

   ## **Goals:**

* Understand the demographics and behaviors of customers who churn  
* Identify patterns and trends associated with churn  
* Develop strategies to improve customer retention based on data-driven insights

2. # **Data Overview**

   ## **Data Columns:** 

   customerID, gender, SeniorCitizen, Partner, Dependents, tenure, PhoneService, MultipleLines, InternetService, OnlineSecurity, OnlineBackup, DeviceProtection, TechSupport, StreamingTV, StreamingMovies, Contract, PaperlessBilling, PaymentMethod, MonthlyCharges, TotalCharges, numAdminTickets, numTechTickets, Churn

   ## **Data Attributes:**

* **Demographics:** customerID, gender, SeniorCitizen, Partner, Dependents  
* **Services:** PhoneService, MultipleLines, InternetService, OnlineSecurity, OnlineBackup, DeviceProtection, TechSupport, StreamingTV, StreamingMovies  
* **Contracts and Billing:** Contract, PaperlessBilling, PaymentMethod  
* **Financials:** MonthlyCharges, TotalCharges  
* **Support Tickets:** numAdminTickets, numTechTickets  
* **Outcome:** Churn (Yes/No)

3. # **Key Performance Indicators (KPIs) and Metrics:**

* **Churn Rate:** Percentage of customers who have churned over a specific period

  *Churn Rate \= (Number of churned customers/Total number of customers)\*100*

  **Result:** 26.54%

  **Query:**

  SELECT

  ROUND((SUM(CASE WHEN Churn \= 'Yes' THEN 1 ELSE 0 END)/

  COUNT(\*) \*100), 2\) AS 'Churn\_rate'

  FROM telecom\_churn;

* **Average tenure:** Average duration (in months) customers stay with the company

  **Result:**

| Churn | Average\_tenure |
| :---: | :---: |
| No | 37.57 |
| Yes | 17.98 |


  **Query:**

  SELECT Churn, ROUND(AVG(tenure), 2\) AS 'Average\_tenure'

  FROM telecom\_churn

  GROUP BY Churn;

* **Average Monthly revenue:** Average monthly revenue from all customers

  **Result:** 64.76

  **Query:**

  SELECT ROUND(AVG(MonthlyCharges), 2\) AS 'Average\_monthly\_charges'

  FROM telecom\_churn;


* **Customer Lifetime Value (CLV):** Projected revenue a customer will generate during their relationship with the company

  *CLV \= ARPU \* Customer Lifetime*

  *Where ARPU: Average monthly revenue from the customer*

  *Customer Lifetime: The expected number of months the customer will remain a customer*

  **Result:**

| Avg\_tenure | Avg\_LT | Avg\_CLV |
| :---: | :---: | :---: |
| 32.3711 | 7.4919 | 573.1361 |


  **Approach:**

  Step 1: Calculating ARPU

  Same as MonthlyRevenue for each customer


  Step 2: Calculating Churn probability

  SELECT ROUND(COUNT(\*)/(SELECT COUNT(\*) FROM telecom\_churn), 2\) AS 'Churn\_probability'

  FROM telecom\_churn

  WHERE Churn \= 'Yes';


  Step 3: Calculating Customer lifetime, which is the reciprocal of the churn rate

  SELECT 1/COUNT(\*)/(SELECT COUNT(\*) FROM telecom\_churn) AS 'Customer\_lifetime'

  FROM telecom\_churn

  WHERE Churn \= 'Yes';


  Step 4: Bringing it all together

  WITH Churn\_Stats AS(SELECT ROUND(COUNT(\*)/(SELECT COUNT(\*) FROM telecom\_churn), 2\) AS 'Churn\_probability'

  FROM telecom\_churn

  WHERE Churn \= 'Yes')

  SELECT customerId, MonthlyCharges, tenure,

  CASE WHEN Churn \= 'Yes' THEN tenure

  ELSE 1.0/Churn\_probability END AS 'Estimated\_lifetime',

  MonthlyCharges \* CASE WHEN Churn \= 'Yes' THEN tenure

  ELSE 1.0/Churn\_probability END AS 'CLV'

  FROM telecom\_churn CROSS JOIN Churn\_Stats;


		**Query:**  
WITH Churn\_Stats AS(SELECT ROUND(COUNT(\*)/(SELECT COUNT(\*) FROM telecom\_churn), 2\) AS 'Churn\_probability'  
FROM telecom\_churn  
WHERE Churn \= 'Yes'),  
CTE AS(SELECT customerId, MonthlyCharges, tenure,  
CASE WHEN Churn \= 'Yes' THEN tenure  
ELSE 1.0/Churn\_probability END AS 'Estimated\_lifetime',  
MonthlyCharges \* CASE WHEN Churn \= 'Yes' THEN tenure  
ELSE 1.0/Churn\_probability END AS 'CLV'  
FROM telecom\_churn CROSS JOIN Churn\_Stats)  
SELECT AVG(tenure) AS 'Avg\_tenure', AVG(Estimated\_lifetime) AS 'Avg\_LT', AVG(CLV) AS 'Avg\_CLV'  
FROM CTE;

* **Service Utilization Metrics:**  
  * **Phone Service Utilization:** Percentage of customers using phone services

    **Result:** 90.32%

    **Query:**

    SELECT

    ROUND((SUM(CASE WHEN PhoneService \= 'Yes' THEN 1 ELSE 0 END)/

    COUNT(\*) \* 100), 2\) AS 'PhoneService\_utilization'

    FROM telecom\_churn;

  * **Internet Service Utilization:** Distribution of different internet services

    **Result:**

| InternetService | distribution\_percentage |
| :---: | :---: |
| FiberOptic | 43.96 |
| DSL | 34.37 |
| No | 21.67 |

    **Query:**

    **With CTE:**

    WITH CTE AS(SELECT InternetService, COUNT(\*) AS 'distribution\_count'

    FROM telecom\_churn

    GROUP BY InternetService)

    SELECT CTE.InternetService, ROUND(CTE.distribution\_count/COUNT(\*)\*100, 2\) AS 'distribution\_percentage'

    FROM CTE, telecom\_churn

    GROUP BY CTE.InternetService

    ORDER BY distribution\_percentage DESC;

    **Without CTE:**

    SELECT InternetService,

    ROUND((COUNT(\*)/(SELECT COUNT(\*) FROM telecom\_churn))\*100, 2\) AS 'distribution\_percentage'

    FROM telecom\_churn

    GROUP BY InternetService

    ORDER BY distribution\_percentage DESC;

* **Support Ticket Analysis:**  
  * **Average number of admin tickets**

    **Result:** 0.52

    **Query:**

    SELECT ROUND(AVG(numAdminTickets), 2\) AS 'Average\_admin\_tickets'

    FROM telecom\_churn;

  * **Average number of tech tickets**

    **Result:** 0.42

    **Query:**

    SELECT ROUND(AVG(numTechTickets), 2\) AS 'Average\_tech\_tickets'

    FROM telecom\_churn;

* **Demographic Breakdown:**  
  * **Churn rate by gender**

    **Result:**

| Gender | Churn\_rate |
| :---: | :---: |
| Female | 26.92 |
| Male | 26.16 |

    **Query:**

    SELECT gender,

    ROUND((SUM(CASE WHEN Churn \= 'Yes' THEN 1 ELSE 0 END)/

    COUNT(\*) \*100), 2\) AS 'Churn\_rate'

    FROM telecom\_churn

    GROUP BY gender

    ORDER BY Churn\_rate DESC;

  * **Churn rate by senior citizen status**

    **Result:**

| SeniorCitizen | Churn\_rate |
| :---: | :---: |
| 1 | 41.68 |
| 0 | 23.61 |

    **Query:**

    SELECT SeniorCitizen,

    ROUND((SUM(CASE WHEN Churn \= 'Yes' THEN 1 ELSE 0 END)/

    COUNT(\*) \*100), 2\) AS 'Churn\_rate'

    FROM telecom\_churn

    GROUP BY SeniorCitizen

    ORDER BY Churn\_rate DESC;

  * **Churn rate by partnership status**

    **Result:**

| Partner | Churn\_rate |
| :---: | :---: |
| No | 32.96 |
| Yes | 19.66 |

    **Query:**

    SELECT Partner,

    ROUND((SUM(CASE WHEN Churn \= 'Yes' THEN 1 ELSE 0 END)/

    COUNT(\*) \*100), 2\) AS 'Churn\_rate'

    FROM telecom\_churn

    GROUP BY Partner

    ORDER BY Churn\_rate DESC;

* **Contract Analysis:**  
  * **Churn rate by contract type**

    **Result:**

    

| Contract | Churn\_rate |
| :---: | :---: |
| Month-to-month | 42.71 |
| One year | 11.27 |
| Two year | 2.83 |

    **Query:**

    SELECT Contract,

    ROUND((SUM(CASE WHEN Churn \= 'Yes' THEN 1 ELSE 0 END)/

    COUNT(\*) \*100), 2\) AS 'Churn\_rate'

    FROM telecom\_churn

    GROUP BY Contract

    ORDER BY Churn\_rate DESC;

* **Payment Method Preferences:**  
  * **Churn rate by payment method**

    **Result:**

| PaymentMethod | Churn\_rate |
| :---: | :---: |
| Electronic check | 45.29 |
| Mailed check | 19.11 |
| Bank transfer (automatic) | 16.71 |
| Credit card (automatic) | 15.24 |

    **Query:**

    SELECT PaymentMethod,

    ROUND((SUM(CASE WHEN Churn \= 'Yes' THEN 1 ELSE 0 END)/

    COUNT(\*) \*100), 2\) AS 'Churn\_rate'

    FROM telecom\_churn

    GROUP BY PaymentMethod

    ORDER BY Churn\_rate DESC;

4. # **Business Questions**

* **Is there a correlation between tenure and churn**

  **Result:** Yes, it was observed that lower tenure had the highest churn rate

  The top 10 values of tenure having the highest churn rate are:


| tenure | Churn\_rate | Churn\_rank |
| :---: | :---: | :---: |
| 1 | 61.99 | 1 |
| 2 | 51.68 | 2 |
| 5 | 48.12 | 3 |
| 4 | 47.16 | 4 |
| 3 | 47.00 | 5 |
| 7 | 38.93 | 6 |
| 10 | 38.79 | 7 |
| 9 | 38.66 | 8 |
| 15 | 37.37 | 9 |
| 6 | 36.36 | 10 |

  **Query:**

  WITH CTE AS(SELECT tenure,

  ROUND((SUM(CASE WHEN Churn \= 'Yes' THEN 1 ELSE 0 END)/

  COUNT(\*) \*100), 2\) AS 'Churn\_rate'

  FROM telecom\_churn

  GROUP BY tenure)

  SELECT tenure, Churn\_rate,

  DENSE\_RANK()OVER(ORDER BY Churn\_rate DESC) AS Churn\_rank

  FROM CTE;

* **Average number of tech support tickets for churned vs. retained customers**

  **Result:**

| Churn | Average\_tech\_tickets |
| :---: | :---: |
| Yes | 1.2 |
| No | 0.2 |

  **Query:**

  SELECT Churn, ROUND(AVG(numTechTickets), 1\) AS Average\_tech\_tickets

  FROM telecom\_churn

  GROUP BY Churn

  ORDER BY Average\_tickets DESC;

* **Average number of admin support tickets for churned vs. retained customers**

  **Result:**


| Churn | Average\_admin\_tickets |
| :---: | :---: |
| Yes | 0.5 |
| No | 0.5 |

  **Query:**

  SELECT Churn, ROUND(AVG(numAdminTickets), 1\) AS Average\_admin\_tickets

  FROM telecom\_churn

  GROUP BY Churn

  ORDER BY Average\_admin\_tickets DESC;

* **Which payment method collected the most revenue?**

  **Result:** Electronic check

  **Query:**

  SELECT PaymentMethod, ROUND(SUM(TotalCharges), 2\) AS Total\_charges

  FROM telecom\_churn

  GROUP BY PaymentMethod

  ORDER BY Total\_charges DESC;

* What combination of services increases the likelihood of churn?

  Result: The following combination has the highest likelihood of churn (approx 2%)

  PhoneService \= ‘Yes’, MultipleLines \= ‘No’, InternetService \= ‘Fibre Optic’, OnlineSecurity \= ‘No’, OnlineBackup \= ‘No’, DeviceProtection \= ‘No’, TechSupport \= ‘No’, StreamingTV \= ‘No’, StreamingMovies \= ‘No’

  **Query:**

  SELECT PhoneService, MultipleLines, InternetService, OnlineSecurity, OnlineBackup, DeviceProtection, TechSupport, StreamingTV, StreamingMovies,

  ROUND(COUNT(\*)\*1.0/(SELECT COUNT(\*) FROM telecom\_churn)\*100, 2\) AS 'Churn\_probability'

  FROM telecom\_churn

  WHERE Churn \= 'Yes'

  GROUP BY PhoneService, MultipleLines, InternetService, OnlineSecurity, OnlineBackup, DeviceProtection, TechSupport, StreamingTV, StreamingMovies

  ORDER BY Churn\_probability DESC

  LIMIT 1;

* What combination of services decreases the likelihood of churn?

  Result: The following combinations have the highest likelihood of customer retention

* PhoneService \= ‘Yes’, MultipleLines \= ‘No’, InternetService \= ‘No’, OnlineSecurity \= ‘No internet service’, OnlineBackup \= ‘No internet service’, DeviceProtection \= ‘No internet service’, TechSupport \= ‘No internet service’, StreamingTV \= ‘No internet service’, StreamingMovies \= ‘No internet service’  
* PhoneService \= ‘Yes’, MultipleLines \= ‘Yes’, InternetService \= ‘No’, OnlineSecurity \= ‘No internet service’, OnlineBackup \= ‘No internet service’, DeviceProtection \= ‘No internet service’, TechSupport \= ‘No internet service’, StreamingTV \= ‘No internet service’, StreamingMovies \= ‘No internet service’  
* PhoneService \= ‘Yes’, MultipleLines \= ‘Yes’, InternetService \= ‘Fibre Optic’, OnlineSecurity \= ‘Yes’, OnlineBackup \= ‘Yes’, DeviceProtection \= ‘Yes’, TechSupport \= ‘Yes’, StreamingTV \= ‘Yes’, StreamingMovies \= ‘Yes’

  **Query:**

  SELECT PhoneService, MultipleLines, InternetService, OnlineSecurity, OnlineBackup, DeviceProtection, TechSupport, StreamingTV, StreamingMovies,

  ROUND(COUNT(\*)\*1.0/(SELECT COUNT(\*) FROM telecom\_churn)\*100, 2\) AS 'Non\_Churn\_probability'

  FROM telecom\_churn

  WHERE Churn \= 'No'

  GROUP BY PhoneService, MultipleLines, InternetService, OnlineSecurity, OnlineBackup, DeviceProtection, TechSupport, StreamingTV, StreamingMovies

  ORDER BY Non\_Churn\_probability DESC

  LIMIT 3;

* What is the average Customer Lifetime Value (CLV) for churned vs. retained customers

  **Result:**

| Churn | CLV |
| :---: | :---: |
| No | 2549.91 |
| Yes | 1531.8 |

  **Query:**

  SELECT Churn, ROUND(AVG(TotalCharges), 2\) AS 'CLV'

  FROM telecom\_churn

  GROUP BY Churn

  ORDER BY CLV DESC;

5. # **Analysis and Insights**

   The following insights were observed after the analysis:  
* **High churn segments:** The churn rate is higher for customers with a small duration for tenure, who have electronic and mail checks as a payment method, and who have taken month-to-month contract type instead of one or two years of contract length. 

* **Service influence:** It was observed that customers who took only phone service and no internet services, customers who took only phone and multiple-line service and no internet services, and customers who took both phone and multiple-line service along with all the internet services had a very low churn rate.

* **Support interactions:** A higher number of tech support tickets is positively correlated with a higher probability of customer churn.

6. # **Recommendations**

   The steps that can be taken to increase customer retention are:  
* **Enhance customer engagement:** Focusing on customers who have taken month-to-month contract type with targeted strategies like loyalty programs or contract incentives. 

* **Packaged services:** Develop tiered service packages, bundling multiple offerings for customers at a fixed price.

* **Improve service quality:** Address common tech issues reported in support tickets to reduce customer frustration and potential churn.