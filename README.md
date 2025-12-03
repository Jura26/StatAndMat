# Stat&Mat

A full-stack web application for delivering mathematics and statistics video courses with user authentication, subscription payments, and protected content streaming.

## 🚀 Live Demo

Deployed on [Render](https://statandmat-mcht.onrender.com) as a Web Service.

---

## 📋 Table of Contents

-  [Tech Stack](#-tech-stack)
-  [Architecture](#-architecture)
-  [Features](#-features)
-  [Email Strategy](#-email-strategy)
-  [Authentication Flow](#-authentication-flow)
-  [Payment Integration](#-payment-integration)
-  [Project Structure](#-project-structure)
-  [Environment Variables](#-environment-variables)
-  [Local Development](#-local-development)
-  [Deployment](#-deployment)
-  [API Endpoints](#-api-endpoints)

---

## 🛠 Tech Stack

### Backend

| Technology     | Purpose                                        |
| -------------- | ---------------------------------------------- |
| **Node.js**    | JavaScript runtime                             |
| **Express.js** | Web framework for routing, middleware, and API |
| **MongoDB**    | NoSQL database for user data and subscriptions |
| **Mongoose**   | MongoDB ODM for schema modeling and validation |

### Authentication & Security

| Technology                  | Purpose                              |
| --------------------------- | ------------------------------------ |
| **Passport.js**             | Authentication middleware            |
| **passport-google-oauth20** | Google OAuth 2.0 strategy            |
| **JSON Web Tokens (JWT)**   | Email confirmation tokens            |
| **bcrypt**                  | Password hashing                     |
| **express-session**         | Session management                   |
| **connect-mongo**           | MongoDB session store                |
| **AES Encryption**          | Device IP encryption (crypto module) |

### Email Services

| Technology       | Purpose                                |
| ---------------- | -------------------------------------- |
| **Nodemailer**   | SMTP email transport (local/fallback)  |
| **SendGrid API** | HTTP-based email delivery (production) |

### Payments

| Technology          | Purpose                       |
| ------------------- | ----------------------------- |
| **Stripe Checkout** | Secure payment processing     |
| **Stripe Webhooks** | Payment confirmation handling |

### Frontend

| Technology              | Purpose                |
| ----------------------- | ---------------------- |
| **EJS**                 | Server-side templating |
| **HTML/CSS/JavaScript** | Static frontend pages  |
| **Responsive Design**   | Mobile-friendly UI     |

### DevOps & Deployment

| Technology        | Purpose                         |
| ----------------- | ------------------------------- |
| **Render**        | Cloud hosting (Web Service)     |
| **MongoDB Atlas** | Cloud database hosting          |
| **Git/GitHub**    | Version control                 |
| **dotenv**        | Environment variable management |

---

## 🏗 Architecture

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│                 │     │                  │     │                 │
│   Client        │─ ──▶│   Express.js     │────▶│   MongoDB       │
│   (Browser)     │     │   Server         │     │   Atlas         │
│                 │◀────│                  │◀────│                 │
└─────────────────┘     └──────────────────┘     └─────────────────┘
                               │
                               │
              ┌────────────────┼────────────────┐
              │                │                │
              ▼                ▼                ▼
       ┌──────────┐    ┌──────────────┐   ┌──────────┐
       │ SendGrid │    │    Stripe    │   │  Google  │
       │   API    │    │   Checkout   │   │  OAuth   │
       └──────────┘    └──────────────┘   └──────────┘
```

---

## ✨ Features

-  **User Registration** with email confirmation (JWT-signed tokens)
-  **Google OAuth 2.0** sign-in integration
-  **Session Persistence** in MongoDB via connect-mongo
-  **Stripe Checkout** for subscription payments
-  **Protected Video Streaming** via proxy endpoints with subscription validation
-  **Device Limiting** - restrict account sharing (default: 2 devices per user)
-  **Encrypted Device Tracking** - AES-encrypted IP addresses
-  **Responsive UI** - works on desktop and mobile

---

## 📧 Email Strategy

The application implements a **dual-transport email system** to ensure reliable email delivery across different hosting environments.

### The Problem

Many cloud hosting providers (Render, Heroku, AWS, etc.) **block outbound SMTP connections** on ports 25, 465, and 587 to prevent spam abuse. This causes `ETIMEDOUT` errors when trying to send emails via traditional SMTP.

### The Solution

```javascript
// Unified sendMail helper with automatic fallback
async function sendMail({ from, to, subject, html, attachments }) {
   if (process.env.SENDGRID_API_KEY && sgMail) {
      // Production: Use SendGrid HTTP API (port 443 - not blocked)
      return sgMail.send(msg);
   } else {
      // Development: Use Nodemailer SMTP
      return transporter.sendMail({ from, to, subject, html, attachments });
   }
}
```

### How It Works

| Environment             | Email Transport   | Port        | Why                                 |
| ----------------------- | ----------------- | ----------- | ----------------------------------- |
| **Local Development**   | Nodemailer SMTP   | 465 (SSL)   | Direct SMTP works on local machines |
| **Production (Render)** | SendGrid HTTP API | 443 (HTTPS) | HTTP requests bypass SMTP blocks    |

### Configuration

**Local Development** (`.env`):

```env
EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=465
EMAIL_USER=your-email@gmail.com
EMAIL_PASS=your-app-password
```

**Production** (Render Environment Variables):

```env
SENDGRID_API_KEY=SG.xxxxx
EMAIL_USER=your-verified-sender@gmail.com
```

---

## 🔐 Authentication Flow

### Email/Password Registration

```
1. User submits registration form
2. Server hashes password with bcrypt
3. JWT token generated for email confirmation
4. Confirmation email sent via SendGrid/Nodemailer
5. User clicks link → account activated
```

### Google OAuth

```
1. User clicks "Sign in with Google"
2. Redirect to Google consent screen
3. Google returns auth code
4. Server exchanges code for user profile
5. User created/updated in database
6. Session established
```

### Device Management

```
1. On login, server captures IP + User-Agent
2. IP encrypted with AES before storage
3. Device added to user's devices array
4. If devices >= limit, login rejected
```

---

## 💳 Payment Integration

### Stripe Checkout Flow

```
1. User clicks "Subscribe" button
2. Server creates Stripe Checkout Session
3. User redirected to Stripe's hosted payment page
4. After payment, redirected to success URL
5. Server verifies session and grants access
```

### Subscription Types

-  **Mathematics - First Exam** (`/checkout1`)
-  **Mathematics - Second Exam** (`/checkout2`)

---

## 📁 Project Structure

```
statandmat/
├── server.js           # Main Express application
├── package.json        # Dependencies and scripts
├── .env                # Environment variables (not committed)
├── nodemon.json        # Nodemon configuration
├── Web.config          # IIS configuration (if needed)
│
├── models/
│   └── user.js         # Mongoose user schema
│
├── views/
│   ├── loading.ejs     # Loading/redirect page
│   ├── math1.ejs       # Math course 1 content
│   ├── math2.ejs       # Math course 2 content
│   ├── statistics1.ejs # Statistics course 1
│   └── statistics2.ejs # Statistics course 2
│
├── public/
│   ├── main.html       # Homepage
│   ├── login.html      # Login page
│   ├── register.html   # Registration page
│   ├── mathFirstExam.html
│   ├── mathSecondExam.html
│   └── sprites/        # Images and icons
│
└── data/
    └── users.json      # (Legacy/backup data)
```

---

## ⚙️ Environment Variables

Create a `.env` file in the project root:

```env
# App Configuration
NODE_ENV=development
PORT=3000
BASE_URL=http://localhost:3000

# Database
MONGODB_URI=mongodb+srv://user:pass@cluster.mongodb.net/statandmat

# Authentication
EMAIL_SECRET=your-jwt-secret-for-email-tokens
RANDOM_ENCRYPT=32-character-base64-or-hex-key

# Email (SMTP - for local development)
EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=465
EMAIL_USER=your-email@gmail.com
EMAIL_PASS=your-app-specific-password

# Email (SendGrid - for production)
SENDGRID_API_KEY=SG.your-api-key

# OAuth
GOOGLE_CLIENT_ID=your-google-client-id
GOOGLE_CLIENT_SECRET=your-google-client-secret

# Payments
STRIPE_SECRET_KEY=sk_test_your-stripe-key
```

> ⚠️ **Never commit `.env` to version control!**

---

## 🖥 Local Development

1. **Clone the repository**

   ```bash
   git clone https://github.com/Jura26/StatAndMat.git
   cd StatAndMat
   ```

2. **Install dependencies**

   ```bash
   npm install
   ```

3. **Create `.env` file** (see above)

4. **Start the server**

   ```bash
   npm start
   ```

5. **Open browser** to `http://localhost:3000`

---

## 🚀 Deployment

### Render (Recommended)

1. Create a **Web Service** (not Static Site)
2. Connect your GitHub repository
3. Configure:
   -  **Build Command**: `npm install`
   -  **Start Command**: `npm start`
4. Add all environment variables in the Environment tab
5. Deploy

### Required Production Environment Variables

| Variable               | Description                                               |
| ---------------------- | --------------------------------------------------------- |
| `NODE_ENV`             | `production`                                              |
| `BASE_URL`             | Your Render URL (e.g., `https://statandmat.onrender.com`) |
| `MONGODB_URI`          | MongoDB Atlas connection string                           |
| `SENDGRID_API_KEY`     | SendGrid API key for emails                               |
| `EMAIL_USER`           | Verified sender email                                     |
| `EMAIL_SECRET`         | JWT secret                                                |
| `RANDOM_ENCRYPT`       | AES encryption key                                        |
| `GOOGLE_CLIENT_ID`     | Google OAuth client ID                                    |
| `GOOGLE_CLIENT_SECRET` | Google OAuth secret                                       |
| `STRIPE_SECRET_KEY`    | Stripe secret key                                         |

---

## 🔌 API Endpoints

### Authentication

| Method | Endpoint               | Description                 |
| ------ | ---------------------- | --------------------------- |
| `POST` | `/api/register`        | Register new user           |
| `GET`  | `/confirmation/:token` | Confirm email               |
| `POST` | `/api/login`           | Login with credentials      |
| `POST` | `/api/logout`          | Logout and destroy session  |
| `GET`  | `/api/check-login`     | Check authentication status |
| `POST` | `/resend-email`        | Resend confirmation email   |

### OAuth

| Method | Endpoint                | Description           |
| ------ | ----------------------- | --------------------- |
| `GET`  | `/auth/google`          | Initiate Google OAuth |
| `GET`  | `/auth/google/callback` | OAuth callback        |

### Payments

| Method | Endpoint     | Description                |
| ------ | ------------ | -------------------------- |
| `POST` | `/checkout1` | Create checkout for Math 1 |
| `POST` | `/checkout2` | Create checkout for Math 2 |
| `GET`  | `/complete1` | Payment success (Math 1)   |
| `GET`  | `/complete2` | Payment success (Math 2)   |

### Protected Content

| Method | Endpoint        | Description         |
| ------ | --------------- | ------------------- |
| `GET`  | `/math1`        | Math course 1 page  |
| `GET`  | `/math2`        | Math course 2 page  |
| `GET`  | `/proxy/1video` | Stream Math 1 video |
| `GET`  | `/proxy/2video` | Stream Math 2 video |

---

## 📄 License

ISC

---

## 👤 Author

**Jurica Slibar** - [GitHub](https://github.com/Jura26)
