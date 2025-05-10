# Automated CSV Data Pipeline with AWS S3, Lambda, Glue, and QuickSight

##  Project Overview

This project sets up an automated data pipeline using AWS services to process, transform, and visualize CSV data. It leverages the power of **Amazon S3**, **AWS Lambda**, **AWS Glue**, and **Amazon QuickSight** to build a serverless and scalable data flow.

---

##  Architecture Summary

1. **Amazon S3**: Stores the data at various pipeline stages.
2. **AWS Lambda**: Triggers on file uploads, performs data cleaning.
3. **AWS Glue**: Executes ETL (Extract, Transform, Load) jobs.
4. **Amazon QuickSight**: Provides BI dashboards and visualizations.

---

##  S3 Buckets Structure

Set up the following S3 buckets:

| Bucket Name         | Purpose                              |
|---------------------|--------------------------------------|
| `raw-data`          | Stores raw incoming CSV files        |
| `processed-data`    | Contains cleaned CSV files           |
| `transformed-data`  | Holds transformed datasets for BI    |

---

##  IAM Roles and Permissions

Create two IAM roles with specific permissions:

### 1. Lambda Execution Role

- **Permissions**:
  - `s3:GetObject`, `s3:PutObject` on `raw-data` and `processed-data`
  - `logs:*` for logging

- **Trust Relationship**:
  - Service: `lambda.amazonaws.com`

### 2. Glue Execution Role

- **Permissions**:
  - `s3:GetObject`, `s3:PutObject` on `processed-data` and `transformed-data`
  - `glue:*`, `logs:*`, `cloudwatch:*`

- **Trust Relationship**:
  - Service: `glue.amazonaws.com`
- **Policy**:
  ```JSON
  {
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:PutObject",
                "s3:DeleteObject"
            ],
            "Resource": [
                "arn:aws:s3:::your-initials-processed-data-pipeline/*",
                "arn:aws:s3:::your-initials-transformed-data-pipeline/*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": "s3:ListBucket",
            "Resource": [
                "arn:aws:s3:::your-initials-processed-data-pipeline",
                "arn:aws:s3:::your-initials-transformed-data-pipeline"
            ]
        }
    ]
  }
  ```

---

##  AWS Lambda Setup

1. **Trigger**: Configure the Lambda function to trigger on `s3:ObjectCreated:*` for the `raw-data` bucket.
2. **Function Logic**:
   - Load the new CSV file
   - Clean the data (e.g., remove empty rows, standardize headers)
   - Write the cleaned file to the `processed-data` bucket

3. **Language**: Python (or Node.js)

```python
# Example: Basic Lambda pseudocode
def lambda_handler(event, context):
    raw_file = get_csv_from_s3('raw-data')
    cleaned_data = clean_csv(raw_file)
    upload_to_s3(cleaned_data, 'processed-data')
```

---

##  AWS Glue ETL Job

1. **Job Type**: Spark (Python or Scala)
2. **Source**: S3 `processed-data` bucket
3. **Transformations**:
   - Schema normalization
   - Calculations or joins (if needed)
   - Data type corrections
4. **Destination**: Write to `transformed-data` bucket in Parquet or CSV format

5. **Crawler (optional)**: Use a Glue Crawler to catalog data into AWS Glue Data Catalog.

---

##  Amazon QuickSight Setup

1. **Connect to S3**:
   - Set up a manifest file or use Athena if querying Parquet
   - Grant QuickSight permission to access `transformed-data`

2. **Create Dataset**:
   - Import transformed data into QuickSight
   - Create calculated fields if necessary

3. **Build Visualizations**:
   - Charts, tables, KPIs, etc.
   - Build dashboards using key metrics

---

##  Final Flow Summary

```text
[raw-data (S3)] 
   │
   └─▶ [Lambda Cleaning]
           │
           ▼
[processed-data (S3)]
   │
   └─▶ [Glue ETL Transformations]
           │
           ▼
[transformed-data (S3)]
   │
   └─▶ [QuickSight Visualization]
```

---

##  Notes

- Ensure bucket policies allow access between services
- Set up CloudWatch logging for debugging Lambda and Glue jobs
- Secure data using proper encryption (S3 SSE or KMS)

---

##  Benefits

- Serverless and scalable
- Low operational overhead
- Fully integrated AWS analytics stack

---

##  Useful Links

- [AWS Lambda Documentation](https://docs.aws.amazon.com/lambda/)
- [AWS Glue Documentation](https://docs.aws.amazon.com/glue/)
- [Amazon QuickSight Documentation](https://docs.aws.amazon.com/quicksight/)
