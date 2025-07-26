# Automated_Data_Pipeline_with_CICD_on_GCP
This project simulates a real-world industrial data pipeline that processes **raw flight booking data** to deliver actionable business insights. It leverages **Apache Airflow** for orchestration, **PySpark on Google Cloud Dataproc Serverless** for scalable and distributed data processing, and **BigQuery** for analytics and reporting.

To ensure consistency and reliability across environments, the pipeline includes robust **CI/CD automation using GitHub Actions**, enabling seamless and automated deployments to both **development** and **production** environments.

---

## Data Architecture

![Data Architecture](/Data_Architecture.jpg)


- The pipeline is triggered by an Apache Airflow DAG running on Cloud Composer, initiated by a file sensor that detects new CSV uploads in Google Cloud Storage.

- Once triggered, the DAG launches a PySpark job on Dataproc Serverless, which processes and enriches the flight booking data.

- The transformed data is then loaded into the appropriate BigQuery tables for analytics and reporting.

- The entire workflow is integrated with CI/CD automation using GitHub Actions, enabling seamless deployment and version control across development and production environments.

---

## Tech Stack Used

| Component                        | Purpose                                                                 |
|----------------------------------|-------------------------------------------------------------------------|
| **Google Cloud Composer (Airflow)** | Orchestrate the pipeline (file arrival → Spark job → BigQuery)         |
| **Google Cloud Dataproc (Serverless)** | Run PySpark jobs to transform and process the flight booking data       |
| **Google Cloud Storage (GCS)**       | Store raw CSV files and Spark job files              |
| **Google Cloud BigQuery (BQ)**       | Store the final processed and aggregated data for business insights     |
| **GitHub Actions (CI/CD)**           | Automate deployment of DAGs and Spark jobs based on branch push (dev → DEV, main → PROD) |
| **Python / PySpark**                | Data processing, transformation, and aggregation                        |

---

## Data Pipeline Flow

Here’s how the data flows through the pipeline:

### 📥 Data Ingestion
- A CSV file (`flight_booking.csv`) is uploaded to **Google Cloud Storage (GCS)**.
- An **Airflow DAG** waits for the file using `GCSObjectExistenceSensor`.

### ⚙️ Data Processing (Spark Job)
- Once the file is detected, Airflow triggers a **Dataproc Serverless PySpark job**.
- The Spark job reads the data from GCS, performs transformations.

### 🗃️ Data Storage (BigQuery)
Transformed data are written to **BigQuery** in separate tables:
- ✅ **Transformed Table**: Contains cleaned and enriched flight booking data.
- ✅ **Route Insights Table**: Shows insights on popular routes, average flight duration, etc.
- ✅ **Origin Insights Table**: Provides insights based on the origin of the booking.

### 🚀 Deployment Automation (CI/CD)
- `dev` branch → Deploys to **DEV** environment  
- `main` branch → Deploys to **PROD** environment

---

## **Files Structure**  

The repository has the following structure:  

```
.
│
├── airflow_job/
│   ├── airflow_job.py                <-- Airflow DAG to orchestrate the pipeline
│
├── spark_job/
│   ├── spark_transformation_job.py    <-- PySpark script for data processing
│
├── variables/
│   ├── dev/variables.json            <-- Environment variables for DEV
│   ├── prod/variables.json           <-- Environment variables for PROD
│
├── .github/workflows/
│   ├── ci-cd.yaml                    <-- GitHub Actions for CI/CD deployment
│
├── README.md                         <-- This file
```

---

## **Environment Setup**  

### Step 1: Clone the Repository  
```bash
git clone <repository-url>
cd flight-booking-data-pipeline
```

### Step 2: Create Environment Variables in GitHub  
In your **GitHub repository**, go to:  
`Settings` → `Secrets and variables` → `Actions` → Add the following secrets:  

| Secret Name               | Description                                       |  
|--------------------------|---------------------------------------------------|  
| **GCP_SA_KEY**            | Service account key JSON from GCP.                |  
| **GCP_PROJECT_ID**        | Your GCP project ID.                              |  


### Step 3: Initial Setup: Enable Required APIs

When using **BigQuery** or **Cloud Composer (Airflow)** for the first time in your GCP project, you must enable specific Google Cloud APIs to allow these services to function properly.

### ✅ Required APIs:

- **BigQuery API** – Required to create and manage datasets, tables, and run SQL queries.
- **Cloud Composer API** – Required to create and manage Cloud Composer (Airflow) environments.

> Ensure these APIs are enabled before proceeding with pipeline development or environment creation to avoid setup errors.


### Step 4: Granting Permissions to Airflow Service Account

To ensure that Airflow (Cloud Composer) can operate seamlessly, you need to assign the necessary IAM roles to its associated **Service Account**.

#### Steps:

1. Go to **IAM & Admin** in the Google Cloud Console.
2. Open the **IAM** tab.
3. Locate the **Service Account** associated with your Airflow environment (Cloud Composer).
4. Click **Edit** on that service account.
5. Add the following **roles**:

### ✅ Required IAM Roles:

1. `Cloud Composer API Service Agent`
2. `Cloud Composer v2 API Service Agent Extension`
3. `Composer Administrator`
4. `Composer Shared VPC Agent`
5. `Composer User`
6. `Composer Worker`
7. `BigQuery Admin`
8. `Dataproc Administrator`
9. `Storage Admin`

---

## **Deployment Using CI/CD**  

The **CI/CD Pipeline (ci-cd.yaml)** is set up as:  

| Branch Push       | Deployment Environment       |  
|-------------------|-----------------------------|  
| **dev**           | Deploys to **DEV**           |  
| **main**          | Deploys to **PROD**          |  

---

## **Airflow DAG Overview**  

The **Airflow DAG** performs the following tasks:  

1. **Wait for File:** Uses `GCSObjectExistenceSensor` to detect when the file arrives.  
2. **Trigger Spark Job:** Runs the Spark job in **Dataproc Serverless**.  
3. **Load to BigQuery:** Pushes the transformed data to BigQuery tables.  

---

## **Data Processing Flow in Spark Job**  

The **Spark Job** (`spark_transformation_job.py`) does the following:  

1. ✅ **Read Data from GCS.**  
2. ✅ **Transform Data:**  
   - Add **is_weekend** column.  
   - Categorize lead time into **Last-Minute, Short-Term, Long-Term**.  
   - Calculate **booking_success_rate**.  
3. ✅ **Generate Insights:**  
   - Route Insights (top routes, avg stay, etc.)  
   - Origin Insights (origin success rate, etc.)  
4. ✅ **Write Data to BigQuery.**

---

## **How to Test the Pipeline**  

### **Trigger the DAG**  
In **Airflow UI**, trigger the DAG manually.  

### **Upload CSV File to GCS**  
Upload a sample CSV file to the GCS bucket:  
```
gs://airflow-projects-kv/airflow-project-1/source-dev/flight_booking.csv
```

### **Check BigQuery Tables**  
Run queries in BigQuery:  
```sql
SELECT * FROM `proven-record-452706-g5.flight_data_dev.transformed_table`;
SELECT * FROM `proven-record-452706-g5.flight_data_dev.route_insights_table`;
SELECT * FROM `proven-record-452706-g5.flight_data_dev.origin_insights_table`;
```

---

## **Troubleshooting**  

| Error                                              | Solution                                               |  
|---------------------------------------------------|--------------------------------------------------------|  
| **Airflow DAG not running**                      | Check if file path matches the GCS bucket.            |  
| **Spark Job failed**                             | View logs in Dataproc > Serverless > Job logs.        |  
| **BigQuery table empty**                         | Verify that Spark job completed without errors.       |  

---


## Conclusion

This project demonstrates a full-fledged data engineering pipeline for processing flight booking data with:

- **CI/CD Deployment** using GitHub Actions  
- **Airflow Orchestration** via Cloud Composer  
- **Serverless Spark Jobs** on Dataproc  
- **BigQuery Insights** for analytics and reporting

---

## Learnings

- Designed and implemented end-to-end ETL pipeline from scratch using **PySpark** and **BigQuery**  
- Gained practical experience in orchestrating **environment-specific workflows** with **Airflow** using parameterized DAGs  
- Learned how to set up and manage **CI/CD pipelines** for data engineering projects using **GitHub Actions**  
- Developed a solid understanding of integrating **GCP services** like Cloud Storage, Dataproc Serverless, and BigQuery  
- Built confidence in deploying **scalable, maintainable, and production-ready data pipelines**

---

### 🔗 Connect With Me

- [LinkedIn](https://www.linkedin.com/in/ishant-kumar-534989233)

 Tags
#GoogleCloud #CloudComposer #Airflow #Dataproc #Serverless #PySpark #BigQuery #GCS #DataEngineering #CICD #GitHubActions #DataPipeline #CloudAutomation #GCP #ETL

