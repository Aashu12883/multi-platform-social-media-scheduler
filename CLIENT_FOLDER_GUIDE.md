# Client folder guide

`client/` is the React single-page application (SPA) for the social-media scheduler. It is built with Vite, TypeScript, React Router, Tailwind CSS, Axios, and Lucide/simple-icons.

## Scope

This guide covers every project-authored file currently under `client/`. The `client/node_modules/` directory is intentionally not listed file-by-file: it is generated from `package-lock.json` when dependencies are installed and contains third-party package code. Its purpose is to provide the dependencies declared in `package.json`.

## Root files

| File | What it does |
| --- | --- |
| `client/.env` | Holds Vite environment variables. In this app, `VITE_API_BASE_URL` can set the backend API URL; its value is not included here because environment files may contain local or sensitive configuration. |
| `client/package.json` | Defines the client package, its scripts (`dev`, `build`, `lint`, and `preview`), and its runtime/development dependencies. |
| `client/package-lock.json` | Locks the exact dependency tree used by npm so installations are reproducible. |
| `client/index.html` | The browser HTML shell. It provides the `#root` element React mounts into, sets metadata/font links, and loads `src/main.tsx`. |
| `client/vite.config.ts` | Vite build-server configuration. Enables React, the React Compiler Babel preset, and the Tailwind Vite plugin. |
| `client/eslint.config.js` | Flat ESLint configuration for JavaScript/TypeScript, React hooks, and Vite refresh rules; ignores generated `dist/`. |
| `client/tsconfig.json` | TypeScript solution entry point that references the separate browser-app and Node/Vite configurations. |
| `client/tsconfig.app.json` | TypeScript settings for `src/`: modern JavaScript target, DOM types, JSX, Vite types, bundler resolution, and no output (Vite performs the build). |
| `client/tsconfig.node.json` | TypeScript settings for Node-side tooling, specifically `vite.config.ts`. |
| `client/README.md` | The default Vite React/TypeScript template documentation. It explains the starter tooling rather than this scheduler's features. |

## Static public files

| File | What it does |
| --- | --- |
| `client/public/logo.svg` | Scheduler logo used in the landing navigation, sign-in screen, sidebar, and footer. |
| `client/public/favicon.svg` | Browser-tab favicon. |
| `client/public/icons.svg` | SVG icon asset available as a static public resource; it is not imported by the current TypeScript/React source. |

## Application entry and shared infrastructure

| File | What it does |
| --- | --- |
| `client/src/main.tsx` | React bootstrap file. Mounts the app in `#root`, enables `StrictMode`, and wraps it in `BrowserRouter` and `AuthProvider`. |
| `client/src/App.tsx` | Defines all routes and installs the top-right toast container. Public routes are `/` and `/login`; authenticated pages are rendered inside `Layout`. |
| `client/src/index.css` | Imports Tailwind and Google fonts, defines design tokens and global body/button styles, smooth scrolling, and custom scrollbar styling. |
| `client/src/api/axios.ts` | Creates the shared Axios HTTP client. It uses `VITE_API_BASE_URL`, falling back to `http://localhost:3000`. |
| `client/src/context/AuthContext.tsx` | Provides authentication state and helpers. It restores the user/token from `localStorage`, attaches the bearer token to Axios after login or refresh, and removes it on logout. |

## Layout and reusable components

| File | What it does |
| --- | --- |
| `client/src/components/Layout.tsx` | Protected application shell. Shows a loading state, redirects unauthenticated visitors to `/login`, renders the responsive sidebar/top bar, and places the active route in an `Outlet`. |
| `client/src/components/Sidebar.tsx` | Desktop/mobile navigation for Dashboard, Accounts, Scheduler, and AI Composer. Displays the signed-in user's details and handles logout. |
| `client/src/components/AccountList.tsx` | Renders the connected-account cards, their status, an empty state, and a confirmation flow before disconnecting an account. |
| `client/src/components/PlatformPickerModal.tsx` | Modal for selecting a social platform to connect. It prevents connecting a platform twice and shows connection progress/status. |

## Pages

| File | What it does |
| --- | --- |
| `client/src/pages/Home.tsx` | Composes the public landing page from the navigation, hero, feature, process, testimonial, pricing, CTA, and footer sections. |
| `client/src/pages/Login.tsx` | Combined sign-in/sign-up form. Posts credentials to `/api/auth/login` or `/api/auth/register`, stores returned auth data through the context, displays errors as toasts, and navigates to the dashboard. |
| `client/src/pages/Dashboard.tsx` | Fetches posts, social accounts, and activity from `/api/posts`, `/api/accounts`, and `/api/activity`. It calculates headline counts and renders the recent-activity feed. |
| `client/src/pages/Accounts.tsx` | Manages connected social accounts. It loads/disconnects accounts, starts OAuth using `/api/oauth/:platform/url`, handles OAuth callback query parameters, and optionally synchronizes through `/api/oauth/sync`. |
| `client/src/pages/Scheduler.tsx` | Lets a user create a scheduled post: select platforms, write content, select date/time, optionally upload media, and submit multipart data to `/api/posts`. It reloads posts every 10 seconds and lists scheduled/published content. Instagram posts require media. |
| `client/src/pages/AIComposer.tsx` | AI content-generation workspace. It sends a prompt/tone to the generation API, shows generated content/history, supports copying/removing results, and can hand a selected result off for scheduling. |

## Landing-page sections

| File | What it does |
| --- | --- |
| `client/src/components/Home/Navbar.tsx` | Sticky landing-page navigation. Links to page sections and shows either sign-in/get-started links or a dashboard link based on auth state. |
| `client/src/components/Home/Hero.tsx` | Landing-page introduction, primary calls to action, and a static dashboard mockup. |
| `client/src/components/Home/Features.tsx` | Data-driven grid describing the scheduler's main features. |
| `client/src/components/Home/HowItWorks.tsx` | Three-step overview: connect accounts, create content, then schedule/publish. |
| `client/src/components/Home/Testimonials.tsx` | Static customer-testimonial cards. |
| `client/src/components/Home/Pricing.tsx` | Static Starter, Pro, and Agency pricing cards. |
| `client/src/components/Home/CTA.tsx` | Final landing-page conversion section with links to sign up and pricing. |
| `client/src/components/Home/Footer.tsx` | Brand/footer navigation and legal/sign-in links. Several product/company/legal links are currently placeholders (`#`). |

## Assets and shared data

| File | What it does |
| --- | --- |
| `client/src/assets/assets.tsx` | Central asset/data module. Defines supported platform metadata/icons, a custom LinkedIn SVG component, and mock posts/accounts/activity/AI-generation records used as sample data. |
| `client/src/assets/img-1.jpg` | Image asset referenced by sample post and AI-generation data. |
| `client/src/assets/img-2.jpg` | Image asset referenced by sample post and AI-generation data. |
| `client/src/assets/img-3.jpg` | Image asset referenced by sample post and AI-generation data. |
| `client/src/assets/img-4.jpg` | Image asset referenced by sample post and AI-generation data. |

## Route flow

`index.html` → `src/main.tsx` → `src/App.tsx` → public landing/login routes or the authenticated `Layout` → dashboard, accounts, scheduler, or AI-composer page.

