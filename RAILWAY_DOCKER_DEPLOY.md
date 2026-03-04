# Railway (Auth+Backend) + Vercel (Frontend)

This guide deploys API services on Railway and frontend on Vercel.

## 1) Prerequisites

- Docker daemon running
- Docker Hub account
- Railway account + project
- MongoDB URI(s)

Login once to Docker Hub:

```bash
docker login
```

If your environment has no `buildx` plugin, use classic builder:

```bash
export DOCKER_BUILDKIT=0
```

## 2) Build and Push Images

### Auth image (`clean-auth` repo)

```bash
cd /home/zippy/clean-auth
npm run docker:publish -- <DOCKERHUB_USER> <VERSION_TAG>
```

Published as:

- `<DOCKERHUB_USER>/clean-auth:<VERSION_TAG>`
- `<DOCKERHUB_USER>/clean-auth:latest`

### Backend image (`clean-periodic-table/Backend`)

```bash
cd /home/zippy/clean-periodic-table/Backend
npm run docker:publish -- <DOCKERHUB_USER> <VERSION_TAG>
```

Published as:

- `<DOCKERHUB_USER>/clean-periodic-table-backend:<VERSION_TAG>`
- `<DOCKERHUB_USER>/clean-periodic-table-backend:latest`

## 3) Railway Service Setup (Deploy from Image)

Create Railway services only for APIs:

- `auth` -> `<DOCKERHUB_USER>/clean-auth:<VERSION_TAG>`
- `backend` -> `<DOCKERHUB_USER>/clean-periodic-table-backend:<VERSION_TAG>`

## 4) Environment Variables to Fill

Use these templates:

- Backend: `/home/zippy/clean-periodic-table/Backend/railway.env.example`
- Auth: `/home/zippy/clean-auth/railway.env.example`
- Frontend (Vercel): `/home/zippy/clean-periodic-table/frontend/DEPLOY_VERCEL.md`

### Auth required vars

- `NODE_ENV=production`
- `DATA_SOURCE=mongo`
- `MONGO_URI=<YOUR_AUTH_MONGO_URI>`
- `JWT_ACCESS_SECRET=<STRONG_SECRET>`
- `JWT_REFRESH_SECRET=<STRONG_SECRET>`
- `CORS_ORIGINS=https://<FRONTEND_DOMAIN>`
- `COOKIE_SAME_SITE=none`
- `COOKIE_SECURE=true`
- `COOKIE_DOMAIN=` (optional)

Notes:

- Railway injects `PORT`; no fixed value required.
- `APPLICATION_PORT` is optional fallback.

### Backend required vars

- `NODE_ENV=production`
- `HOST=0.0.0.0`
- `DATA_SOURCE=mongo`
- `MONGO_URI=<YOUR_BACKEND_MONGO_URI>`
- `AUTH_REQUIRED=true`
- `AUTH_SERVICE_URL=https://<AUTH_PUBLIC_DOMAIN>`
- `AUTH_VALIDATE_PATH=/validate-token`
- `CORS_ORIGINS=https://<FRONTEND_DOMAIN>`

Note:

- Railway injects `PORT`; no fixed value required.

### Frontend required vars (Vercel project)

- `NEXT_PUBLIC_AUTH_API_URL=https://<AUTH_PUBLIC_DOMAIN>`
- `NEXT_PUBLIC_BACKEND_API_URL=https://<BACKEND_PUBLIC_DOMAIN>`

Important:

- Set these in Vercel Project Environment Variables.
- If they change, trigger a new Vercel deployment.

## 5) CORS and Cookies for Production

- `CORS_ORIGINS` in both auth/backend must include the frontend domain exactly.
- Use comma-separated list if needed: `https://app.example.com,https://staging.example.com`.
- For cross-domain auth cookie in browsers, keep `COOKIE_SAME_SITE=none` and `COOKIE_SECURE=true`.

## 6) Smoke Checks

After deploy:

- Auth health/root: `GET https://<AUTH_PUBLIC_DOMAIN>/`
- Register: `POST https://<AUTH_PUBLIC_DOMAIN>/register`
- Backend health: `GET https://<BACKEND_PUBLIC_DOMAIN>/health`
- Frontend flow: login/register and fetch `/elements`

## 7) Vercel Frontend Setup

- Import monorepo in Vercel
- Set Root Directory to `frontend`
- Configure:
  - `NEXT_PUBLIC_AUTH_API_URL=https://<AUTH_PUBLIC_DOMAIN>`
  - `NEXT_PUBLIC_BACKEND_API_URL=https://<BACKEND_PUBLIC_DOMAIN>`
- After Vercel domain is assigned, set that domain in Railway:
  - Auth `CORS_ORIGINS`
  - Backend `CORS_ORIGINS`
