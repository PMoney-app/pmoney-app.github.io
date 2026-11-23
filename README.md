# ບໍລິຫານເງິນ (money-app)

Personal income/expense tracker — mobile-first PWA.

- **Frontend:** plain HTML/CSS/JS (no build step), Chart.js, supabase-js via CDN
- **Database/Auth:** Supabase project `money-app` (`eieeessmqfavwtmzzaoc`, Singapore)
- **Hosting:** Vercel (static)

## Files
| File | Purpose |
|---|---|
| `index.html` | All screens: login, home, transactions, plan, dashboard, sheets |
| `app.js` | App logic (phone + password login, profile, pages) |
| `i18n.js` | Lao / English text |
| `config.js` | Supabase URL + publishable key (safe to be public — RLS protects data) |
| `style.css` | Styles |
| `sw.js`, `manifest.json`, `icon*` | PWA (install to home screen) |

## Database
- `profiles` (name, avatar, language), `categories`, `transactions`, `budgets` — all with RLS `user_id = auth.uid()`
- Trigger seeds default Lao categories on sign-up
- RPCs: `get_balance()`, `get_monthly_summary(from,to)`, `get_category_summary(from,to)`
- Amounts are `bigint` kip (no decimals)

## Run locally
Any static server, e.g. `npx serve .` then open http://localhost:3000
- Storage bucket `avatars` (public read, each user writes only to `<uid>/`)
- Phone sign-up is free (no SMS): edge function `phone-signup` creates a confirmed account with login email `<856number>@phone.money-app.la`; the app logs in with phone + password
- Edge function `notify-ingest`: receives bank-app notifications (iPhone Shortcuts / Android MacroDroid) at `/functions/v1/notify-ingest?token=<profiles.ingest_token>` with JSON {title, body, app}; parses amount, income/expense, category and saves a transaction (source code in supabase/functions/notify-ingest)
- Android app (WebView + notification listener): source in android/ (separate zip). APK: https://eieeessmqfavwtmzzaoc.supabase.co/storage/v1/object/public/downloads/money-app.apk. The page passes the user's ingest token to the app via window.MoneyApp.setToken()
