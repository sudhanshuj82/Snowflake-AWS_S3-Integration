# Snowflake Data Ingestion with AWS S3 Integration
DATA FLOW DIAGRAM
![storage-integration-s3](https://github.com/user-attachments/assets/4112772e-ed26-4c5b-a7cc-a7b7b9022141)


This project demonstrates how to integrate AWS S3 with Snowflake to enable seamless, secure data loading using external stages and storage integration.

## 🔧 Setup Overview

**1. Database & Schema Initialization**
```sql
CREATE OR REPLACE DATABASE Demo_DB;
CREATE SCHEMA demo_migration; 
```
**2. Storage Integration with AWS**
```sql
CREATE OR REPLACE STORAGE INTEGRATION aws_s3_integration
  TYPE = EXTERNAL_STAGE
  STORAGE_PROVIDER = 'S3'
  ENABLED = TRUE
  STORAGE_AWS_ROLE_ARN = '<Your-Role-ARN>'
  STORAGE_ALLOWED_LOCATIONS = ('s3://<your-bucket>/');
```
#Ensure that the IAM role and external ID are correctly configured in AWS.

**3. Stage & File Format Configuration**
```sql
CREATE OR REPLACE FILE FORMAT demo_format
  TYPE = CSV
  FIELD_DELIMITER = ','
  SKIP_HEADER = 1
  NULL_IF = ('')
  EMPTY_FIELD_AS_NULL = TRUE;

CREATE OR REPLACE STAGE demo_s3_stage
  STORAGE_INTEGRATION = aws_s3_integration
  FILE_FORMAT = demo_format
  URL = 's3://<your-bucket>/';
```
**4. Table Creation**
```sql
CREATE OR REPLACE TABLE customer_data (
  "CustomerID" INT,
  "Gender" STRING,
  "Age" INT,
  "Annual Income (k$)" NUMBER(10, 2),
  "Spending Score (1-100)" INT
);
```
**5. Data Loading from S3**
```sql
COPY INTO customer_data
FROM @demo_s3_stage/customers_info.csv
FILE_FORMAT = (FORMAT_NAME = 'demo_format')
ON_ERROR = 'CONTINUE';
```
#Use FORCE = TRUE to reload the same file and PURGE = TRUE to delete the file post ingestion.

**Notes:**
The trust policy must reference the external ID provided by Snowflake.
The S3 bucket policy should explicitly allow access for the role.

