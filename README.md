# Agricultural Production Data Warehouse Design.
In today's data-driven world, the agriculture sector is no exception to the benefits of robust data management and analysis. This project focuses on designing a data warehouse to manage agricultural production data using AWS services. The data warehouse facilitates efficient data storage, transformation, and analysis, enabling stakeholders to make informed decisions based on comprehensive data insights. The relevance of this project lies in its ability to handle large volumes of data from diverse sources, ensuring data integrity, scalability, and ease of access.

## Project Architecture
![project architecture](https://github.com/Samuel-Njoroge/agricultural-production-data-warehousing/blob/main/project_images/agricultural_production.svg)

## Objectives
- *Data Aggregation*: Collect agricultural production data from various sources and store it in a centralized S3 bucket.
- *Data Discovery*: Use AWS Glue Crawler to scan and catalog the data for easy accessibility.
- *Data Analysis*: Employ Amazon Athena for querying and analyzing the raw data stored in S3.
- *ETL Processing*: Utilize AWS Glue for Extract, Transform, Load (ETL) operations to clean and transform the data.
- *Data Warehousing*: Load the processed data into Amazon Redshift for efficient querying and reporting.

## Skills
1. Hands-on experience with `AWS S3, AWS Glue, Amazon Athena, Amazon Redshift`.
2. `Data Warehousing`: Understanding of data warehousing concepts and best practices.
3. `ETL Processes`: Skills in designing and implementing ETL processes.

## Tools
1. `AWS S3`: For storing raw agricultural production data from various sources.
2. `AWS Glue`: For data cataloging and ETL processes.
3. `Amazon Athena`: For querying and analyzing data directly from S3.
4. `Amazon Redshift`: For storing and querying transformed data in a data warehouse.
5. `AWS IAM`: For managing access and permissions.
6. `SQL`: For querying data in Athena and Redshift.

## Approach : Quick guide to replicate.
### 1. Data Collection and Storage
The first step involves collecting agricultural production data from various sources and storing it in Amazon S3. S3 provides a secure, scalable, and cost-effective storage solution. 
In this case, the data was collected from a secondary source [Agricultural Database](https://www.kaggle.com/datasets/jacopoferretti/usa-department-of-agricultures-usda-database/data).

Copying the data to s3 bucket.
```
aws s3 cp source_data.csv s3://agricultural-data-bucket/source_data/

```

### 2. Data Cataloging with AWS Glue Crawler
To manage and discover the data stored in S3, an AWS Glue Crawler is used. The Glue Crawler scans the data in S3 and automatically creates metadata tables in the Glue Data Catalog. This step is crucial for making the data easily accessible to other AWS services like Athena and Redshift.

```
import boto3

glue = boto3.client('glue')
response = glue.start_crawler(Name='agricultural-data-crawler')
```

### 3. Data Analysis with Amazon Athena
Amazon Athena is used to query and analyze the raw data stored in S3. Athena allows running SQL queries directly on data in S3 without the need for ETL processes. This helps in initial data exploration and validation.

```
SELECT * FROM "agricultural_data"."source_data" WHERE Year = 2023;
```

### 4. ETL Processing with AWS Glue
AWS Glue is employed for the ETL process. Glue jobs extract data from the S3 bucket, transform it and load it into a format suitable for data warehousing.
```
import sys
from awsglue.transforms import *
from awsglue.utils import getResolvedOptions
from pyspark.context import SparkContext
from awsglue.context import GlueContext
from awsglue.job import Job

args = getResolvedOptions(sys.argv, ['JOB_NAME'])
sc = SparkContext()
glueContext = GlueContext(sc)
spark = glueContext.spark_session
job = Job(glueContext)
job.init(args['JOB_NAME'], args)

# Extract
datasource = glueContext.create_dynamic_frame.from_catalog(database = "agricultural_data", table_name = "source_data")

# Transform
applymapping = ApplyMapping.apply(frame = datasource, mappings = [("field1", "string", "field1", "string"), ("field2", "int", "field2", "int")])

# Load
datasink = glueContext.write_dynamic_frame.from_options(frame = applymapping, connection_type = "redshift", connection_options = {"url": "jdbc:redshift://<redshift-cluster>:5439/<db>", "user": "awsuser", "password": "password", "redshiftTmpDir": args["TempDir"]}, transformation_ctx = "datasink")

job.commit()

```

### 4. Loading Data into Amazon Redshift
The transformed data is then loaded into Amazon Redshift, a fully managed data warehouse service.

```
COPY agricultural_data
FROM 's3://agricultural-data-bucket/transformed_data/'
IAM_ROLE 'arn:aws:iam::<account-id>:role/<role-name>'
CSV;
```
### Full Implementation
To read more on the full implementation, check [Complete Code](https://github.com/Samuel-Njoroge/agricultural-production-data-warehousing/blob/main/agricultural_data.ipynb)
