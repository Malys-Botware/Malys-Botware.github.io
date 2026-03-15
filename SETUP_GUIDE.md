# Supabase Setup Guide

Step-by-step instructions to get the dashboard and journal backend running.
Time: ~15 minutes.

---

## Step 1: Create a Supabase Account & Project

1. Go to [https://supabase.com](https://supabase.com) and sign up (free tier is fine)
2. Click **"New Project"**
3. Give it a name (e.g., `malys-site`)
4. Set a **database password** — save this somewhere safe
5. Choose a region close to you (e.g., West US or Canada)
6. Click **"Create new project"** and wait ~2 minutes

---

## Step 2: Get Your API Keys

1. In your Supabase dashboard, go to **Settings → API** (left sidebar)
2. You need two values:
   - **Project URL** — looks like `https://abcdefg.supabase.co`
   - **anon / public key** — the long `eyJhbGciOi...` string under "Project API keys"

3. Open your `index.html` and replace the placeholder values near the top of the `<script>` section:

```javascript
const SUPABASE_URL = 'https://your-actual-project-url.supabase.co';
const SUPABASE_ANON_KEY = 'eyJhbGciOi...your-actual-anon-key';
```

These are safe in client-side code. RLS (Row Level Security) enforces all access rules at the database level.

---

## Step 3: Create the Database Tables

1. In your Supabase dashboard, go to **SQL Editor** (left sidebar)
2. Click **"New Query"**
3. Paste the following SQL and click **"Run"**:

```sql
-- ============================================================
-- TABLE: dashboard_tiles
-- ============================================================
-- Dashboard tiles support published/draft like journal entries.
-- Only authenticated owner can create/edit/delete.

CREATE TABLE dashboard_tiles (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  title TEXT NOT NULL,
  content TEXT NOT NULL,
  published BOOLEAN DEFAULT false,
  sort_order INTEGER DEFAULT 0,
  user_id UUID REFERENCES auth.users(id),
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
);

ALTER TABLE dashboard_tiles ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Public can read published tiles"
  ON dashboard_tiles FOR SELECT
  USING (published = true);

CREATE POLICY "Owner can read all own tiles"
  ON dashboard_tiles FOR SELECT
  TO authenticated
  USING (auth.uid() = user_id);

CREATE POLICY "Owner can create tiles"
  ON dashboard_tiles FOR INSERT
  TO authenticated
  WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Owner can update own tiles"
  ON dashboard_tiles FOR UPDATE
  TO authenticated
  USING (auth.uid() = user_id)
  WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Owner can delete own tiles"
  ON dashboard_tiles FOR DELETE
  TO authenticated
  USING (auth.uid() = user_id);


-- ============================================================
-- TABLE: journal_entries
-- ============================================================

CREATE TABLE journal_entries (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  title TEXT NOT NULL,
  entry_date TEXT NOT NULL,
  content TEXT NOT NULL,
  published BOOLEAN DEFAULT false,
  user_id UUID REFERENCES auth.users(id),
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
);

ALTER TABLE journal_entries ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Public can read published entries"
  ON journal_entries FOR SELECT
  USING (published = true);

CREATE POLICY "Owner can read all own entries"
  ON journal_entries FOR SELECT
  TO authenticated
  USING (auth.uid() = user_id);

CREATE POLICY "Owner can create entries"
  ON journal_entries FOR INSERT
  TO authenticated
  WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Owner can update own entries"
  ON journal_entries FOR UPDATE
  TO authenticated
  USING (auth.uid() = user_id)
  WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Owner can delete own entries"
  ON journal_entries FOR DELETE
  TO authenticated
  USING (auth.uid() = user_id);
```

You should see "Success. No rows returned." — that's correct.

---

## Step 4: Create Your Admin Account

1. Go to **Authentication → Users** (left sidebar)
2. Click **"Add User" → "Create New User"**
3. Enter your email and a strong password
4. Check **"Auto Confirm User"**
5. Click **"Create User"**
6. **Copy your User ID** — it's the UUID shown in the users list (you need it in Step 6)

---

## Step 5: Disable Public Signups

1. Go to **Authentication → Providers** (left sidebar)
2. Under **Email**, find **"Enable Sign Ups"**
3. **Turn it OFF**
4. Click **Save**

---

## Step 6: Seed Your Existing Data

Run this in the SQL Editor. Content is stored as HTML (from the rich text editor).

**IMPORTANT:** Replace `YOUR_USER_ID_HERE` with the UUID you copied in Step 4.

```sql
-- ============================================================
-- SEED: Dashboard tiles (HTML content from rich text editor)
-- ============================================================

INSERT INTO dashboard_tiles (title, content, published, sort_order, user_id) VALUES
(
  'System',
  '<p>D.E.''s: Cosmic and i3 till im happy with the i3 config things to fix in i3:</p><p>- some apps not working properly or as expected</p><p>ex. - mission control opens to blank/translucent window</p><p>- garuda toolbox can''t run certain diags or sys update "Error creating textual authentication agent: error opening current controlling terminal for the process (''/dev/tty''): no such device or address". ?? works fine on cosmic though.</p><p>general: lockscreen needs theme still</p>',
  true, 1, 'YOUR_USER_ID_HERE'
),
(
  'Lessons',
  '<p>- Onward from ch3 at last! got to ch5 l2</p>',
  true, 2, 'YOUR_USER_ID_HERE'
),
(
  'This Site',
  '<p>- tweak "hero"</p>',
  true, 3, 'YOUR_USER_ID_HERE'
),
(
  'Keybindings Cheatsheet / Editor',
  '<p>undecided where i want to go with it rn, seems to be functioning as expected.</p><p>gonna try to learn how exactly it works before going too much farther.</p>',
  true, 4, 'YOUR_USER_ID_HERE'
);


-- ============================================================
-- SEED: Journal entries (HTML content)
-- ============================================================

INSERT INTO journal_entries (title, entry_date, content, published, user_id, created_at) VALUES
(
  'lesson day',
  'March 14, 2026',
  '<p>how many lessons can you get through? be sure to use potions if you have the gems! don''t forget about the other usefull tools at your disposal like embers, potions, quests and more!</p><p>talk to boots if needed! he can be rather helpful if you talk through your problem with him!</p><p>go back and review your previous couple lessons or maybe even chapters! especially if you had an extended session previously</p>',
  true, 'YOUR_USER_ID_HERE', '2026-03-14T12:00:00Z'
),
(
  'veg day',
  'March 13, 2026',
  '<p>just a day to sit back and breath. lay back, get comfy and poke around in the tools you''re learning or go browse some youtube / twitch / whatever tickles your fancy. or maybe you''ve been putting off a project around the house, gaming? nows the day</p><p>what i got up to today: explored github some more. all commits updating the site today done from github, without AI. just the keyboard and a little mouse action here and there thought about getting back into boot.dev but just need to let my brain settle. watched some youtube. answered a q wrong on x shoulda been pwd - print working directory shows absolute path ended up doing a couple/few lessons</p><p>fairly light easy day, weather coulda beem nicer, plans for tomorrow? hmm.</p>',
  true, 'YOUR_USER_ID_HERE', '2026-03-13T12:00:00Z'
),
(
  'the post that started this',
  'March 12, 2026',
  '<p>I wrote a long post on r/learnprogramming. Put real thought into it — where I''m at, what I''m struggling with, honest questions about balancing learning with building, time management, and how an introvert starts connecting with other people in tech. The mods removed it before anyone could even read it.</p><p>And you know what? No ill will. The sub has rules and I probably should''ve read the FAQ more carefully. But it did help me realize something: I need my own space. Somewhere my thoughts can exist without getting caught in someone else''s filter.</p><p>So here we are. This site is that space. It started as frustration but turned into something I''m actually excited about. Just writing that reddit post — even though nobody saw it — helped me organize my thoughts. Turns out writing things down is weirdly therapeutic. Who knew.</p><p>I''m about a month into learning to code. Started by installing linux on an old alienware laptop. Put my first repo on github yesterday — an i3 keybinding cheatsheet and editor. It''s small but it''s real and it''s mine. Every new thing I learn opens another door and then I''m down the next rabbithole. Can''t stop won''t stop... until I crash. Then repeat.</p><p>I know I need to slow down. I know I rely on AI too much. I know I should talk to actual humans more. I''m working on all of it. But at least now I have somewhere to write about the process while I figure it out.</p><p>Gonna keep on keepin'' on, I suppose.</p>',
  true, 'YOUR_USER_ID_HERE', '2026-03-12T12:00:00Z'
);
```

---

## Step 7: Test It

1. Push the updated `index.html` to your GitHub Pages repo
2. Visit your site — you should see your dashboard tiles and journal entries
3. Look for the small power icon next to "Malys" in the top nav bar — click it
4. Log in with the email/password from Step 4
5. You should see:
   - "ADMIN MODE" toolbar on both Dashboard and Journal
   - "+ NEW TILE" and "+ NEW ENTRY" buttons
   - A rich text editor with formatting toolbar (headers, bold, italic, lists, etc.)
   - Edit, delete, publish/unpublish controls on each tile and entry
   - Share controls on journal entries
6. Try creating a new journal entry with some formatting — it saves as a draft
7. Try creating a new dashboard tile — also saves as a draft
8. Publish both and verify they appear when logged out
9. Log out and verify drafts are hidden

---

## Troubleshooting

**"Supabase not configured yet" toast when clicking the power icon:**
You haven't replaced the placeholder URL and key in `index.html`. See Step 2.

**Login fails with "Email not confirmed":**
Go to Authentication → Users, click your user, and make sure they're confirmed.

**Entries/tiles don't load (empty sections):**
Check the browser console (F12 → Console) for errors. Most likely the RLS policies weren't created — re-run the SQL from Step 3.

**"new row violates row-level security policy":**
The `user_id` in your INSERT doesn't match your actual user UUID. Double-check the value from Step 4.

**Rich text editor not appearing in the modal:**
Check that the Quill CDN links are loading (no network errors in console). The editor should appear as a white toolbar above a dark text area.

---

## Architecture Summary

```
GitHub Pages (static hosting)
  └── index.html (your entire site)
        ├── Google Fonts (CDN) — typefaces
        ├── Supabase JS v2 (CDN, MIT license) — auth + database
        ├── Quill v2 (CDN, BSD-3-Clause license) — rich text editor
        ├── dashboard_tiles table (published/draft system)
        └── journal_entries table (published/draft system)
              └── PostgreSQL with Row Level Security
                    ├── Public: read published tiles + entries only
                    └── Your account: full CRUD on all content
```

No server code. No build step. Still one HTML file.
