# The Vault — Personal Card & TCG Collection Ledger

A single-page web app for tracking a sports card / TCG collection: inventory, sales,
profit/loss, and an auditable transaction ledger. No build step — just open `index.html`.

## What it does

- **Dashboard** — cost basis, market value, unrealized gain/loss, realized profit,
  ROI, and cash on hand
- **Inventory** — add/edit cards, grading (PSA/BGS/etc.), status (In Inventory,
  Listed for Sale, Personal Collection, Sold, Traded Away), and filters (status,
  category, grading, free-text search)
- **Transaction Ledger** — every purchase and sale is logged as its own timestamped,
  append-only entry (mistakes get marked *Voided* rather than deleted, to preserve
  the audit trail). Includes per-transaction profit/loss, filters, and a CSV export.

## Data storage

Data is stored in [Supabase](https://supabase.com) (free tier), not in this repo or
in the file itself. The app requires signing in with an email/password created in
the Supabase project's **Authentication → Users**.

Access is protected by:
- Supabase Auth (email/password login)
- Row Level Security policies scoped to `auth.uid()`, so only the logged-in user
  can read or write their own rows in the `kv_store` table

## Security note

This repo (and the deployed page) is public, since GitHub Pages on a free personal
account requires a public repository. That's fine here — the embedded Supabase key
is a **publishable** key, meant to be public. It cannot read or write any data on
its own; every request still has to pass the RLS policy above, which requires a
valid login. In other words: the code being visible does not expose the data.

## Setup (for reference)

1. Supabase project with a `kv_store` table:
   ```sql
   create table kv_store (
     key text not null,
     user_id uuid not null references auth.users(id),
     value jsonb not null,
     updated_at timestamptz default now(),
     primary key (key, user_id)
   );

   alter table kv_store enable row level security;

   grant select, insert, update, delete on kv_store to authenticated;

   create policy "Users manage only their own data"
   on kv_store
   for all
   using (auth.uid() = user_id)
   with check (auth.uid() = user_id);
   ```
2. A user created under Authentication → Users
3. `index.html` deployed via GitHub Pages (Settings → Pages → Deploy from branch)

## Backups

Use the **Export ledger (.csv)** button periodically as an independent backup of
transaction history, separate from what's stored in Supabase.
