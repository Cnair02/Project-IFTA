# IFTA Project Pipeline (AWS Services)

An Objective of this project to demo how the IFTA different files formats available can be processed to a format usable for further analysis in audits utilizing the AWS services.

---

## 1. Project overview

 There are mainly 2 types of files to be processed,

- Distance logs - Tables scanned as PDF file and Excel files
- Fuel invoice - Receipts of the fuel charges being made.

Assumptions: The test code used has been rendered using Pandas as it is lightweight and quick for small files. In a production environment these will be processed using Spark with its distributed data platform abilities. 

---



## 2. Architecture

A high level overview of the process:
- Have an IAM role that can utilize the AWS services the end-to-end with minimal accesses needed.
- AWS S3 will be used as the storage layer for this project. When creating the buckets encryption (SSE-KMS) be enforced.

  <img width="1010" height="126" alt="Screenshot 2026-05-23 at 7 16 16 PM" src="https://github.com/user-attachments/assets/22fc3475-3019-4c63-a595-ebe43375e27f" />

  
- Each layer from raw storage to pre-process to transform layer will have a separate zone in S3 to maintain proper distinction and access

  ### 1. Pre-processing ###

  - Pre-processing input layer : This step focusses on working on unstructrued data in the form of invoices and tables provided to us in a PDF format the first step would be to extract these into a structured format suitable for further processing.
  
  - This can be done using the AWS Textract Service that is also extended as an API. "analyze_expense" as well as "analyze_document" are some of the APIs available that helps to extract invoice/receipt specific details and table format data respectively. Post extraction these can be rendered as a CSV file for further analysis. AWS lambda function will help to trigger the AWS Glue script that will pre-process these files with the textract APIs.
    
  - Input raw files are placed S3 zone: cn01-project-input
      - distance_log_excel_files/file_name.xlsx
      - distance_log_files_pdf/file_name.pdf
      - fuel_invoice_files/file_name.pdf
        
  - Target location in S3 zone: cn01-project-pre-processed-files
      - /input_files/distancelog_files/file_name.csv
      - /input/invoice_files/file_name.csv
  - The below diagram shows the overview of the process.

 
<img width="1010" height="192" alt="Screenshot 2026-05-23 at 7 16 57 PM" src="https://github.com/user-attachments/assets/4b7800e5-cb17-475e-bcd8-d01c1773682b" />


  ### 2. ETL Layer ###
  
  - Processing : Once we have the data from fuel invoices and tables extracted from the PDF files into a CSV structured format, we can proceed further with the main next transformation job. Here we can work on the excel distance log files and the CSVs.

  - AWS Glue is a suitable ETL platform that provides many options to process and transform data into target. Once such use case is using the interactive notebooks within AWS glue that helps to programmatically perform transformation. This can then be run as a script that will be triggered with the help of AWS Lambda function. Lambda function will check for the files in the configured locations and start the glue script to perform the transformations.
    
  - Input pre-processed files are placed S3 zone: cn01-project-pre-processed-files/input_files/
    
  - Target location in S3 zone: cn01-project-output-205096516800-us-east-2-an
      - output_parquet_files/distance_logs/year=2016/month=3/file_name.parquet
      - output_parquet_files/fuel_invoice/year=2016/month=3/file_name.parquet
        
  - The output files are partitioned by year and date and saved as parquet files for effective data storage and access.
    
  - The tables are registered with the Glue Data Catalog that will also help to keep the metadata as well as track the lineage.
    
  - Once these are done, external tables are created on Redshift that will access the data from the Data Catalog thereby ensuring that when new files arrive, the warehouse will automatically reflect the most updated data.
    
  - The below diagram shows the overview of the main processing layer

 
<img width="896" height="417" alt="Screenshot 2026-05-23 at 7 17 32 PM" src="https://github.com/user-attachments/assets/e7ee8f76-b595-43e0-9550-a5281d1c6381" />



  ### 3. Dimensional Modelling ###
  
  - Dimensional Modelling: Once the data is available to read from AWS Redshift it can then be used to create dimension and fact tables. The following dimensional model has been designed for this use case.
    
  - Dimension tables are location_dim, date_dim, jurisdiction_dim.
    
  - Fact tables are fuel_purchase_facts, trip_distance_facts.
    
  - location_dim: consists of geographical details with the distinct city and province level mapping.
    
  - date_dim: consists of the data level details for hierarchy, currently it has been set at the year, quarter and month level. Further down to days and day of week can be extended as per need.
    
  - jusrisdiction_dim: it consists of the province level distance covered details for each record of the distance log.
    
  - fuel_purchase_facts: consists the values like receipt_date, receipt_id, vendor_name, total_price, litres etc.
    
  - trip_distance_facts: consists of values like odometer readings, total_fuel, vin_number etc
    
  - The below diagram shows the overview of the dimensional model
 

<img width="960" height="584" alt="Screenshot 2026-05-23 at 7 18 00 PM" src="https://github.com/user-attachments/assets/dedfe645-afb3-42a1-9a6d-af632f3c15f1" />


  - After these dimension and fact tables are created, materialized views can be made for aggregate tables that can be refreshed to get the most updated result. For example
    
    - aggregation by year: trip_distance_facts table can be aggregated over the distance column to find the total distance covered at each year level, quarter and month level by joining it with the date dimension and grouping by the desired date level.
      
    - aggregation by jurisdiction: trip_distance_facts table can be aggregated over the distance column to find the total distance covered at each jusisdiction level by joining with the jurisdiction_dim and grouping.

---

## 3. Tech stack

- S3  
- Lambda  
- Glue  
- Textract
- Redshift

---

## 4. Reconciliation

Since the distance logs contain date, origin and destination, mapping the date and city of the logs with the fuel invoice city and date would help to correctly identity the corresponding fuel related data points. 


---

