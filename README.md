# redshift-s3-export-guide
"Step-by-step guide to exporting data from Amazon Redshift to Amazon S3 using the UNLOAD command."

##Step 1: Set Up S3 
1. Create an S3 Bucket:
   
o Go to the S3 Console. 

o Click Create Bucket. 

o Name your bucket (e.g., my-redshift-data), select the region, and click Create. 

3. Upload Data: 

o Prepare a simple CSV file (sales_data.csv). 

o In your bucket, click Upload → Add Files → Select File → Upload. 

##Step 2: Set Up Redshift 

A. Create a Redshift Cluster 

1. Go to the RedShift Console. 

2. Click Create Cluster and configure: 

o Cluster Identifier: my-redshift-cluster. 

o Node Type: dc2.large or ra3.xlplus. 

o Number of Nodes: 1 (for testing). 

o Master Username: admin. 

o Password: YourSecurePassword123. 

3. Wait until the cluster status changes to Available. 

B. Set Up IAM Role for Redshift 

1. In the Redshift Console, click Existing IAM Role name and Attach the AmazonS3ReadOnlyAccess 
policy.

##Step 3: Connect to Redshift 

1. Open Query Editor v2 in the Redshift Console: 

o Navigate to Query Editor v2. 

o Connect to your cluster. 

▪ If the Database Name is not specified during cluster creation, use the default 
database name dev. 

Workflow: Store, Analyze, and Export Data
 
##Step 1: Create a Table in Redshift 

Run this SQL command to create a table: 

CREATE TABLE sales_data ( 
sales_id INT, 
product_name VARCHAR(50), 
quantity INT, 
price DECIMAL(10, 2), 
sale_date DATE 
);

##Step 2: Load Data from S3 into Redshift 

Run the COPY command to load data from your S3 bucket: 

COPY sales_data 
FROM 's3://my-redshift-data/sales_data.csv' 
IAM_ROLE 'arn:aws:iam::account-id:role/RedshiftS3Role' 
CSV 
IGNOREHEADER 1;

Replace: 

• my-redshift-data with your bucket name. 

• arn:aws:iam::account-id:role/RedshiftS3Role with your IAM role ARN.

##Step 3: Query Data in Redshift 

To verify the data: 

SELECT * FROM sales_data;

Run analytics queries, such as:
 
1. Total sales for each product: 

SELECT product_name, SUM(quantity * price) AS total_sales 
FROM sales_data 
GROUP BY product_name;

2.Find the most popular product: 

SELECT product_name, SUM(quantity) AS total_quantity 
FROM sales_data 
GROUP BY product_name 
ORDER BY total_quantity DESC 
LIMIT 1;

##Step 4: Export Data from Redshift to S3 

Use the UNLOAD command to export filtered data back to S3: 

UNLOAD ('SELECT * FROM sales_data WHERE quantity > 10') 
TO 's3://my-redshift-data/sales_data_filtered_' 
IAM_ROLE 'arn:aws:iam::account-id:role/RedshiftS3Role' 
PARALLEL OFF; 

Replace: 

• my-redshift-data with your bucket name. 
• arn:aws:iam::account-id:role/RedshiftS3Role with your IAM role ARN. 
