# Emergency Evacuation Route Planner — Backend

A small FastAPI service for the frontend prototype. It serves one fictional venue graph and calculates routes around blocked edges and closed exits. All venue data is static and all incident state is supplied per request; the API does not persist incidents.

> **Safety:** This is a software demo using fictional data, not certified life-safety software. Do not use it to direct a real evacuation. Real deployment requires verified venue maps, reliable incident reporting, accessibility review, operational resilience, and approval from relevant safety authorities.

## Run locally

```bash
cd /home/ubuntu/evacuation-route-backend
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

Interactive API docs: `http://localhost:8000/docs`  
Health check: `GET http://localhost:8000/health`

Set allowed frontend origins as a comma-separated list before launch. The local Vite origin is allowed by default.

```bash
CORS_ORIGINS="http://localhost:5173,https://your-frontend.example" uvicorn app.main:app --host 0.0.0.0 --port 8000
```

## Endpoints

| Method | Path | Purpose |
|---|---|---|
| `GET` | `/health` | Liveness check |
| `GET` | `/api/v1/venues/demo-building` | Return the fictional venue, nodes, edges, and safety notice |
| `POST` | `/api/v1/routes` | Calculate a best route and optional alternate exits |

### Route request

```json
{
  "venueId": "demo-building",
  "startNodeId": "central",
  "blockedEdgeIds": ["north-central"],
  "closedExitIds": [],
  "avoidStairs": false,
  "maxAlternatives": 2
}
```

### Response shape

A successful response includes `routeFound: true`, a `route` with exit, ordered node/edge IDs, approximate distance, and plain-language directions, plus alternate reachable exits. When no route is available, the API returns HTTP 200 with `routeFound: false`, `route: null`, and a clear message. Unknown venues return 404; unknown node, edge, or exit IDs return 422.

The route engine uses Dijkstra’s algorithm. Edges are treated as bidirectional in this sample. Blocked paths, closed exits, stairs, and inaccessible edges are excluded as requested. Distance values are approximate demo values.

## Frontend integration

Fetch the venue once on startup:

```ts
const venue = await fetch(`${API_BASE}/api/v1/venues/demo-building`).then(r => r.json());
```

Request a route whenever the start location or scenario controls change:

```ts
const result = await fetch(`${API_BASE}/api/v1/routes`, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    venueId: "demo-building",
    startNodeId,
    blockedEdgeIds,
    closedExitIds,
    avoidStairs,
    maxAlternatives: 1,
  }),
}).then(r => r.json());
```

Use `result.route.edgeIds` to highlight the route, `result.blockedEdgeIds` and `result.closedExitIds` for incident styles, and suppress the route overlay when `result.routeFound` is false. Keep the safety notice visible in the UI.

## Tests

```bash
pip install -r requirements.txt
pytest -q
```
