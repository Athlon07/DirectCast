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
 
- **LambdaS3Access Policy**:
  ```JSON
  {
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": [
                "arn:aws:s3:::your-initials-raw-data-pipeline/*",
                "arn:aws:s3:::your-initials-processed-data-pipeline/*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": "s3:ListBucket",
            "Resource": [
                "arn:aws:s3:::your-initials-raw-data-pipeline",
                "arn:aws:s3:::your-initials-processed-data-pipeline"
            ]
        }
    ]
  }
  ```
### 2. Glue Execution Role

- **Permissions**:
  - `s3:GetObject`, `s3:PutObject` on `processed-data` and `transformed-data`
  - `glue:*`, `logs:*`, `cloudwatch:*`

- **Trust Relationship**:
  - Service: `glue.amazonaws.com`
- **GlueS3Access Policy**:
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
import boto3
import os

s3 = boto3.client('s3')
processed_bucket_name = os.environ['PROCESSED_BUCKET_NAME']

def lambda_handler(event, context):
    for record in event['Records']:
        bucket_name = record['s3']['bucket']['name']
        key = record['s3']['object']['key']

        print(f"Processing file: {key} from bucket: {bucket_name}")

        try:
            # Download the file from the raw bucket
            response = s3.get_object(Bucket=bucket_name, Key=key)
            file_content = response['Body'].read().decode('utf-8')

            # Simple data cleaning: Convert to uppercase
            processed_content = file_content.upper()

            # Upload the processed content to the processed bucket
            processed_key = f"processed_{key}"
            s3.put_object(Bucket=processed_bucket_name, Key=processed_key, Body=processed_content.encode('utf-8'))

            print(f"Processed file uploaded to: s3://{processed_bucket_name}/{processed_key}")

        except Exception as e:
            print(f"Error processing file {key}: {e}")

    return {
        'statusCode': 200,
        'body': 'CSV files processed successfully!'
    }
#This is just a sample lambda function that abbreviates everything to UpperCase
```
4. **Configure environment Variables**:
  Add a new environment variable with the following key-value pair:
  - Key: `PROCESSED_BUCKET_NAME`
  - Value: The name of your `processed-data bucket `(your-initials-processed-data-pipeline).
---

##  AWS Glue ETL Job

1. **Job Type**: Spark (Python or Scala)
2. **Source**: S3 `processed-data` bucket
3. **Transformations**:
   - Schema normalization
   - Calculations or joins (if needed)
   - Data type corrections
4. **Destination**: Write to `transformed-data` bucket in Parquet or CSV format

5. **Crawler**: Use a Glue Crawler to catalog data into AWS Glue Data Catalog.

---

##  Amazon QuickSight Setup

1. **Connect to S3**:
   - Set up a manifest file or use Athena if querying Parquet
   - **Sample manifest.json**:
     ```JSON
     {
      "fileLocations": [
        {
            "URIPrefixes": [
                "s3://transformed-data-pipeline-bu/"
            ]
        }
      ],
      "globalUploadSettings": {
        "format": "PARQUET"
        
      }
    }
    ```
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
