# Stat&Mat (webstranicaV2)

Lightweight Node/Express site for offering mathematics and statistics video courses and subscriptions.

# Stat&Mat (webstranicaV2)

Professional, minimal web platform for delivering mathematics and statistics video lessons. The app includes user registration, email confirmation, Google OAuth sign-in, subscription payments via Stripe, and protected video streaming.

## Tech stack

-  Node.js + Express
-  MongoDB (Mongoose)
-  EJS + static frontend (under `public/`)
-  Stripe Checkout for payments
-  nodemailer for outgoing email
-  passport-google-oauth20 for Google sign-in

## Highlights / Features

-  User registration with email confirmation (JWT-signed tokens).
-  Google OAuth sign-in and session management.
-  Session persistence in MongoDB via `connect-mongo`.
-  Stripe Checkout integration for paid subscriptions.
-  Protected video streaming via proxy endpoints that enforce subscription checks.
-  Device registration per user to limit how many distinct devices a user can register.
   -  Default device limit in server logic: 2 devices. To allow 3 devices, update the numeric checks in `server.js` (search for `user.devices.length >= 2` and change `2` to `3`).
-  Responsive frontend served from `public/` with a few EJS templates under `views/`.

## Repository layout

-  `server.js` — main Express application, routes, auth, and payments
-  `models/user.js` — Mongoose schema for users and device records
-  `public/` — static frontend HTML/CSS/JS
-  `views/` — EJS templates
-  `.env` — environment variables (local, not committed)

## Environment variables

Create a `.env` file at the project root (next to `server.js`). The app expects these keys:

-  `NODE_ENV` — `development` or `production`
-  `PORT` — server port (default `3000`)
-  `BASE_URL` — full app URL (e.g. `http://localhost:3000`)
-  `MONGODB_URI` — MongoDB connection string (local or Atlas)
-  `STRIPE_SECRET_KEY` — Stripe secret key (`sk_test_...` for development)
-  `EMAIL_SECRET` — JWT secret for email tokens
-  `RANDOM_ENCRYPT` — 32-character base64 (or 32-byte hex) key used for AES encryption
-  `EMAIL_HOST`, `EMAIL_PORT`, `EMAIL_USER`, `EMAIL_PASS` — SMTP configuration for nodemailer
-  `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET` — Google OAuth credentials

Example (sanitized) `.env` snippet:

```
NODE_ENV=development
PORT=3000
BASE_URL=http://localhost:3000
MONGODB_URI=mongodb://localhost:27017/statandmat
STRIPE_SECRET_KEY=sk_test_xxx
EMAIL_SECRET=your_jwt_secret_here
RANDOM_ENCRYPT=base64_or_hex_key_here
EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=465
EMAIL_USER=you@example.com
EMAIL_PASS=app_password_here
GOOGLE_CLIENT_ID=...
GOOGLE_CLIENT_SECRET=...
```

> IMPORTANT: never commit `.env` to source control. Add it to `.gitignore`.

## Setup & run (development)

1. Install dependencies

```powershell
npm install
```

2. Create and populate `.env` (see above).

3. Start the server

```powershell
npm start
# or
node server.js
```

4. Open your browser to `http://localhost:3000` (or your configured `BASE_URL`).

## Important endpoints

-  `POST /api/register` — register a user; sends confirmation email
-  `GET /confirmation/:token` — confirm email address
-  `POST /api/login` — login (username/email + password)
-  `POST /api/logout` — log out and destroy session
-  `GET /api/check-login` — returns session login state
-  `POST /checkout1`, `POST /checkout2` — create Stripe Checkout sessions
-  `GET /proxy/1video`, `GET /proxy/2video` — proxy video endpoints (subscription-protected)

## Frontend

The frontend is a responsive static UI in `public/` (HTML/CSS/JS) with a few server-rendered EJS pages in `views/`. The homepage shows subscription plans, register/login flows, and conditionally displays links to subscription content when a user has access.

## Device registration / limit

When users sign in the app records devices (IP + user agent) in the user's `devices` array. The server prevents registering more than the configured limit to avoid account sharing. The default hard-coded limit is `2` devices; change the limit by editing the numeric checks in `server.js` (both in Google OAuth handling and in the login flow).

## Stripe integration

-  Use **test** keys from the Stripe Dashboard for local development (enable **View test data**).
-  The app uses Stripe Checkout and redirects to success/cancel URLs configured with `BASE_URL`.

## Security notes

-  Keep all secrets out of version control. Use your host's secret manager in production.
-  Use app-specific passwords for mail providers (Gmail) and enable 2FA.
-  Regenerate API keys if you suspect a leak.
