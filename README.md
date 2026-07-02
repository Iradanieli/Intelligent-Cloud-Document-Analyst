# Intelligent Cloud Document Analyst 📄🧠

An automated, AI-driven document processing pipeline built with n8n, Python (FastAPI), and Google Gemini. This microservice architecture automatically ingests cloud documents, extracts text, performs LLM-powered entity extraction, applies custom business routing rules, and distributes the enriched data to end-users and databases.

## 🚀 Features

* **Automated Cloud Ingestion:** Watches a designated Google Drive folder and triggers instantly when a new file is uploaded.
* **Hybrid Text Extraction:** n8n handles TXT/PDF extraction, while the Python FastAPI microservice handles DOCX extraction.
* **AI Entity Extraction:** Leverages a Google Gemini 3.5 Flash Basic LLM Chain to classify documents, determine sentiment, and extract structured JSON data (people, organizations, dates, amounts, and action items).
* **Intelligent Business Logic:** Custom Python routing engine dynamically calculates confidence scores, identifies sensitive information (e.g., NDAs, financial data), and assigns departmental routing tags.
* **Real-Time Notifications:** Sends formatted HTML email alerts via Gmail for every processed document.
* **Centralized Data Logging:** Appends all processed document metadata to a centralized Google Sheet for auditing.
* **Output Reports:** Writes processed JSON records and Markdown summary reports to the `output_docs` Google Drive folder.
* **Failure Handling:** Sends fallback email notifications if DOCX extraction, Gemini analysis, or metadata enrichment fails after retries.
* **Daily Digest:** An automated cron job generates and emails a 24-hour summary report of all pipeline activity.

## 🏗️ Architecture & Workflow

The system is divided into two distinct n8n workflows and one Python microservice.

### 1. Main Ingestion Pipeline
1. **Trigger:** A Google Drive node listens for the `fileCreated` event in the `incoming_docs` folder.
2. **Download:** The file's binary data is downloaded via the Google Drive API.
3. **Routing:** A Switch node routes files by extension: `txt`, `docx`, or `pdf`.
4. **Extraction:** TXT and PDF files are extracted with n8n Extract from File nodes. DOCX files are sent to the Python microservice at `http://host.docker.internal:8000/extract`.
5. **LLM Processing:** The extracted text is passed to a Basic LLM Chain connected to the Google Gemini Chat Model.
6. **Parsing:** A JavaScript Code node parses the Gemini response into structured JSON.
7. **Enrichment:** An HTTP POST request sends the parsed JSON to `http://host.docker.internal:8000/enrich` to apply business routing logic.
8. **Parallel Outputs:** The enriched result is sent in parallel to Google Sheets, Gmail, and Google Drive output nodes.
9. **Output Folder:** The workflow writes both a processed JSON file and a Markdown summary report to the `output_docs` Google Drive folder.
10. **Fallback Alerts:** Error branches send Gmail notifications for DOCX extraction, LLM, and enrichment failures.

### 2. Daily Summary Report
1. **Trigger:** A Schedule Trigger runs daily at 14:00.
2. **Data Retrieval:** Fetches all historical rows from the active Google Sheet.
3. **Filtering & Formatting:** A custom JavaScript Code node filters for documents processed in the last 24 hours and generates an HTML summary table.
4. **Distribution:** Emails the final HTML report to the administrator via Gmail.

## 🛠️ Tech Stack

* **Workflow Engine:** n8n (Dockerized)
* **Backend:** Python 3, FastAPI / Uvicorn
* **LLM:** Google Gemini 3.5 Flash via n8n's Basic LLM Chain
* **APIs:** Google Drive API, Google Sheets API, Gmail API
* **Key Python Libraries:** `fastapi`, `uvicorn`, `python-multipart`, `python-docx`

## ⚙️ Setup & Installation

**1. Python Microservice Setup**

```bash
# Create and activate a virtual environment (Linux/macOS)
python3 -m venv venv
source venv/bin/activate

# (For Windows users)
# python -m venv venv
# venv\Scripts\activate

# Install dependencies
pip install fastapi uvicorn python-multipart python-docx

# Run the backend server locally 
# (Accessible to Docker via host.docker.internal)
uvicorn main:app --host 127.0.0.1 --port 8000
```

**2. n8n Environment Setup**

* Ensure n8n is running (via Docker or desktop app).
* Configure OAuth2 credentials for Google Drive, Google Sheets, and Gmail.
* Configure the API key for Google Gemini (PaLM) API.

**3. Import Workflows**

* Open n8n and click **Import from File**.
* Import `Business Document Analyst-hw.json` for the main document-processing pipeline.
* Import `Business Document Daily Report.json` for the daily summary workflow.
* Update the Google Drive Folder IDs and Google Sheet IDs to match your personal environment.
* Toggle both workflows to Active.

## 👨‍💻 Author

Irad Danieli
