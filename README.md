# Multi-Platform Social Media Scheduler

A full-stack application for creating, scheduling, and publishing social-media posts from one dashboard. Users can connect supported social accounts, create posts manually or with AI assistance, attach media, pick a publication time, and review account activity.

## What it does

- Registers users and authenticates them with JWTs.
- Connects Twitter/X, LinkedIn, Facebook, and Instagram accounts through Zernio OAuth.
- Creates posts for one or more platforms, with optional image or video uploads.
- Uploads media to Cloudinary.
- Checks scheduled posts every minute and publishes due posts through Zernio.
- Records successful publications in an activity feed.
- Generates social-post copy with Google Gemini and, optionally, an image through Leonardo.ai.
- Saves users, accounts, posts, AI generations, and activity logs in MongoDB.

## Tech stack

| Area | Technology |
| --- | --- |
| Frontend | React 19, TypeScript, Vite, React Router, Tailwind CSS |
| UI | Lucide icons, Simple Icons, React Hot Toast |
| HTTP client | Axios |
| Backend | Node.js, Express 5, TypeScript |
| Database | MongoDB with Mongoose |
| Authentication | bcrypt password hashing and JSON Web Tokens |
| Social publishing | Zernio Node SDK |
| Scheduling | node-cron |
| Media storage | Cloudinary |
| AI text | Google Gemini (`gemini-2.5-flash`) |
| AI image generation | Leonardo.ai (optional) |

## Project structure

```text
multi-platform-social-media-scheduler/
├── client/                 # Vite + React browser application
│   ├── src/
│   │   ├── api/            # Shared Axios client
│   │   ├── assets/         # Platform metadata, sample data, and images
│   │   ├── components/     # Reusable UI and landing-page sections
│   │   ├── context/        # Authentication context
│   │   └── pages/          # Home, Login, Dashboard, Accounts, Scheduler, AI Composer
│   └── public/             # Static logo and favicon assets
├── server/                 # Express API and background scheduler
│   ├── config/             # MongoDB, Cloudinary, Multer, and Zernio configuration
│   ├── controllers/        # Request handlers and application logic
│   ├── middlewares/        # JWT protection middleware
│   ├── models/             # Mongoose schemas
│   ├── routes/             # API route definitions
│   └── services/           # Cron-based publisher
└── README.md               # This project guide
```

## Prerequisites

- Node.js 20 or later
- npm
- A MongoDB database (local or Atlas)
- A Zernio API key for social-account connection and publishing
- A Cloudinary account for media uploads
- A Google Gemini API key for AI copy generation
- A Leonardo.ai API key only when AI image generation is needed

## Environment configuration

Create `server/.env` and provide the values below. Never commit real credentials.

```env
PORT=3000
MONGODB_URI=your_mongodb_connection_string
JWT_SECRET=replace_with_a_long_random_secret

ZERNIO_API_KEY=your_zernio_api_key

CLOUDINARY_CLOUD_NAME=your_cloud_name
CLOUDINARY_API_KEY=your_cloudinary_api_key
CLOUDINARY_API_SECRET=your_cloudinary_api_secret

GEMINI_API_KEY=your_gemini_api_key
LEONARDO_API_KEY=your_leonardo_api_key
```

Create `client/.env` when the API is not running at the default local address:

```env
VITE_API_BASE_URL=http://localhost:3000
```

`VITE_*` variables are bundled into the browser application. Do not put server secrets in `client/.env`.

## Run locally

Install and start the server in one terminal:

```bash
cd server
npm install
npm run dev
```

Install and start the client in another terminal:

```bash
cd client
npm install
npm run dev
```

Open the local URL printed by Vite (normally `http://localhost:5173`). The API defaults to `http://localhost:3000` unless `VITE_API_BASE_URL` is set.

Useful commands:

| Location | Command | Purpose |
| --- | --- | --- |
| `client` | `npm run build` | Type-check and build the production frontend. |
| `client` | `npm run lint` | Run ESLint. |
| `client` | `npm run preview` | Serve the built frontend locally. |
| `server` | `npm run dev` | Run the API with Nodemon and `tsx`. |
| `server` | `npm run build` | Compile TypeScript to `server/dist/`. |
| `server` | `npm start` | Run the API with `tsx`. |

## User flow

1. Create an account or sign in on `/login`.
2. Connect a social platform from **Accounts**. The app redirects to a Zernio-managed OAuth flow, then syncs connected accounts into MongoDB.
3. Create a post in **Scheduler**, choose target platforms, optionally attach media, and choose a local date/time.
4. Or use **AI Composer** to generate copy, optionally generate an image, and schedule the saved generation.
5. The server checks every minute for posts with `scheduledFor <= now`.
6. Due posts are published through connected Zernio accounts. A successful post becomes `published` and creates an activity-log record; unsuccessful publication is marked `failed`.

## Application routes

| Frontend route | Access | Purpose |
| --- | --- | --- |
| `/` | Public | Marketing/landing page. |
| `/login` | Public | Sign-in and account-registration screen. |
| `/dashboard` | Authenticated | Post/account counts and recent publishing activity. |
| `/accounts` | Authenticated | Connect, synchronize, view, and disconnect social accounts. |
| `/schedule` | Authenticated | Create and schedule manual posts. |
| `/ai-composer` | Authenticated | Generate AI content and schedule a generation. |

The frontend keeps the returned user/token in `localStorage` and attaches the token as `Authorization: Bearer <token>` for protected API calls.

## API summary

All endpoints other than registration/login require a bearer token.

| Method | Endpoint | Purpose |
| --- | --- | --- |
| `GET` | `/` | Basic API health response. |
| `POST` | `/api/auth/register` | Create a user and return a JWT. |
| `POST` | `/api/auth/login` | Authenticate a user and return a JWT. |
| `GET` | `/api/accounts` | List the signed-in user's connected accounts. |
| `POST` | `/api/accounts` | Manually create an account record. |
| `DELETE` | `/api/accounts/:id` | Disconnect an account and delete it from Zernio when applicable. |
| `GET` | `/api/oauth/:platform/url` | Create a Zernio OAuth connection URL. |
| `GET` | `/api/oauth/sync` | Import/synchronize Zernio accounts into MongoDB. |
| `GET` | `/api/posts` | List the signed-in user's posts. |
| `POST` | `/api/posts` | Create a scheduled post; accepts optional multipart `media`. |
| `POST` | `/api/posts/generate` | Generate and save AI post content; optional image generation. |
| `GET` | `/api/posts/generations` | List the signed-in user's AI generations. |
| `GET` | `/api/activity` | Return the 10 most recent activity records. |

## Data model

| Model | Main fields | Role |
| --- | --- | --- |
| `User` | `name`, `email`, hashed `password`, `zernioProfileId` | Application user and Zernio workspace reference. |
| `Account` | `user`, `platform`, `handle`, `zernioAccountId`, `status` | A connected social account. |
| `Post` | `content`, `platforms`, `mediaUrl`, `scheduledFor`, `status` | A manually scheduled/published/failed post. |
| `Generation` | `prompt`, `content`, `tone`, `mediaUrl` | A saved AI-generated draft. |
| `ActivityLog` | `actionType`, `description`, `relatedPost` | A record of publishing activity. |

## Current implementation notes

- Supported platform IDs in the interface are `twitter`, `linkedin`, `facebook`, and `instagram`.
- Instagram scheduling is validated in the manual scheduler to require an image or video.
- The scheduler only publishes to connected accounts that have a Zernio account ID.
- Scheduling uses the server machine's current time. The browser converts the selected date/time to ISO before it sends the post.
- The landing page’s pricing, testimonials, and many footer links are presentational/static content.

## Security notes

- Use a long, unique `JWT_SECRET`; the fallback secret in the source is appropriate only for local experimentation and should be removed or replaced for deployment.
- Keep API keys solely in `server/.env` or your deployment platform’s secret store.
- Configure CORS restrictively before production; the current API enables CORS without an origin allowlist.
- Use HTTPS in production, especially because the client stores the JWT in `localStorage`.

