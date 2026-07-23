# Dean Remake

## Overview

This repository contains a university Dean portal remake built as a React + Vite frontend with an Express + MySQL backend.

It provides:

- Student-facing pages for appointments, complaints, chat, and dashboard access
- Admin-facing pages for managing appointments, complaints, hostels, suggestions, and visitor bookings
- A backend API with JWT authentication, file uploads, notification workflows, and AI chat support
- A development workflow that can run frontend and backend together

## What the project does

Students can:

- Log in and access their personal dashboard
- Submit appointment requests and see status updates
- File complaints and view admin responses
- Chat with the Dean/administration using an AI-assisted conversation flow

Admins can:

- Log in to an admin dashboard
- Review and update appointment requests
- Manage complaints, hostel reports, suggestions, and visitor bookings
- Send messages and manage notifications

## Architecture

The code is organized under the `dean/` folder:

- `dean/src/` → React + Vite frontend
- `dean/backend/` → Express API server and database logic

## Key technologies

- Frontend: React, Vite, Tailwind CSS, React Router
- Backend: Node.js, Express, MySQL (`mysql2`), JWT authentication
- AI chat: `cohere-ai`
- Email: Brevo (Sendinblue) API for OTP verification
- Development tools: ESLint, Prettier, `concurrently`

## OTP Email Verification System

The backend includes a complete OTP (One-Time Password) email verification system:

### Features

- **Secure OTP Generation**: 6-digit random OTP with 5-minute expiration
- **Email Delivery**: Uses Brevo API for transactional emails
- **Database Storage**: Hashed OTPs stored in MySQL with attempt tracking
- **Rate Limiting**: Basic protection against spam requests (1 per minute per email)
- **Attempt Limits**: Maximum 5 verification attempts per OTP
- **Auto Cleanup**: Expired OTPs are automatically removed after verification

### API Endpoints

- `POST /auth/request-otp` - Request OTP for an email address
- `POST /auth/verify-otp` - Verify OTP code
- `POST /auth/student/forgot-password` - Request OTP for password reset (requires valid student email)
- `POST /auth/student/reset-password` - Reset password using OTP verification

### Password Reset Flow

1. **Request OTP**: `POST /api/auth/student/forgot-password`
   ```json
   {
     "email": "student@university.edu"
   }
   ```

2. **Reset Password**: `POST /api/auth/student/reset-password`
   ```json
   {
     "email": "student@university.edu",
     "otp": "123456",
     "newPassword": "newpassword123"
   }
   ```

### Setup Requirements

Add to your `.env` file:
```
BREVO_API_KEY=your_brevo_api_key_here
FROM_EMAIL=noreply@deanoffice.com
```

### Database Table

The system creates an `email_otps` table automatically via migration:
```sql
CREATE TABLE email_otps (
  id INT AUTO_INCREMENT PRIMARY KEY,
  email VARCHAR(255) NOT NULL,
  otp_hash VARCHAR(255) NOT NULL,
  expires_at DATETIME NOT NULL,
  attempts INT DEFAULT 0,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  INDEX idx_email (email),
  INDEX idx_expires_at (expires_at)
);
```

## Getting started

### Prerequisites

- Node.js v18+ (recommended)
- npm
- MySQL database server

### Setup

1. Open a terminal in the `dean/` folder.
2. Install dependencies:

```bash
cd dean
npm install
```

3. Create or update the backend `.env` file in `dean/backend/` with the required values.

Required variables:

- `DB_HOST`
- `DB_USER`
- `DB_PASS`
- `DB_NAME`
- `JWT_SECRET`
- `COHERE_API_KEY`
- `ADMIN_USERNAME`
- `ADMIN_PASSWORD`
- `FRONTEND_URL`

Optional backend variable:

- `PORT` (defaults to `5000`)

4. Run the backend server:

```bash
npm run dev:backend
```

5. Run the frontend app:

```bash
npm run dev
```

6. To launch both frontend and backend together:

```bash
npm run dev:all
```

### Environment helper

The repository includes `dean/scripts/set-env.js`, which updates the frontend `.env` and appends the frontend origin to `dean/backend/.env` when `npm run dev` or `npm run dev:all` starts.

## API routes

The backend exposes routes under `/api` for:

- `/api/auth`
- `/api/appointments`
- `/api/complaints`
- `/api/suggestions`
- `/api/notifications`
- `/api/chat`
- `/api/hostels`
- `/api/visitor`
- `/api/health`

## Project structure

- `dean/src/`
  - `pages/` → student, admin, hostel, and visitor pages
  - `components/` → reusable UI components
  - `context/` → authentication context
  - `api/` → Axios API client
- `dean/backend/`
  - `controllers/` → request handlers and business logic
  - `routes/` → route definitions
  - `models/` → database models and connection
  - `middleware/` → auth and request middleware
  - `migrations/` → schema and data migrations

## Available scripts

From the `dean/` folder:

- `npm run dev` → start the frontend locally
- `npm run dev:backend` → start the backend server locally
- `npm run dev:all` → start both frontend and backend together
- `npm run build` → build the frontend for production
- `npm run lint` → run ESLint checks
- `npm run preview` → preview the built frontend

## Notes

- The frontend uses Vite and can proxy API requests to the backend.
- The backend listens on `0.0.0.0` and defaults to port `5000`.
- Uploaded files are served from `dean/backend/uploads`.
- The project uses role-based authentication for students and admins.

## Recommended next steps

- Secure environment variables for production deployments
- Add database seed scripts for admin user creation
- Implement password recovery and user registration flows
- Customize AI prompts and messaging for your institution
