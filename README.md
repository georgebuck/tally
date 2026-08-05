# Hosting Tally yourself

The app is one HTML file with no build step. You need two things: somewhere to serve the file, and somewhere to keep the data. Total cost: nothing.

---

## Step 1 — Get your data out of Claude first

Do this **before** anything else, and do not unpublish the Claude artifact until you've confirmed the self-hosted copy works. Unpublishing permanently deletes the stored data and the artifact can't be republished.

1. Open the published artifact.
2. Tap your name (top right) → **Make a backup**.
3. Copy the text that appears into a note or a file.

Your partner should do the same on their own device — their private pots live under their account, not yours. Shared pots only need backing up once.

---

## Step 2 — Create the database

1. Sign up at [supabase.com](https://supabase.com) and create a new project (free tier is plenty). Pick a region near you.
2. Open the **SQL Editor** in the left sidebar and run this:

```sql
create table kv (
  key        text not null,
  scope      text not null,
  value      text not null,
  updated_at timestamptz not null default now(),
  primary key (key, scope)
);

alter table kv enable row level security;

create policy "app access" on kv
  for all to anon
  using (true) with check (true);

-- Newer projects need these explicitly; harmless on older ones.
grant usage on schema public to anon;
grant select, insert, update, delete on table kv to anon;
```

3. Collect two values:
   - **Project URL** — click **Connect** at the top of the project dashboard. It looks like `https://abcdefgh.supabase.co`.
   - **Key** — go to **Settings → API Keys**. Either take the `anon` key from the **Legacy API Keys** tab, or click **Create new API Keys** and take the `sb_publishable_...` value. Prefer the publishable one; the legacy keys are being retired at the end of 2026.

   Never use the `service_role` or `sb_secret_...` key. It bypasses every security rule and this file is public.

That's the whole backend. One table, key-value, nothing else.

---

## Step 3 — Point the app at it

Open `tally.html` in any text editor. Near the top of the `<script>` block you'll find:

```js
var SUPABASE_URL = '';   // e.g. 'https://abcdefgh.supabase.co'
var SUPABASE_KEY = '';   // your project's anon / publishable key
```

Paste your two values between the quotes. Save. Rename the file to **`index.html`**.

If you leave these blank the app still runs, but data stays in that one browser on that one device — fine for a solo pot, no good for a shared one.

---

## Step 4 — Put it online

Any static host works. Easiest first:

**GitHub Pages** is the pick if your personal site is going there too — one account, one workflow.

1. Create a repo named `tally`. Public (Pages needs a paid plan for private repos).
2. Put `index.html` at the top level, alongside an empty file called `.nojekyll`.
3. Settings → Pages → Source: **Deploy from a branch**, branch `main`, folder `/ (root)`.
4. Wait for the Actions tab to go green, then read the live URL printed at the top of Settings → Pages. For a repo called `tally` it'll be `https://username.github.io/tally/` — case-sensitive, trailing slash included.

To update later, commit a new `index.html`. The site rebuilds itself.

Alternatives if you'd rather not use GitHub: **Netlify Drop** ([app.netlify.com/drop](https://app.netlify.com/drop)) lets you drag a folder in and be live in ten seconds, and **Cloudflare Pages** does the same with unlimited bandwidth.

---

## Step 5 — Restore your data

Open your new URL, sign in with your name, tap your name → paste the backup into the box → **Restore from paste**.

Two notes on identity:

- **Shared pots** land in Supabase and are visible to anyone using your URL. Send that URL to your partner and they'll see the pot in "Shared pots you can join".
- **Private pots** are filed under a random ID generated per browser, shown on the You screen as your *private-data ID*. To see your private pots on a second device, copy that ID from the first device and paste it into the same box on the second. Without that, each browser gets its own private space.

---

## Step 6 — Put it on the home screen

**iPhone:** open the URL in Safari (it has to be Safari, not Chrome) → Share → *Add to Home Screen*. It launches full-screen with no browser chrome, like an app.

**Android:** Chrome → menu → *Add to Home screen*.

The icon is a small embedded graphic. If you want a proper polished icon, drop a 180×180 PNG called `apple-touch-icon.png` next to `index.html` and iOS will use it.

---

## What this is and isn't

**Yours now.** The data is in your Supabase project, exportable at any time, and the app is a file you control. No dependency on Claude, no subscription.

**Not secured by user accounts.** The anon key sits in the page source, and the policy above lets anyone holding it read and write the table. For two people at an unguessable URL that's a reasonable trade. It is not suitable for anything sensitive, and if you eventually open this up to a wider group of friends, that's the moment to add Supabase Auth and tighten the policy to per-user rows.

**No offline support yet.** It needs a connection to load and save. If you want it working on the Tube, that's a service worker — worth adding once you know you'll keep using it.

---

## If something breaks

- **Table exists but the app reads nothing.** Missing Data API grants — re-run the two `grant` lines from Step 2.

- **Nothing saves, no error.** Check the You screen — it names the backend in use. If it says "This device only", the two constants in Step 3 didn't get filled in.
- **Blank screen.** Open the browser console. A CORS or 401 error means the URL or key is wrong; a 404 on `/rest/v1/kv` means the table wasn't created.
- **Partner can't see the shared pot.** Confirm you're both on the exact same URL, and that they're not looking at a cached copy — a hard refresh usually settles it.
