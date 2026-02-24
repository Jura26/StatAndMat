# Stat&Mat

A full-stack web application for delivering mathematics and statistics video courses with user authentication, subscription payments, and protected content streaming.

## 🚀 Live Demo

Deployed on [Render](https://statandmat-mcht.onrender.com) as a Web Service.

---

## 📋 Table of Contents

- [Tech Stack](#-tech-stack)
- [Screenshots](#-screenshots)
- [Architecture](#-architecture)
- [Features](#-features)
- [Email Strategy](#-email-strategy)
- [Authentication Flow](#-authentication-flow)
- [Device Management](#-device-management)
- [Payment Integration](#-payment-integration)
- [Environment Variables](#-environment-variables)
- [Local Development](#-local-development)
- [Deployment](#-deployment)
- [API Endpoints](#-api-endpoints)

---

## 🛠 Tech Stack

### Backend

- **Node.js & Express.js**: The core of the application is built on Node.js, utilizing Express.js as the web framework. Express was chosen for its minimalist approach, robust routing capabilities, and extensive middleware ecosystem, which makes handling API requests, serving static files, and managing authentication flows straightforward.
- **MongoDB & Mongoose**: MongoDB serves as the primary NoSQL database. It was chosen because its document-oriented structure perfectly aligns with the application's need to store flexible, JSON-like user profiles, dynamic subscription statuses, and arrays of encrypted device identifiers. Mongoose is used as the Object Data Modeling (ODM) library to enforce schema validation, manage relationships, and simplify database queries, ensuring data integrity while maintaining MongoDB's inherent flexibility.

### Authentication & Security

- **Passport.js**: Implemented for robust authentication middleware, specifically utilizing the `passport-google-oauth20` strategy to allow users a seamless sign-in experience using their Google accounts.
- **JSON Web Tokens (JWT)**: Used for generating secure, time-limited tokens for email confirmation links during the registration process.
- **bcrypt**: Ensures user passwords are securely hashed and salted before being stored in the database, protecting against data breaches.
- **express-session & connect-mongo**: Manages user sessions persistently by storing session data in MongoDB, ensuring users remain logged in even if the server restarts.
- **AES Encryption**: The native Node.js `crypto` module is used to encrypt sensitive user data, such as IP addresses, before storing them in the database, adding an extra layer of privacy and security.

### Email Services

- **Nodemailer & SendGrid API**: A dual-transport email system is implemented. Nodemailer handles local development testing via SMTP, while the SendGrid API is used in production to reliably deliver transactional emails (like registration confirmations) and bypass common cloud provider SMTP port blocks.

### Payments

- **Stripe Checkout & Webhooks**: Stripe is integrated to handle secure payment processing for course subscriptions. Stripe Webhooks are utilized to asynchronously receive payment confirmations and automatically update the user's subscription status in the database without requiring manual intervention.

### Video Streaming

- **Bunny.net (Bunny Stream)**: Used as the primary video hosting and streaming service. Bunny Stream provides fast, globally distributed video delivery via its CDN, ensuring smooth playback for users regardless of their location. It also offers robust security features to protect the premium video content from unauthorized downloading or hotlinking.

### Frontend

- **EJS (Embedded JavaScript templating)**: Used for server-side rendering of dynamic content, allowing the application to inject user data and subscription statuses directly into the HTML before sending it to the client.
- **HTML/CSS/JavaScript**: Standard web technologies are used to build a responsive, mobile-friendly user interface that provides a smooth learning experience across all devices.

### DevOps & Deployment

- **Render**: The application is deployed as a Web Service on Render, providing continuous deployment from GitHub and automatic SSL certificate management.
- **MongoDB Atlas**: Hosts the production database in the cloud, ensuring high availability, automated backups, and secure access controls.

---

## 📸 Screenshots

<div align="center">
  <img src="assets/pictures/landing.JPG" alt="Landing Page" width="800" style="border-radius: 8px; box-shadow: 0 4px 8px rgba(0,0,0,0.1); margin-bottom: 20px;">
  <p><em>The main landing page showcasing available courses.</em></p>
</div>

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
                               │
                               ▼
                       ┌──────────────┐
                       │  Bunny.net   │
                       │   Stream     │
                       └──────────────┘
```

---

## ✨ Features

- **User Registration** with email confirmation (JWT-signed tokens)
- **Google OAuth 2.0** sign-in integration
- **Session Persistence** in MongoDB via connect-mongo
- **Stripe Checkout** for subscription payments
- **Protected Video Streaming** via Bunny.net with subscription validation
- **Device Limiting** - restrict account sharing (default: 2 devices per user)
- **Encrypted Device Tracking** - AES-encrypted IP addresses
- **Responsive UI** - works on desktop and mobile

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

<div align="center">
  <img src="assets/pictures/login.JPG" alt="Login Page" width="600" style="border-radius: 8px; box-shadow: 0 4px 8px rgba(0,0,0,0.1); margin-bottom: 20px;">
  <p><em>Secure login portal supporting both email/password and Google OAuth.</em></p>
</div>

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

## 📱 Device Management

To prevent account sharing and protect the premium video content, the application enforces a strict **2-device limit** per user account. This ensures that subscriptions are not shared among multiple users while still allowing a single user to access their content from, for example, a laptop and a mobile phone.

### The Logic Behind It

1. **Device Identification**: When a user attempts to log in, the server captures their IP address and `User-Agent` string to create a unique device identifier.
2. **Privacy & Encryption**: For privacy and security, the IP address is encrypted using AES-256 encryption (via Node.js `crypto` module) before being stored in the MongoDB database. This ensures that even if the database is compromised, the raw IP addresses remain secure.
3. **Device Tracking**: The encrypted device identifier is added to the user's `devices` array in the database.
4. **Limit Enforcement**: During the login process, the system checks the user's `devices` array. If the device is new and the user already has 2 devices registered, the login attempt is rejected with an error message.
5. **Session Management**: If the login is successful, the session is established and stored in MongoDB via `connect-mongo`, allowing the user to remain logged in on that specific device.

---

## 💳 Payment Integration

<div align="center">
  <img src="assets/pictures/math-first.JPG" alt="Math First Exam Course" width="800" style="border-radius: 8px; box-shadow: 0 4px 8px rgba(0,0,0,0.1); margin-bottom: 20px;">
  <p><em>Course overview page where users can initiate the Stripe checkout process.</em></p>
</div>

### Stripe Checkout Flow

```
1. User clicks "Subscribe" button
2. Server creates Stripe Checkout Session
3. User redirected to Stripe's hosted payment page
4. After payment, redirected to success URL
5. Server verifies session and grants access
```

### Subscription Types

- **Mathematics - First Exam** (`/checkout1`)
- **Mathematics - Second Exam** (`/checkout2`)

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
   - **Build Command**: `npm install`
   - **Start Command**: `npm start`
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
