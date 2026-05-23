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
  - This can be done using the AWS Textract Service that is also extended as an API. "analyze_expense" as well as "document_analysis" are some of the APIs available that helps to extract invoice/receipt specific details and table format data respectively. Post extraction these can be rendered as a CSV file for further analysis.
  - Input raw files are placed S3 zone: cn01-project-input
  - Target location in S3 zone: cn01-project-pre-processed-files
      - distance_log_files_pdf/
      - fuel_invoice_files/
      - distance_log_excel_files/
  - The below diagram shows the overview of the process.
 

<img width="1034" height="259" alt="Screenshot 2026-05-23 at 3 47 14 PM" src="https://github.com/user-attachments/assets/eef6fac3-eba8-42c0-9d8b-1c502c12deda" />





- **Stage 2**
  - Processing : Once we have the data from fuel invoices and tables extracted from the PDF files into a CSV structured format, we can process further with the main ETL job.
  - AWS Glue is a suitable ETL platform that provides many options to process and transform data into target. Once such use case is using the interactive notebooks within AWS glue that helps to programmatically perform transformation. This can then be run as a script that will be triggered with the help of AWS Lambda function. Lambda function will check for the files in the configured locations and start the glue script to perform the transformations.
  - Input pre-processed files are placed S3 zone: cn01-project-pre-processed-files/input_files/
  - Target location in S3 zone: cn01-project-output-205096516800-us-east-2-an
      - output_parquet_files/distance_logs/
      - output_parquet_files/fuel_invoice/
  - The below diagram shows the overview of the main processing layer
 
<img width="968" height="450" alt="Screenshot 2026-05-23 at 3 47 24 PM" src="https://github.com/user-attachments/assets/df52c2be-1311-43de-8cde-d324087b9cbc" />



  

- **Stage 3**
  - Dimensional Modelling: 
  - Categorical columns: top‑N category frequency bar charts.

- **Bivariate analysis**
  - Numeric vs numeric: scatter plots.
  - Categorical vs numeric: mean metric by category (bar chart).

- **Compact summary for LLM**
  - Generates a concise text summary with:
    - Number of rows.
    - Up to N columns: type, non‑null count, missing %, min/max/mean (numeric), top 3 categories (categorical). 

- **AI EDA Insights (Gemini / Google ADK)**
  - Gemini agent (via Google ADK) reads the summary and returns:
    - 4 notable patterns.
    - 4 potential data quality issues.
    - 4 recommended EDA steps.
    - 4 business questions in stakeholder‑friendly language.
  - Uses an ADK `Agent` + `Runner` with proper async session creation and error handling.

---

## 3. Tech stack

**Core:**

- Python  
- pandas  
- Streamlit  
- matplotlib, seaborn  

**AI / LLM:**

- Gemini (via `google-genai`)  
- Google Agent Development Kit (`google-adk-python`) 

**Other:**

- In‑memory session management with `InMemorySessionService`  
- Async agent execution wrapped in a sync helper for Streamlit

---
## 4. Usage guide

1. **Overview tab**
   - Inspect shape, data types, numeric summary, and sample rows.
   - Quickly confirm that dates and numeric fields parsed correctly.

2. **Univariate analysis**
   - Select a column and:
     - For numeric: review histogram and boxplot to check distribution, skew, and outliers.
     - For categorical: inspect the top categories and their frequencies. 

3. **Relationships**
   - Numeric vs numeric: look at scatter plots (e.g., quantity vs revenue).
   - Categorical vs numeric: compare mean revenue or quantity by region, product category, etc. 

4. **Summary Text (for LLM)**
   - Adjust `max_cols` and view the compact summary text.
   - This is exactly what gets passed to the Gemini agent as context.

5. **AI EDA Insights**
   - Choose `max_cols` and click **“Generate AI EDA insights”**.
   - The Gemini/ADK agent returns:
     - 4 patterns (distributions, dominant categories, ranges).
     - 4 potential data quality issues.
     - 4 recommended next EDA steps.
     - 4 business questions phrased for stakeholders. 

---

## 5. Example use cases

- **Data / Business Analyst:** Use the app to quickly profile new ecommerce datasets and get AI‑generated ideas for further analysis and stakeholder questions.  
- **Portfolio piece:** Demonstrate how you combine Python/pandas EDA with LLM agents for analysis acceleration and storytelling.  

---

## 6. Screenshots

_Add 2–3 screenshots here, for example:_

- Overview tab (shape, schema, summary).
  <img width="1235" height="726" alt="Screenshot 2026-05-14 at 2 01 19 PM" src="https://github.com/user-attachments/assets/4ceb9e4e-2f5a-4a27-8d5c-a6bf3d843551" />

- Univariate analysis on `price` or `revenue`.
  <img width="1128" height="778" alt="Screenshot 2026-05-14 at 2 02 06 PM" src="https://github.com/user-attachments/assets/283c9cf6-3b57-4687-b272-eef7ba0ffe30" />

  <img width="1128" height="778" alt="Screenshot 2026-05-14 at 2 02 52 PM" src="https://github.com/user-attachments/assets/8765854a-2981-4e4f-b66c-778d08702215" />


- AI EDA Insights tab showing Gemini’s markdown output.
 <img width="1128" height="778" alt="Screenshot 2026-05-14 at 2 03 43 PM" src="https://github.com/user-attachments/assets/7db789a2-3617-4d40-8ae5-bc2c1d087c58" />

---

## 7. Roadmap

Possible future enhancements:

- Connect to a **SQL backend** (DuckDB/Postgres/BigQuery) instead of only CSVs.   
- Add **time‑series EDA** (e.g., revenue by day/week/month, rolling averages).  
- Extend the AI layer into a multi‑agent system (e.g., separate agents for anomalies, segmentation, marketing recommendations).  
- Export AI‑generated EDA reports to Markdown/PDF for stakeholders.

---
