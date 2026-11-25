# AirGuard — Know Your Air. Plan Your Day. Protect Your Tomorrow.

An AI-powered air quality intelligence platform for Indian cities. AirGuard helps users understand local pollution, plan outdoor activities and take practical action — not just read an AQI number.

**Default location:** Udaipur, Rajasthan.

---

## Running the Application

### Prerequisites
- Python 3.10+
- Node.js 18+

### Backend (FastAPI)

```powershell
Set-Location "c:\IBM_airguard\backend"
pip install -r requirements.txt
python -m uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

**Backend URL:** http://localhost:8000  
**API Docs (Swagger):** http://localhost:8000/docs

### Frontend (React + Vite)

```powershell
Set-Location "c:\IBM_airguard\frontend"
npm install
npm run dev
```

**Frontend URL:** http://localhost:5173

---

## Configuration (backend/.env)

Copy `.env.example` to `.env` and configure:

```env
# PRIMARY: OpenAQ API v3 — get free key at https://explore.openaq.org/register
OPENAQ_API_KEY=your_openaq_api_key_here

# FALLBACK: WAQI API — get free token at https://aqicn.org/api/
WAQI_API_TOKEN=your_waqi_token_here

# DATA_MODE: "live" uses APIs above | "demo" uses illustrative data (default)
DATA_MODE=demo

CORS_ORIGINS=http://localhost:5173,http://localhost:3000
```

**To enable live data:** Set `DATA_MODE=live` and add your `OPENAQ_API_KEY`.

---

## Features

| Feature | Status | Notes |
|---|---|---|
| AQI Dashboard | ✅ Working | Udaipur default; all 9 cities |
| Air Quality Compass | ✅ Working | CPCB-calibrated semicircular gauge |
| CPCB AQI Calculation | ✅ Working | Correct Indian sub-index methodology |
| Pollutant Breakdown | ✅ Working | PM2.5, PM10, NO2, SO2, CO, O3 |
| 7-Day Trend Chart | ✅ Working | Demo trend data (illustrative) |
| 3-Day Forecast Estimate | ✅ Working | Linear trend estimate, clearly labelled |
| Health Advisory | ✅ Working | By AQI category + user profile |
| Plan Your Day | ✅ Working | Morning/afternoon/evening/night slots |
| Why This Recommendation | ✅ Working | Shown in Plan Your Day section |
| Ask AirGuard Chat | ✅ Working | Routes to 4 agents by intent |
| Personalized Advisory | ✅ Working | Adult / Child / Older Adult / Sensitive |
| Community Action Wall | ✅ Working | Pledge actions, weekly plan |
| Explore Map | ✅ Working | Leaflet + OpenStreetMap |
| GPS Location | ✅ Working | Browser geolocation API |
| City/location search | ✅ Working | Nominatim (OpenStreetMap geocoding) |
| OpenAQ Live Stations | ✅ Integrated | Requires OPENAQ_API_KEY + DATA_MODE=live |
| Station Markers on Map | ✅ Working | With AQI bubbles and popup details |
| Radius Filter | ✅ Working | 5 km / 10 km / 25 km / 50 km |
| Data Status Labels | ✅ Working | LIVE / CACHED / DEMO / UNAVAILABLE |
| LIVE AQI data | ⚙️ Requires key | Configure OPENAQ_API_KEY |
| Live 7-day history | ❌ Not implemented | OpenAQ historical API needed |
| Official weather forecast | ❌ Not implemented | No weather API configured |

---

## API Endpoints

| Endpoint | Description |
|---|---|
| `GET /health` | Health check + data source status |
| `GET /cities` | List cities + coordinates |
| `GET /aqi/{city}` | Current AQI for a city |
| `GET /dashboard/{city}?profile=adult` | Full dashboard (all 4 agents) |
| `GET /trend/{city}` | 7-day trend data |
| `POST /stations/nearby` | OpenAQ stations near lat/lon |
| `GET /stations/city/{city}` | OpenAQ stations near city |
| `POST /chat` | Chat with agent coordinator |
| `GET /knowledge/search?q=...` | Knowledge base search |

---

## AQI Accuracy

AirGuard uses the **Indian CPCB National AQI methodology** (not US AQI):

| Range | Category |
|---|---|
| 0–50 | Good |
| 51–100 | Satisfactory |
| 101–200 | Moderately Polluted |
| 201–300 | Poor |
| 301–400 | Very Poor |
| 401–500 | Severe |

Sub-index breakpoints from CPCB NAAQS 2009 are implemented in `backend/services/aqi_service.py`.

**Data honesty rules enforced in code:**
- Demo data is always labelled "DEMONSTRATION DATA"
- Readings older than 6 hours are labelled "CACHED"
- Missing live API = "UNAVAILABLE" (never replaced with demo values)
- Forecasts are labelled "estimates" and point to SAFAR for official forecasts
- Station distance + measurement timestamp always shown on map

---

## Agent Architecture

```
User Request
     │
     ▼
  Coordinator (agents/coordinator.py)
     │
     ├── AQI Monitoring Agent    — aqi_agent.py
     ├── Health Advisory Agent   — health_agent.py
     ├── Forecasting Agent       — forecast_agent.py
     └── Community Action Agent  — community_agent.py
```

Chat queries are routed by keyword intent:
- **Health intent** (safe, outdoor, mask, precaution) → Health Advisory Agent
- **Forecast intent** (tomorrow, trend, improve, worsen) → Forecasting Agent
- **Community intent** (community, carpool, reduce, tree) → Community Action Agent
- **AQI intent** (aqi, pollutant, pm2.5, reading) → AQI Monitoring Agent

---

## Explore Map

The Explore Map page uses:
- **Leaflet** + **OpenStreetMap** (no API key required)
- **OpenAQ v3 `/locations` API** for station search (requires `OPENAQ_API_KEY`)
- **Nominatim** for location search (OpenStreetMap geocoding, no key required)
- Browser **Geolocation API** (requires user permission)

**Accuracy note:** Station measurements reflect the station's location, not the user's exact GPS coordinates. Distance and timestamp are always displayed.

---

## Knowledge Base

Verified sources embedded in `backend/services/knowledge_service.py`:
- CPCB — National Air Quality Index
- CPCB NAAQS 2009
- WHO Global Air Quality Guidelines 2021
- MoEFCC guidelines
- CAQM (Commission for Air Quality Management)
- SAFAR (System of Air Quality and Weather Forecasting)

---

## Project Structure

```
airguard/
├── backend/
│   ├── main.py                    # FastAPI routes
│   ├── config.py                  # Settings (pydantic-settings)
│   ├── requirements.txt
│   ├── .env.example
│   ├── agents/
│   │   ├── aqi_agent.py
│   │   ├── health_agent.py
│   │   ├── forecast_agent.py
│   │   ├── community_agent.py
│   │   └── coordinator.py
│   └── services/
│       ├── aqi_service.py         # OpenAQ v3 + WAQI + demo fallback
│       └── knowledge_service.py   # CPCB/WHO knowledge base
└── frontend/
    └── src/
        ├── App.tsx                # Main app + navigation
        ├── api.ts                 # Typed API client
        ├── utils.ts               # Helpers
        └── components/
            ├── AirQualityCompass.tsx  # SVG compass gauge
            ├── Dashboard.tsx          # Main dashboard
            ├── TrendChart.tsx         # Recharts area chart
            ├── PollutantGrid.tsx      # Pollutant cards
            ├── ChatPanel.tsx          # AI chat interface
            ├── CitySelector.tsx       # City picker
            ├── ProfileSelector.tsx    # Health profile
            ├── CommunityWall.tsx      # Community actions
            ├── ExploreMap.tsx         # Leaflet map page
            └── UI.tsx                 # Shared UI components
```
