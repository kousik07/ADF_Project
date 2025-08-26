# Azure Data Factory – Data Ingestion Project

This project demonstrates the implementation of an **end-to-end data ingestion framework** using **Azure Data Factory (ADF)**. 
The solution integrates data from multiple heterogeneous sources (On-Premises, REST API, and Azure SQL Database) into 
**Azure Data Lake Storage Gen2 (ADLS Gen2)**, applying best practices such as parameterization, incremental loads, 
and failure alerting via Logic Apps.

## Project Overview
- **Pipeline 1**: On-Premises to ADLS Gen2 (File-based ingestion)
- **Pipeline 2**: REST API to ADLS Gen2 (API ingestion)
- **Pipeline 3**: Azure SQL Database to ADLS Gen2 (Incremental ingestion with watermarking)
- **Pipeline 4**: Master Orchestration (Pipeline execution & monitoring with Logic App alerts)


## Pipeline 1: On-Premises Ingestion → ADLS Gen2
This pipeline ingests files from an **On-Premises file system** into ADLS Gen2.

**Key Features**
- Established connectivity using **Self-Hosted Integration Runtime (SHIR)**.
- Created **Linked Services** for On-Premises source and ADLS Gen2 sink.
- Designed a **parameterized pipeline**:
  - Defined an **Array parameter (`Files`)** containing filenames in JSON format.
  - Used a **ForEach activity** to iterate over the list of files.
- Inside ForEach:
  - Used **Copy Activity** to move files from On-Premises folder (`ADF_Project`) into a newly created container in ADLS Gen2.
- Created **two datasets** (Source: On-Prem, Sink: ADLS).


## Pipeline 2: REST API Ingestion → ADLS Gen2
This pipeline ingests JSON data from an external **REST API endpoint** into ADLS Gen2.

**Key Features**
- Used **Web Activity** to trigger API calls (HTTP GET method).
- Created an **HTTP Linked Service** with:
  - **Base URL** defined in the linked service.
  - **Relative URL** passed dynamically for flexibility.
- Ingested API response (JSON format) into ADLS Gen2 using **Copy Activity**.
- Created **two datasets**: one for the API source and one for the ADLS sink.


## Pipeline 3: Azure SQL Database → ADLS Gen2 (Incremental Ingestion)
This pipeline extracts data from **Azure SQL Database** into ADLS Gen2 with **incremental loading** using the **watermark technique**.

**Key Features**
- Source: **Azure SQL Dataset** (Linked Service to SQL DB).
- Implemented **watermarking mechanism**:
  - Maintained an **empty JSON file** in ADLS Gen2 to store the last load timestamp.
  - **Lookup Activity** reads the last loaded timestamp.
  - Query extracts only new/changed records:
    ```sql
    SELECT * 
    FROM Booking 
    WHERE Booking_date > @LastLoadDate
    ```
  - Another **Lookup Activity** fetches the **latest load timestamp**.
  - Used **Copy Activity** to update the watermark JSON with the new timestamp.
- Applied **Set Variable activity** to capture `utcNow()` for timestamping incremental loads.


## Pipeline 4: Orchestration & Failure Handling
This is the **master pipeline** responsible for orchestrating Pipelines 1, 2, and 3, and handling failure alerts.

**Key Features**
- Used **Execute Pipeline activity** to invoke ingestion pipelines sequentially.
- Implemented **Azure Logic Apps** for **alerting**:
  - Logic App workflow sends an **email notification** when any pipeline fails.
  - Integrated with ADF using **Web Activity** (POST method), triggered from the **failure output** of pipelines.
- Ensures full **end-to-end automation and monitoring** of ingestion pipelines.

### Pipeline 4 – Conceptual Venn Diagram
![Pipeline 4 Orchestration](Pipeline4_VennDiagram.png)


## Key Learnings & Highlights
- Hybrid connectivity with **Self-Hosted Integration Runtime (SHIR)**.
- Ingestion from **multiple heterogeneous sources** (On-Premises, REST API, Azure SQL DB).
- **Parameterization** for reusability and dynamic file handling.
- **Incremental loading** using **watermarking technique**.
- **End-to-end orchestration** with failure alerting via **Logic Apps**.
