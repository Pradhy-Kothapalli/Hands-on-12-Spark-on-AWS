# ITCS 6190 - Hands-on-12-Spark-on-AWS

---

## Approach

### 1. S3 Setup
- Created two S3 buckets:
  - handsonfinallanding-us-east-1-an (landing/input)
  - handsonfinalprocessed (processed/output)
- Uploaded reviews.csv into the landing bucket

![S3 setup Screenshot](Screenshot%202026-04-14%231826.png)

---

### 2. IAM Role
- Created IAM role: AWSGlueServiceRole-Reviews
- Attached policies:
  - AWSGlueServiceRole
  - AmazonS3FullAccess
- Updated bucket policy to allow Glue role to access S3 buckets


---

### 3. AWS Glue Job
- Created Glue job: process_reviews_job
- Used PySpark script to:
  - Read CSV data from S3
  - Perform transformations
  - Run Spark SQL queries
  - Write results back to S3

#### Transformations:
- Cast rating to integer and replace null values with 0
- Convert review_date to date format
- Fill missing review_text with default value
- Convert product_id to uppercase


---

### 4. Lambda Trigger
- Created Lambda function: start_glue_job_trigger
- Function starts Glue job using boto3:

```python
glue_client.start_job_run(JobName="process_reviews_job")
```

- Added IAM permission:
  - glue:StartJobRun
- Configured S3 trigger:
  - Event type: ObjectCreated
  - Bucket: landing bucket

---

### 5. Data Flow

S3 (Upload) -> Lambda (Trigger) -> AWS Glue (Spark Job) -> S3 (Processed Results)
---

### 6. Spark SQL Queries

#### 1. Average Rating per Product
```sql
SELECT 
    product_id_upper, 
    AVG(rating) as average_rating,
    COUNT(*) as review_count
FROM product_reviews
GROUP BY product_id_upper
ORDER BY average_rating DESC;
```

#### 2. Date-wise Review Count
```sql
SELECT 
    review_date,
    COUNT(*) as total_reviews
FROM product_reviews
GROUP BY review_date
ORDER BY review_date;
```

#### 3. Top 5 Most Active Customers
```sql
SELECT 
    customer_id,
    COUNT(*) as review_count
FROM product_reviews
GROUP BY customer_id
ORDER BY review_count DESC
LIMIT 5;
```

#### 4. Rating Distribution
```sql
SELECT 
    rating,
    COUNT(*) as count
FROM product_reviews
GROUP BY rating
ORDER BY rating;
```

---

## Results

- Built a fully automated serverless ETL pipeline
- Data ingestion triggered automatically via S3 upload
- Glue processed and transformed data using Spark
- Analytical results stored in S3 for further use (Athena-ready)
![S3 results Screenshot](Screenshot%202026-04-14%231138.png)
![S3 results Screenshot](Screenshot%202026-04-14%231112.png)

---

## Issues Faced

- Encountered S3 AccessDenied (403) error
- Resolved by:
  - Correcting bucket name in Glue script
  - Fixing bucket policy with correct ARN
  - Ensuring Glue IAM role had proper permissions
