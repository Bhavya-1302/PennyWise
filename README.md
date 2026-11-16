# Pennywise

Pennywise is a premium, student-first personal finance dashboard built with React, TypeScript, Clerk, Supabase, and Vite. It is designed to be polished enough for a college presentation while remaining production-aware and Netlify-friendly.

## Features

- Clerk authentication with sign-in/sign-up flows and protected routes
- Supabase data persistence for profiles, preferences, transactions, budgets, goals, recurring items, debts, wishlist entries, and challenges (after configuring Clerk third-party auth and applying the schema)
- Dashboard analytics with live totals and charts
- Income and expense tracking with categories
- Budget monitoring and goal tracking
- Savings, recurring items, borrowed/lent records, wishlist, and no-spend challenge management
- Responsive UI with light/dark theme support
- CSV export for transactions
- Netlify-ready build and routing configuration

## Tech stack

- React
- TypeScript
- Vite
- Tailwind CSS
- Lucide React
- Recharts
- Clerk
- Supabase
- Netlify

## Local setup

1. Install dependencies:
   npm install
2. Copy `.env.example` to `.env` and replace placeholder values.
3. Start the app:
   npm run dev
4. Open the local Vite URL shown in the terminal.

## Required environment variables

Create a local .env file (do not commit it) with:

```bash
VITE_CLERK_PUBLISHABLE_KEY=pk_test_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
VITE_SUPABASE_URL=https://your-project.supabase.co
VITE_SUPABASE_PUBLISHABLE_KEY=sb_publishable_xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

The root .env.example contains placeholders only. Add these public variables to Netlify's environment settings for deployments. Never use a Supabase service-role key in the browser.

## Clerk setup

1. Create or select your Clerk application.
2. Add its **publishable key** to the root `.env` as `VITE_CLERK_PUBLISHABLE_KEY`. This key is intended for frontend use; never add a Clerk secret key to Vite variables.
3. Enable the sign-in methods you want in Clerk Dashboard. Clerk's prebuilt sign-in component provides account recovery when password authentication is enabled. Google sign-in is available when Google OAuth is enabled for the Clerk application.
4. The Vite entry point wraps Pennywise with the current `@clerk/react` `ClerkProvider`. The application shows sign-in/sign-up while signed out and only renders the finance workspace after Clerk reports a signed-in session. Clerk's `UserButton` provides profile/account management and sign-out.
5. For production, configure the site domain and allowed redirect URLs in Clerk Dashboard.

Pennywise authentication does not require Supabase. If Supabase is not configured, Clerk sign-in remains available and the signed-in user sees a setup notice instead of a failing finance-data view.

## Supabase setup

1. Confirm Clerk is configured as a Supabase third-party authentication provider for this Supabase project.
2. Add your Supabase project URL and publishable key to the root .env using the variable names above.
3. Run all SQL in ./supabase/schema.sql in the Supabase SQL Editor. Until that succeeds, this repository's database schema is not applied.
4. Verify each table has row-level security enabled. The policies check that `user_id` equals `auth.jwt()->>'sub'`, which is the Clerk user ID in the third-party JWT.
5. The frontend obtains the current token from Clerk's `useSession()` session resource for every Supabase request through the `accessToken` callback. The token must contain a Clerk subject and be accepted by the configured Supabase third-party auth provider.
6. Pennywise leaves `user_id` out of inserts; the database default derives it from the Clerk JWT, while RLS policies enforce ownership on reads and writes.

The app uses one `transactions` table with a checked `type` value (`income` or `expense`) to represent both income and expense entries. Other features use dedicated tables. Budgets are loaded for the current month and store its first day in `period_start`.

The dashboard balance is transaction income minus transaction expenses, plus unsettled borrowed amounts, minus unsettled lent amounts. Borrowed and lent records do not count as income or expenses, and settled records no longer adjust the balance. Avoid recording the same debt principal as a transaction as well, or it may be counted twice.

The browser stores the selected colour palette locally. Theme and currency are also cached locally for startup, then loaded from and saved to the signed-in user's `user_preferences` row. The current app does not persist financial records in LocalStorage, so there is no existing local financial dataset to import or overwrite.

## Netlify deployment

1. Push the project to GitHub.
2. In Netlify, create a new site from the repository.
3. Use the build command: `npm run build`
4. Use the publish directory: `dist`
5. Add the environment variables from `.env.example` in the Netlify dashboard.
6. Deploy.

The repository also includes `netlify.toml` to provide a SPA redirect for client-side routes.

## Database notes

After you configure Supabase and execute the schema, the application uses Supabase as the source of truth for user-owned financial records. The schema in `supabase/schema.sql` defines tables for:

- profiles
- transactions
- budgets
- savings goals
- recurring transactions
- borrowed/lent records
- wishlist entries
- no-spend challenges
- user preferences

The SQL defines RLS policies for each table. These protections are not active until you execute the SQL in your Supabase project and verify its Clerk third-party JWT configuration.
Profiles are refreshed from Clerk, and appearance/currency preferences are read from and saved to `user_preferences`.

## Project structure

- `src/App.tsx` — main app shell, routes, UI, and state
- `src/lib/supabase.ts` — reusable Supabase client using the current Clerk session token
- `src/types/finance.ts` — shared data types
- `src/App.css` — styling
- `supabase/schema.sql` — database schema and RLS policy setup
- `netlify.toml` — Netlify build and redirect config
- `.env.example` — required public environment variables

## Production notes

- Do not commit `.env` files.
- Do not expose service role keys in the browser.
- Only public variables should be shipped to the frontend.
- Clerk and Supabase credentials must be configured in the hosting dashboard before production usage.
