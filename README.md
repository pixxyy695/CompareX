# CompareX

CompareX is an automated price comparison tool that checks the price of a product across **Amazon**, **Flipkart**, and **Myntra** and tells you which platform has the best deal. Instead of scraping websites, it drives real mobile apps in the cloud (via [MobileRun](https://mobilerun.ai)) using an LLM agent, extracts the price for each platform, and compares the results.

The project has two parts:
- **`backend/`** — a Flask API that kicks off MobileRun tasks for each platform, polls them for completion, and computes the cheapest option.
- **`frontend/`** — a React + Vite + Tailwind CSS single-page app where you search for a product and see live results.

## How it works

1. You type a product name in the frontend search bar.
2. The frontend calls the Flask backend (`/api/search`).
3. The backend creates a MobileRun cloud task for each platform (Amazon, Flipkart, Myntra), instructing an LLM-driven agent to open the platform's app, search for the product, and read off the price of the first result.
4. The backend polls each task until it completes (or times out), collects the prices, and works out which platform is cheapest.
5. The frontend displays all prices side by side and highlights the best deal.

## Tech stack

| Layer      | Technology                                   |
|------------|-----------------------------------------------|
| Frontend   | React 18, Vite, Tailwind CSS, Axios           |
| Backend    | Python, Flask, Flask-CORS, Requests           |
| Automation | [MobileRun Cloud](https://mobilerun.ai) (LLM-driven mobile app automation) |

## Project structure

```
CompareX/
├── backend/
│   ├── app.py            # Main Flask API (parallel task dispatch + polling)
│   ├── run.py            # Alternate entrypoint (sequential polling, extra test routes)
│   └── requirements.txt  # Python dependencies
└── frontend/
    ├── src/
    │   ├── App.jsx               # Main app + search/result orchestration
    │   ├── main.jsx              # React entry point
    │   ├── index.css             # Tailwind base styles
    │   └── components/
    │       ├── SearchBar.jsx     # Product search input
    │       ├── PriceResults.jsx  # Renders per-platform prices + lowest price
    │       └── LoadingSpinner.jsx
    ├── index.html
    ├── package.json
    ├── tailwind.config.js
    ├── postcss.config.js
    └── vite.config.js
```

## Prerequisites

- Python 3.9+
- Node.js 18+ and npm
- A [MobileRun](https://mobilerun.ai) account with an API key and at least one registered Android device that has the Amazon, Flipkart, and Myntra apps installed

## Setup

### 1. Backend

```bash
cd backend
python -m venv venv
source venv/bin/activate      # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

Open `app.py` (or `run.py`, if you're using that entrypoint) and set your own credentials:

```python
API_KEY = 'YOUR_MOBILERUN_API_KEY_HERE'
...
'deviceId': 'YOUR_DEVICE_ID_HERE',
```

> **Note:** These values are currently hardcoded for local development. See [Configuration](#configuration) below for a safer approach.

Run the server:

```bash
python app.py
```

The API will start on `http://localhost:5000`.

### 2. Frontend

```bash
cd frontend
npm install
npm run dev
```

The app will be available at `http://localhost:5173` (Vite's default port) and is configured to call the backend at `http://localhost:5000`.

## API endpoints

| Method | Endpoint             | Description                                              |
|--------|-----------------------|------------------------------------------------------------|
| GET    | `/`                   | Health/status check                                        |
| GET    | `/health`             | Simple health check                                         |
| POST   | `/api/test`           | Returns mock price data, useful for testing the frontend without hitting MobileRun |
| POST   | `/api/search`         | Starts (and, in `run.py`, waits for) price lookups across Amazon, Flipkart, and Myntra for a given `product_name` |
| GET    | `/api/task/<task_id>` | Checks the status/result of a single MobileRun task (used by `app.py`'s async flow) |
| POST   | `/api/compare`        | Given a set of raw price strings, extracts numeric values and returns the lowest |

Example request:

```bash
curl -X POST http://localhost:5000/api/search \
  -H "Content-Type: application/json" \
  -d '{"product_name": "boat headphones"}'
```

## Configuration

Before deploying or sharing this project, move the following out of source code and into environment variables or a `.env` file (not committed to git):

- `MOBILERUN_API_KEY`
- `MOBILERUN_DEVICE_ID`

The Flask backend can then load them with something like:

```python
import os
API_KEY = os.environ.get("MOBILERUN_API_KEY")
DEVICE_ID = os.environ.get("MOBILERUN_DEVICE_ID")
```

## Known limitations

- Price lookups depend on a live mobile automation task per platform, so a search can take anywhere from several seconds to a few minutes.
- `app.py` and `run.py` overlap significantly (parallel task dispatch vs. sequential polling with extra debug routes) — pick one as your canonical backend entrypoint.
- Currently supports only Amazon, Flipkart, and Myntra, and only the price of the first search result.

## Roadmap ideas

- Add more platforms
- Cache recent results to avoid repeated automation runs for the same product
- Move API keys/device IDs to environment variables
- Add automated tests for price extraction and comparison logic
