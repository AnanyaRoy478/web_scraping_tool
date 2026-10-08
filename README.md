# Company Intelligence — MERN Stack

A MERN-based company intelligence and web data extraction application. The project demonstrates a practical company research workflow: submit one or more company URLs, crawl selected public pages, extract structured business information, optionally enrich the extracted data with an LLM, persist results in MongoDB, and export the collected data as JSON or CSV.

## Technology Stack

- **Frontend:** React 19 + Vite
- **Backend:** Node.js + Express 5
- **Database:** MongoDB + Mongoose
- **HTTP/HTML extraction:** Axios + Cheerio
- **Dynamic page extraction:** Playwright/Chromium
- **Scheduling:** node-cron
- **Rate limiting:** express-rate-limit
- **Robots.txt:** robots-parser
- **Optional AI enrichment:** OpenAI API
- **Development:** Nodemon + concurrently

## Implemented Features

### Core / Minimal Features

- URL-based company research input
- Validation of HTTP/HTTPS URLs
- Public webpage fetching
- Structured extraction of:
  - Company/site name
  - Website URL
  - Meta/OG description
  - Public email addresses
  - Public phone numbers
  - Social media links
  - Detected technologies
  - Products/services from page headings
  - Legal signals such as company-registration/legal wording
  - Market/business signals such as customers, funding, pricing, investors, etc.
- Extraction error handling and per-URL logging
- MongoDB persistence of extracted company records
- Job status tracking
- Searchable company records through the API
- JSON export
- CSV export

### Enhancement Features

- **Dynamic content extraction:** Playwright can render JavaScript-driven pages.
- **Internal link discovery:** The crawler can discover additional same-domain pages.
- **Pagination heuristics:** Links containing terms such as `page`, `next`, or `more` can be added to the crawl queue.
- **Rate limiting/fair-use delay:** Requests are spaced to avoid sending rapid repeated requests to a target site.
- **robots.txt check:** URLs are checked against the target site's robots.txt before extraction.
- **Scheduled extraction:** Jobs can be scheduled using standard cron expressions.
- **AI enrichment:** When an OpenAI API key is configured, extracted content can be normalized into fields such as industry, location, competitors, products, services, technologies, and market/legal signals.
- **Operational logging:** Extraction activity and errors are persisted in MongoDB.
- **Configurable crawl options:** Dynamic rendering, link discovery, pagination, enrichment, and maximum pages can be controlled per job.

## Data Extraction Level Demonstrated

This implementation demonstrates **Level 2 / Medium Data Extraction**, with selected enhancement capabilities from the higher levels.

### Level 1 — Basic extraction

The application extracts the core public company information available directly from a webpage: name/title, website, description, public email addresses, phone numbers, social links, and source URL.

### Level 2 — Medium extraction (chosen demonstration level)

The implementation extends Level 1 with:

- Product/service signals from headings
- Technology detection from page content
- Legal/registration-related signals
- Market/business signals
- Internal-page discovery
- Multiple-page crawling
- MongoDB persistence and searchable company records
- JSON/CSV export

This level was chosen because it demonstrates meaningful structured company intelligence without requiring unreliable assumptions about information that may not be publicly available on every website.

### Enhancement / advanced capabilities

The code also includes optional dynamic rendering, pagination heuristics, scheduled jobs, and LLM-based enrichment. These are deliberately optional because they add cost, runtime, or site-specific complexity.

## How the Extraction Works

1. The user submits one or more public company URLs.
2. The API validates the URLs.
3. The server checks `robots.txt` before requesting a page.
4. Requests are rate-limited and spaced with a small fair-use delay.
5. The server fetches HTML using Axios, or renders the page with Playwright when dynamic extraction is enabled.
6. Cheerio parses the HTML.
7. The extractor collects public business information using deterministic rules and heuristics.
8. Optional internal links and likely pagination links are added to the crawl queue.
9. Optional AI enrichment normalizes the extracted information without intentionally inventing missing facts.
10. The final company record is upserted into MongoDB.
11. Jobs and extraction events are stored for status tracking and troubleshooting.
12. The collected company records can be exported as JSON or CSV.

## Project Structure

```text
company-intelligence-mern/
├── package.json
├── README.md
├── client/
│   ├── package.json
│   ├── index.html
│   └── src/
│       ├── main.jsx
│       └── style.css
└── server/
    ├── package.json
    ├── .env.example
    └── index.js
```

## Prerequisites

Install the following before running the project:

- **Node.js 20+** recommended
- **npm**
- **MongoDB** local installation or a MongoDB Atlas database
- Optional: OpenAI API key for AI enrichment

## Setup and Run

### 1. Open the project root

On Windows PowerShell:

```powershell
cd C:\Users\ASUTOSH\Downloads\company-intelligence-mern\company-intelligence-mern
```

### 2. Install root dependencies

```powershell
npm install
```

This installs development tooling including `concurrently`.

### 3. Install client and server dependencies

```powershell
npm run install:all
```

If you prefer to install them separately:

```powershell
cd server
npm install
cd ..\client
npm install
cd ..
```

### 4. Configure the backend

Copy:

```text
server/.env.example
```

to:

```text
server/.env
```

Example:

```env
PORT=5000
MONGODB_URI=mongodb://127.0.0.1:27017/company_intelligence
CLIENT_URL=http://localhost:5173
OPENAI_API_KEY=
```

For MongoDB Atlas, replace `MONGODB_URI` with the Atlas connection string.

### 5. Install the Playwright browser

Only required when using dynamic extraction:

```powershell
cd server
npx playwright install chromium
cd ..
```

### 6. Start the application

From the project root:

```powershell
npm run dev
```

The root script starts both applications concurrently.

- Frontend: `http://localhost:5173`
- Backend API: `http://localhost:5000`
- Health check: `http://localhost:5000/api/health`

### If `concurrently` is not recognized

Run:

```powershell
npm install --save-dev concurrently
npm run dev
```

Make sure the command is executed from the **project root**, not from the `client` directory.

## MongoDB

The server defaults to:

```text
mongodb://127.0.0.1:27017/company_intelligence
```

Make sure MongoDB is running before starting the server. The application creates the required collections through Mongoose when data is first written.

## Optional AI Enrichment

AI enrichment is disabled unless `OPENAI_API_KEY` is configured.

Set the key in `server/.env`:

```env
OPENAI_API_KEY=your_api_key_here
```

The enrichment prompt explicitly asks the model not to invent facts and uses empty values for unknown information. AI enrichment is therefore treated as a normalization/enrichment layer rather than the primary source of truth.

## API Overview

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/api/health` | Backend health check |
| GET | `/api/companies` | List/search extracted companies |
| GET | `/api/jobs` | List extraction jobs |
| GET | `/api/logs` | View extraction logs |
| POST | `/api/jobs` | Create and execute/schedule an extraction job |
| POST | `/api/jobs/:id/run` | Run an existing job again |
| GET | `/api/export?format=json` | Export company data as JSON |
| GET | `/api/export?format=csv` | Export company data as CSV |

### Example job payload

```json
{
  "query": "Example company",
  "urls": ["https://example.com"],
  "options": {
    "dynamic": false,
    "discoverLinks": true,
    "pagination": true,
    "enrich": false,
    "maxPages": 10
  }
}
```

For a scheduled job, provide a cron expression in the `schedule` field.

## Design Decisions and Assumptions

### 1. Public web data only

The application is designed for publicly accessible webpage information. It does not attempt to bypass authentication, paywalls, access controls, or anti-bot protections.

### 2. Deterministic extraction first

Core fields are extracted with HTML parsing and pattern-based rules rather than relying entirely on an LLM. This makes the baseline extraction predictable, cheaper, and easier to debug.

### 3. Level 2 as the primary demonstration

Level 2 provides a useful balance between implementation depth and reliability. Higher-level company intelligence can become highly dependent on the target website's structure and the availability of authoritative third-party sources.

### 4. Heuristic product/service extraction

Products and services are initially represented by relevant page headings. This is intentionally conservative: it avoids presenting arbitrary page text as verified products or services.

### 5. Technology detection is heuristic

Technology detection looks for known technology names in the downloaded HTML. It is useful as a demonstration but should not be treated as a full browser-fingerprinting or technology-intelligence system.

### 6. Pagination is heuristic

Pagination discovery looks for likely links containing words such as `page`, `next`, or `more`. Different websites implement pagination differently, so this is intentionally a lightweight demonstration rather than a universal pagination engine.

### 7. Same-domain crawling

Discovered pages are limited to the same hostname as the submitted URL. This prevents an ordinary company crawl from unexpectedly expanding into unrelated domains.

### 8. Crawl limits

A configurable maximum page count is used to control runtime and resource consumption. The server also caps the maximum crawl size to 30 pages per job.

### 9. robots.txt and fair-use behavior

The crawler checks robots.txt and introduces a delay between requests. This is intended to demonstrate responsible crawling. Website terms, applicable law, and target-site policies should always be respected.

### 10. AI is optional

The application remains useful without an OpenAI API key. This keeps the basic demonstration deterministic and avoids making the project dependent on a paid external service.

## Limitations / Future Improvements

- More sophisticated entity extraction could improve company name, address, and executive detection.
- A production system should use a persistent queue such as BullMQ/Redis for large crawl workloads instead of in-process job execution.
- Scheduling currently lives in the application process; a production deployment should use a durable scheduler/worker architecture.
- Technology detection could be improved with a dedicated technology fingerprinting approach.
- Pagination could use structured rel attributes, sitemap discovery, and site-specific strategies.
- Authentication/authorization should be added before exposing the API publicly.
- MongoDB indexes and pagination should be added for large datasets.
- More comprehensive automated tests should be added for extraction rules and API behavior.

## Responsible Use

Use the application only on websites and data that you are permitted to access and process. Respect robots.txt, website terms of service, rate limits, privacy requirements, and applicable laws. Contact information extraction is limited to publicly available page content.
