# AI Sales Outreach Automation

An AI-powered sales research and outreach workflow that turns CRM leads into researched, qualified, and personalized sales opportunities using LangGraph, LangChain, LLMs, web research, CRM integrations, and retrieval-augmented generation.
<img width="1363" height="509" alt="image" src="https://github.com/user-attachments/assets/08931c7d-da1a-4189-8db9-11f3837a82b1" />

## Overview

**AI Sales Outreach Automation** is designed to reduce the manual work involved in researching prospects and preparing sales outreach.

The system retrieves leads from a connected CRM, researches the prospect and their company, analyzes the company's website and digital presence, reviews recent business news, generates structured research reports, scores the lead, and—when the lead meets the qualification threshold—creates personalized outreach materials.

The workflow is orchestrated as a stateful **LangGraph** application, allowing each research and outreach stage to operate as a dedicated workflow node while maintaining shared lead and report state.

Generated reports can be saved locally and optionally published to Google Docs. CRM records can also be updated with the resulting lead score, report references, outreach information, and contact status.

---

## Workflow

```text
┌──────────────┐
│  CRM Leads   │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Lead Research│
│ LinkedIn +   │
│ Company Data │
└──────┬───────┘
       │
       ▼
┌─────────────────────┐
│ Digital Research    │
│ Website / Blog /    │
│ YouTube / News      │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Research Reports    │
│ & Lead Analysis     │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Lead Qualification  │
│ Score >= 7 →        │
│ Continue             │
└──────────┬──────────┘
           │
      ┌────┴─────┐
      │           │
   Qualified   Not Qualified
      │           │
      ▼           ▼
┌─────────────┐  ┌──────────────┐
│ RAG / Case  │  │ Save Research│
│ Study Match │  │ Reports      │
└──────┬──────┘  └──────┬───────┘
       │                 │
       ▼                 │
┌─────────────────────┐  │
│ Personalized        │  │
│ Outreach Report     │  │
└──────────┬──────────┘  │
           │             │
      ┌────┴─────┐       │
      ▼          ▼       │
┌───────────┐ ┌──────────────┐
│ Email     │ │ Interview /  │
│ Draft     │ │ SPIN Script  │
└─────┬─────┘ └──────┬───────┘
      └──────┬───────┘
             ▼
      ┌──────────────┐
      │ Save Reports │
      └──────┬───────┘
             ▼
      ┌──────────────┐
      │  CRM Update  │
      └──────┬───────┘
             │
             └──────► Next Lead
```

### High-level flow

**Lead → Research → Analysis → Qualification → Personalization → Outreach → CRM Update**

---

## Key Features

### CRM Integration

The project uses a common lead-loader abstraction so CRM integrations can share the same automation workflow.

Currently implemented loaders include:

* **Airtable**
* **HubSpot**
* **Google Sheets**

Each loader supports retrieving leads and updating CRM records.

The default `main.py` configuration uses Airtable, while Google Sheets is available as an alternative configuration.

### Automated Lead Research

For each lead, the workflow can:

* Search for the prospect's LinkedIn profile
* Retrieve LinkedIn profile information
* Summarize the prospect's professional background
* Identify the prospect's company
* Retrieve company information
* Identify the company's website and LinkedIn information

The LinkedIn lookup uses search plus the configured RapidAPI-based LinkedIn data integration.

### Company Analysis

The workflow researches the prospect's organization using:

* Company LinkedIn information
* Company website content
* Blog content
* YouTube activity
* Recent company news

The website analysis also extracts relevant blog and social-media URLs when available.

### Lead Qualification

After research is completed, the system generates a lead score using an LLM.

The current workflow considers a lead qualified when the generated score is **7 or higher**.

The qualification threshold can be changed in the workflow implementation to match a different sales strategy.

### Personalized Outreach

Qualified leads receive additional AI-generated sales material, including:

* A customized outreach report
* A personalized email
* An interview/discovery preparation script
* SPIN-style sales questions

The personalized email is generated from the lead research and includes the generated outreach-report reference.

The implementation creates a Gmail draft by default. Direct sending is disabled by default and is controlled in the workflow configuration.

### RAG / Knowledge Retrieval

The project includes a retrieval-augmented generation component powered by:

* Chroma
* Google Generative AI embeddings
* LangChain document/retrieval components

Case studies stored under `data/case_studies/` are loaded into the vector store and used to retrieve a relevant case study for a qualified prospect.

This allows generated outreach to reference business examples from the project's knowledge base rather than relying exclusively on the LLM's general knowledge.

### Automated Reports

The workflow generates multiple research and outreach artifacts, including:

* General Lead Research Report
* Blog Analysis Report
* YouTube Analysis Report
* News Analysis Report
* Digital Presence Report
* Global Lead Analysis Report
* Personalized Email
* Interview Script
* Customized Outreach Report

Reports are saved locally under `reports/`.

Google Docs/Drive integration is also implemented and can be enabled through the workflow configuration.

### CRM Updates

After processing a lead, the workflow can update the CRM with information such as:

* Lead status
* Lead score
* Analysis report location
* Outreach report location
* Last-contacted date

The default workflow changes processed leads to `ATTEMPTED_TO_CONTACT`.

---

## How It Works

The application is implemented as a stateful LangGraph workflow.

### 1. Load Leads

The configured CRM loader retrieves leads with the `NEW` status.

The lead information is converted into the application's internal `LeadData` structure.

### 2. Research the Lead

The system searches for the lead's LinkedIn profile and collects available information such as:

* Professional experience
* Current company
* Skills
* Education
* Certifications
* Location
* Previous roles

An LLM converts the collected information into a concise lead profile.

### 3. Research the Company

The company associated with the lead is researched through LinkedIn and the company website.

The system builds a company profile containing information such as:

* Company description
* Industry
* Specialties
* Employee count
* Locations
* Year founded
* Website
* Relevant social links

### 4. Analyze Digital Presence

The company website is scraped and analyzed.

When available, the workflow also analyzes:

* Blog activity
* YouTube activity and channel metrics
* Recent company news

The website-analysis stage extracts blog, Facebook, Twitter, and YouTube URLs.

**Current implementation note:** YouTube analysis is implemented. Facebook and Twitter URLs are detected, but dedicated Facebook/Twitter analysis functions are not currently implemented.

### 5. Generate Research Reports

The collected information is combined into progressively more comprehensive reports:

```text
Lead / Company Research
        │
        ▼
Digital Presence Analysis
        │
        ▼
Global Lead Analysis
```

These reports provide the context used by the qualification and outreach stages.

### 6. Score the Lead

The global research report is passed to the configured LLM to produce a lead score.

The current qualification rule is:

```text
Score >= 7  → Qualified
Score < 7   → Not Qualified
```

### 7. Retrieve Relevant Knowledge

For qualified leads, the system queries the Chroma vector store using the generated research information.

The most relevant case study is retrieved from `data/case_studies/`.

### 8. Generate Outreach

The retrieved case study and lead research are used to create a customized outreach report.

The workflow then generates:

* Personalized email
* SPIN questions
* Interview/discovery script

### 9. Save Results

Generated reports are written to the local `reports/` directory.

Google Docs/Drive saving can also be enabled.

### 10. Update the CRM

Finally, the lead record is updated with its score, report references, status, and contact date.

The workflow then continues with the next available lead.

---

## AI Architecture

The application separates the automation into specialized LangGraph nodes.

```text
                         ┌────────────────────┐
                         │   Lead Loader      │
                         │ Airtable / HubSpot │
                         │ / Google Sheets    │
                         └─────────┬──────────┘
                                   │
                                   ▼
                         ┌────────────────────┐
                         │   Lead Research    │
                         │ LinkedIn + Search  │
                         └─────────┬──────────┘
                                   │
                                   ▼
                         ┌────────────────────┐
                         │ Company Research   │
                         │ LinkedIn + Website │
                         └─────────┬──────────┘
                                   │
                     ┌─────────────┼─────────────┐
                     ▼             ▼             ▼
                  Website        Blog          News
                  Analysis      Analysis      Analysis
                     │             │             │
                     └─────────────┼─────────────┘
                                   ▼
                         ┌────────────────────┐
                         │ Digital Presence   │
                         │     Analysis       │
                         └─────────┬──────────┘
                                   ▼
                         ┌────────────────────┐
                         │ Global Lead Report │
                         └─────────┬──────────┘
                                   ▼
                         ┌────────────────────┐
                         │    Lead Scoring    │
                         └─────────┬──────────┘
                                   │
                           Score >= 7?
                         ┌─────────┴─────────┐
                         │                   │
                        Yes                  No
                         │                   │
                         ▼                   ▼
                 ┌──────────────┐     Save Reports
                 │ RAG Retrieval│
                 │ Case Study   │
                 └──────┬───────┘
                        ▼
                 ┌──────────────┐
                 │ Outreach     │
                 │ Generation   │
                 └──────┬───────┘
                        ▼
              Email + Interview Script
                        │
                        ▼
                 Save + CRM Update
```

LangGraph manages the state and transitions between these stages, while LangChain components provide LLM, embedding, document-loading, and retrieval capabilities.

---

## Tech Stack

| Technology                       | Purpose                                                 |
| -------------------------------- | ------------------------------------------------------- |
| **Python**                       | Application runtime                                     |
| **LangGraph**                    | Stateful AI workflow orchestration                      |
| **LangChain**                    | LLM and AI application components                       |
| **Google Gemini**                | Primary LLM and embedding provider                      |
| **OpenAI**                       | Supported LLM provider through the provider abstraction |
| **ChromaDB**                     | Vector storage and similarity retrieval                 |
| **Pydantic**                     | Structured application and LLM output models            |
| **Airtable API**                 | CRM lead retrieval and updates                          |
| **HubSpot API**                  | CRM lead retrieval and updates                          |
| **Google Sheets API**            | Spreadsheet-based lead source                           |
| **Google Docs / Drive APIs**     | Optional report storage and sharing                     |
| **Gmail API**                    | Email draft creation and optional sending               |
| **RapidAPI**                     | LinkedIn profile data integration                       |
| **Serper API**                   | Web and company-news search                             |
| **YouTube Data API**             | YouTube channel and video metrics                       |
| **BeautifulSoup / Unstructured** | Web/document processing                                 |

---

## Project Structure

The repository is organized around the LangGraph workflow, integrations, prompts, business knowledge, and generated reports.

```text
.
├── .env.example
├── README.md
├── main.py
├── requirements.txt
├── workflow.png
│
├── data/
│   ├── agency-description.md
│   └── case_studies/
│       ├── ecotrend.md
│       ├── spark-retail.md
│       └── wellspring-nutrition.md
│
├── database/
│   ├── chroma.sqlite3
│   └── <Chroma vector-store files>
│
├── docs/
│   ├── customization.md
│   └── system-workflow.md
│
├── reports/
│   ├── Blog Analysis Report.txt
│   ├── Digital Presence Report.txt
│   ├── General Lead Research Report.txt
│   ├── Global Lead Analysis Report.txt
│   ├── Interview Script.txt
│   ├── News Analysis Report.txt
│   ├── Personalized Email.txt
│   └── Youtube Analysis Report.txt
│
└── src/
    ├── graph.py
    ├── nodes.py
    ├── prompts.py
    ├── state.py
    ├── structured_outputs.py
    ├── utils.py
    │
    └── tools/
        ├── company_research.py
        ├── google_docs_tools.py
        ├── lead_research.py
        ├── rag_tool.py
        ├── youtube_tools.py
        │
        ├── base/
        │   ├── gmail_tools.py
        │   ├── linkedin_tools.py
        │   ├── markdown_scraper_tool.py
        │   └── search_tools.py
        │
        └── leads_loader/
            ├── airtable.py
            ├── google_sheets.py
            ├── hubspot.py
            └── lead_loader_base.py
```

---

## Installation

### Prerequisites

Before running the project, make sure you have:

* Python 3.9 or newer
* A configured CRM integration
* An LLM API key
* Serper API credentials for web search
* RapidAPI credentials for the LinkedIn data integration
* Google API credentials if using Google Sheets, Gmail, Google Docs, or Google Drive features

The required Python packages are listed in `requirements.txt`.

### 1. Clone the Repository

```bash
git clone https://github.com/hammadproject/sales-outreach-automation.git
cd sales-outreach-automation
```

### 2. Create a Virtual Environment

```bash
python -m venv venv
```

Activate it on Linux/macOS:

```bash
source venv/bin/activate
```

On Windows:

```powershell
venv\Scripts\activate
```

### 3. Install Dependencies

```bash
pip install -r requirements.txt
```

### 4. Create the Environment File

Copy the provided environment template:

```bash
cp .env.example .env
```

On Windows, you can create `.env` manually from `.env.example` if the `cp` command is unavailable.

Add only the credentials required for the integrations you intend to use.

### 5. Configure Google APIs

Google integrations use OAuth credentials and the project expects Google credentials to be available through the application's Google authentication flow.

The required scopes include access for:

* Gmail
* Google Sheets
* Google Docs
* Google Drive

The application creates/refreshes a local OAuth token as required.

Do not commit OAuth credentials or generated tokens to source control.

---

## Environment Variables

The repository's `.env.example` defines the following configuration areas:

```env
GEMINI_API_KEY=
GOOGLE_API_KEY=
OPENAI_API_KEY=

SERPER_API_KEY=
RAPIDAPI_KEY=

AIRTABLE_ACCESS_TOKEN=
AIRTABLE_BASE_ID=
AIRTABLE_TABLE_NAME=

HUBSPOT_API_KEY=

SHEET_ID=
```

Not every variable is required for every configuration.

### CRM configuration

**Airtable**

```env
AIRTABLE_ACCESS_TOKEN=
AIRTABLE_BASE_ID=
AIRTABLE_TABLE_NAME=
```

**HubSpot**

```env
HUBSPOT_API_KEY=
```

**Google Sheets**

```env
SHEET_ID=
```

The active CRM loader is selected in `main.py`.

### Google authentication files

The Google authentication utility expects OAuth client credentials and manages a local token file during authentication.

These files contain sensitive credentials and should remain outside version control.

---

## Running the Automation

The application's entry point is:

```bash
python main.py
```

The default `main.py` configuration initializes the Airtable loader and processes available leads.

The workflow starts with:

```python
inputs = {"leads_ids": []}
```

An empty list allows the configured loader to retrieve new leads according to its implementation.

To use Google Sheets instead, the corresponding loader configuration in `main.py` can be enabled.

---

## Configuration & Customization

The project is designed so that CRM behavior, business knowledge, qualification logic, and AI prompts can be adapted without rebuilding the entire workflow.

### Customize Business Knowledge

Business information is stored under:

```text
data/
├── agency-description.md
└── case_studies/
```

Update these files with the business information and case studies that should influence generated outreach.

The case studies are used by the RAG pipeline to retrieve relevant examples for qualified leads.

### Add or Replace a CRM

CRM integrations implement the `LeadLoaderBase` abstraction.

A custom CRM loader should implement:

```python
fetch_records(...)
update_record(...)
```

This allows the new CRM to plug into the existing LangGraph workflow.

### Customize Lead Fields

The current lead mapping expects fields such as:

* First Name
* Last Name
* Email
* Phone
* Address

If your CRM uses different field names, update the mapping in the lead-loading stage.

### Customize Lead Statuses

The base loader defines supported statuses including:

```text
NEW
UNQUALIFIED
ATTEMPTED_TO_CONTACT
```

These values can be modified to match the statuses used by your CRM.

### Change the Qualification Threshold

The current qualification rule is implemented as:

```python
float(state["lead_score"]) >= 7
```

Change this value if your sales process requires a different qualification threshold.

### Customize AI Prompts

The project's AI instructions are centralized in:

```text
src/prompts.py
```

Prompts can be adapted for:

* Lead research
* Company research
* Blog analysis
* YouTube analysis
* News analysis
* Digital-presence reporting
* Lead scoring
* Outreach reports
* Personalized emails
* SPIN questions
* Interview scripts

### Change the LLM Provider

The utility layer includes an LLM-provider abstraction supporting Google and OpenAI configurations, with the provider selected through the application code.

The installed dependencies should match the provider you intend to use.

---

## Report Storage

Generated reports are stored locally in:

```text
reports/
```

The application can also use Google Docs/Drive for centralized storage and sharing.

Google Docs saving is controlled through the workflow configuration.

By default, direct Google Docs report saving is disabled:

```python
SAVE_TO_GOOGLE_DOCS = False
```

---

## Email Behavior

The system generates a personalized email and creates a Gmail draft.

Direct email sending is disabled by default:

```python
SEND_EMAIL_DIRECTLY = False
```

This provides an opportunity for human review before messages are sent.

If direct sending is enabled, ensure the generated content and recipient handling have been reviewed for your use case.

---

## Security

This project integrates with multiple external services and may process personal, CRM, and business information.

Follow these practices:

* Never commit API keys to Git.
* Never commit `.env` files.
* Never commit Google OAuth credentials.
* Never commit generated OAuth tokens.
* Do not expose CRM access tokens.
* Avoid storing unnecessary personal information in generated reports.
* Review generated outreach before enabling automated sending.
* Use least-privilege credentials where supported.
* Treat CRM exports, lead information, and research reports as potentially sensitive data.
* Review third-party API terms and applicable privacy requirements before processing prospect information at scale.

Add sensitive files to `.gitignore` before using the project with real credentials or customer data.

---

## Current Limitations

The current implementation has several areas that are intentionally open for further development:

* Facebook analysis is not implemented.
* Twitter/X analysis is not implemented.
* The default entry point is configured for Airtable, although additional CRM loaders exist.
* Some integrations require external API accounts and credentials.
* Generated content should be reviewed before production outreach.
* The current qualification mechanism relies on an LLM-generated numeric score.
* The RAG pipeline currently retrieves the top matching case study rather than performing multi-document synthesis.
* Google Docs saving and direct email sending are disabled by default.

---

## Future Improvements

Potential next steps include:

* Add dedicated Facebook and LinkedIn/X social analysis.
* Support additional CRM platforms.
* Add configurable qualification rules instead of relying on a single numeric threshold.
* Introduce human approval checkpoints before outreach is sent.
* Improve RAG retrieval with metadata filtering and multiple relevant case studies.
* Add automated evaluation of generated emails and reports.
* Add retry handling and structured error reporting for external APIs.
* Add asynchronous processing for larger lead volumes.
* Add observability and workflow execution tracing.
* Introduce automated tests for CRM loaders, workflow nodes, and integrations.
* Add a production-ready configuration layer instead of editing application code for common settings.
* Add stronger privacy controls and data-retention policies.
* Add a web dashboard for monitoring lead-processing progress and generated assets.

---

## Contributing

Improvements, bug fixes, integrations, and workflow enhancements are welcome.

A typical contribution workflow is:

1. Create a feature branch.
2. Make the change.
3. Test the affected workflow or integration.
4. Document configuration changes where necessary.
5. Open a pull request with a clear description of the change.

---

## License

No license file is currently included in the repository.

Until a license is explicitly added, the project should **not be assumed to be open-source licensed for unrestricted reuse, modification, or redistribution**.

If this project is intended for public open-source distribution, add an appropriate `LICENSE` file and update this section accordingly.
