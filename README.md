# IFTA project Pipeline (AWS Services)

An Objective of this project to demo how the different files available can be processed to a format usable for further analysis in IFTA audits utilizing the AWS services.

---

## 1. Project overview

 There are mainly 2 types of files to be processed,

- Distance logs - Tables scanned as PDF file and Excel files
- Fuel invoice - Receipts of the fuel charges being made.

Assumptions: The test code used has been rendered using Pandas as it is lightweight and quick for testing files. In a production environment these will be processed using Spark with its distributed data platform. 

---

## 2. Architecture

- **Stage 1**
  - Pre-processing input layer : This step focusses on working on unstructrued data in the form of invoices and tables provided to us in a PDF format the first step would be to extract these into a structured format suitable for further processing.
  - This can be done using the AWS Textract Service that is also extended as an API. "analyze_expense" as well as "document_analysis" are some of the APIs available that helps to extract invoice/receipt specific details and table format data respectively. Post extraction these can be rendered as a CSV file for further analysis. AWS lambda function will help to trigger the AWS Glue script that will pre-process these files with the textract APIs.
  - Input raw files are placed S3 zone: cn01-project-input
  - Target location in S3 zone: cn01-project-pre-processed-files
      - distance_log_files_pdf/
      - fuel_invoice_files/
      - distance_log_excel_files/
  - The below diagram shows the overview of the process.
 

<img width="1034" height="259" alt="Screenshot 2026-05-23 at 3 47 14 PM" src="https://github.com/user-attachments/assets/eef6fac3-eba8-42c0-9d8b-1c502c12deda" />





- **Stage 2**
  - Processing : Once we have the data from fuel invoices and tables extracted from the PDF files into a CSV structured format, we can proceed further with the main next transformation job. Here we can work on the excel distance log files and the CSVs.
  - AWS Glue is a suitable ETL platform that provides many options to process and transform data into target. Once such use case is using the interactive notebooks within AWS glue that helps to programmatically perform transformation. This can then be run as a script that will be triggered with the help of AWS Lambda function. Lambda function will check for the files in the configured locations and start the glue script to perform the transformations.
  - Input pre-processed files are placed S3 zone: cn01-project-pre-processed-files/input_files/
  - Target location in S3 zone: cn01-project-output-205096516800-us-east-2-an
      - output_parquet_files/distance_logs/
      - output_parquet_files/fuel_invoice/
  - The output files are partitioned by year and date and saved as parquet files for effective data storage and access.
  - The tables are registered with the Glue Data Catalog that will also help to keep the metadata as well as track the lineage.
  - Once these are done, external tables are created on Redshift that will access the data from the Data Catalog thereby ensuring that when new files arrive, the warehouse will automatically reflect the most updated data.
  - The below diagram shows the overview of the main processing layer
 
<img width="968" height="450" alt="Screenshot 2026-05-23 at 3 47 24 PM" src="https://github.com/user-attachments/assets/df52c2be-1311-43de-8cde-d324087b9cbc" />



  

- **Stage 3**
  - Dimensional Modelling: Once the data is available to read from AWS Redshift it can then be used to create dimension and fact tables. The following dimensional model has been designed for this use case.
  - Dimension tables are location_dim, date_dim, jurisdiction_dim.
  - Fact tables are fuel_purchase_facts, trip_distance_facts.
  - location_dim: consists of geographical details with the distinct city and province level mapping.
  - date_dim: consists of the data level details for hierarchy, currently it has been set at the year, quarter and month level. Further down to days and day of week can be extended as per need.
  - jusrisdiction_dim: it consists of the province level distance covered details for each record of the distance log.
  - fuel_purchase_facts: consists the values like receipt_date, receipt_id, vendor_name, total_price, litres etc.
  - trip_distance_facts: consists of values like odometer readings, total_fuel, vin_number etc
  - The below diagram shows the overview of the dimensional model
 









  - After these dimension and fact tables are created, materialized views can be made for aggregate tables that can be refreshed to get the most updated result. For example
    - aggregation by year: trip_distance_facts table can be aggregated over the distance column to find the total distance covered at each year level, quarter and month level by joining it with the date dimension and grouping by the desired date level.
    - aggregation by jurisdiction: trip_distance_facts table can be aggregated over the distance column to find the total distance covered at each jusisdiction level by joining with the jurisdiction_dim and grouping.


## 3. Tech stack

- S3  
- Lambda  
- Glue  
- Textract
- Redshift  


---
