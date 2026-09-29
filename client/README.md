# Satark AI Client

The client is a React 19 single-page application built with Vite. It provides the public landing, login, and registration screens, plus an authenticated portal for legal knowledge, document generation, investigation, and Suraksha Setu analysis.

## Requirements

- Node.js 18 or newer
- npm
- An Auth0 application configured for the local and deployed callback URLs

## Setup

From the repository root:

```bash
cd client
npm install
```

Create `client/.env.local` with the Auth0 settings used by `src/main.jsx`:

```env
VITE_AUTH0_DOMAIN=your-tenant.us.auth0.com
VITE_AUTH0_CLIENT_ID=your-auth0-client-id
VITE_AUTH0_AUDIENCE=https://api.satark.ai
```

`VITE_API_BASE_URL` is also read by the document generator for the proxy request. Set it to the server origin when using a local backend:

```env
VITE_API_BASE_URL=http://localhost:3000
```

The legal knowledge and investigation services currently use the deployed RAG URL defined directly in `src/services/legalApi.js` and `src/services/engineApi.js`. Change those constants when running the RAG service locally.

## Commands

```bash
npm run dev       # Start the Vite development server
npm run build     # Create a production build in dist/
npm run preview   # Preview the production build locally
npm run lint      # Run ESLint
```

The development server normally runs at `http://localhost:5173`.

## Application Routes

Public routes:

- `/` - Landing page
- `/login` - Auth0 login entry point
- `/register` - Auth0 registration entry point

Authenticated portal routes:

- `/dashboard` - Dashboard and sample crime analytics
- `/dashboard/knowledge` - Legal knowledge search
- `/dashboard/generate` - Document generation through the server proxy
- `/dashboard/investigation` - Investigation queries
- `/dashboard/analysis` - Suraksha Setu analysis workspace

Unauthenticated users are redirected away from the portal by `ProtectedRoute`.

## Service Integrations

- Auth0 is initialized in `src/main.jsx`; access tokens are obtained through `AuthContext`.
- The legal knowledge and investigation screens call the configured RAG API directly.
- Document generation calls `${VITE_API_BASE_URL}/proxy/generate`.
- Search history in `legalApi.js` is currently stored in browser `localStorage` for legal knowledge searches.
- The dashboard still includes sample analytics data and references service methods that are not implemented in the current `legalApi.js`.

## Project Structure

```text
src/
├── components/       Shared home and layout components
├── context/           Auth0-backed authentication context
├── pages/             Landing, auth, and portal screens
├── services/          Client integrations for legal and investigation APIs
├── App.jsx            React Router route definitions
├── index.css          Global styles
└── main.jsx           React and Auth0 bootstrap
```

## Deployment

Build the app with `npm run build` and serve the generated `dist/` directory. Configure the same `VITE_*` values in the deployment environment, and add the deployed origin to the Auth0 allowed callback, logout, and web origins settings.
