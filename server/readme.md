# Satark AI API Server

This directory contains the Node.js and Express API used by Satark AI. It connects to MongoDB, validates Auth0 bearer tokens for protected routes, stores user/search data, and proxies document-generation requests to Langflow.

The server is separate from the Python RAG services in `rag/` and `suraksha_setu/`.

## Requirements

- Node.js 18 or newer
- npm
- MongoDB
- Auth0 API/application configuration
- A Langflow endpoint and Astra token if document generation is used

## Setup

```bash
cd server
npm install
```

Create `server/.env`:

```env
PORT=3000
DB_CONNECT=mongodb://127.0.0.1:27017/satark-ai
AUTH0_DOMAIN=your-tenant.us.auth0.com
AUTH0_AUDIENCE=https://api.satark.ai
LANGFLOW_API_URL=https://your-langflow-endpoint
ASTRA_TOKEN=your-astra-token
```

`DB_CONNECT`, `AUTH0_DOMAIN`, and `AUTH0_AUDIENCE` are required for the API and protected routes. `LANGFLOW_API_URL` and `ASTRA_TOKEN` are required by `/proxy/generate`.

## Run

The package currently provides a development script but no `start` script:

```bash
node server.js       # Start the API directly
npm run dev           # Start with nodemon when nodemon is available
```

The default port is `3000`. The root endpoint is available at `http://localhost:3000/`.

## Mounted Endpoints

### Public

| Method | Path | Purpose |
| --- | --- | --- |
| GET | `/` | Basic server response |
| POST | `/proxy/generate` | Forwards a document-generation request to Langflow |

The proxy accepts the JSON request body and an optional `stream` query parameter. It forwards the `Authorization` header generated from `ASTRA_TOKEN` to Langflow.

### Authenticated

Send an Auth0 access token in the header:

```http
Authorization: Bearer <access-token>
```

| Method | Path | Purpose |
| --- | --- | --- |
| GET | `/authorized` | Authenticated smoke test |
| GET | `/users/profile` | Find or create the MongoDB user profile from Auth0 claims |
| POST | `/users/logout` | Add the bearer token to the blacklist and clear the cookie |
| POST | `/legal/search` | Search legal knowledge through the configured RAG bridge |
| GET | `/legal/history` | Return the authenticated user's latest legal searches |
| GET | `/legal/history/:id` | Return one legal search record |

Example legal search request:

```bash
curl -X POST http://localhost:3000/legal/search \
  -H 'Authorization: Bearer <access-token>' \
  -H 'Content-Type: application/json' \
  -d '{"query":"What is the procedure for preserving evidence?"}'
```

## Project Structure

```text
server/
├── app.js                 Express middleware and mounted routes
├── server.js              HTTP server and Langflow proxy
├── controllers/           Request handlers
├── db/                    MongoDB connection
├── middlewares/           Auth0 and token blacklist checks
├── models/                Mongoose models
├── routes/                Express route definitions
└── services/              Reusable server services
```

## Related Services

The client also calls the Python services directly:

- `rag/main.py`: legal QA at `/qa`, investigation analysis at `/investigation`, and status at `/health`.
- `suraksha_setu/main.py`: tactical, command, security, evacuation, and evidence endpoints.

Install and run those services using their own `requirements.txt` files and the `GROQ_API_KEY` environment variable.
