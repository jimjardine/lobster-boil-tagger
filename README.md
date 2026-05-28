# Photo Name Tagger

A simple shareable web page for identifying people in a group photo. You click a numbered
marker on someone's head, type their name + email, and it saves to a shared database so
anyone with the link can help fill in names. Includes a printable list at the end.

This repo is the **Lobster Boil Party** photo. To tag a *different* photo, make a copy of
this whole setup (see "Making a new one" below).

---

## What's here

- **`index.html`** — the entire app (all the page + logic in one file). No build step.
- **`photo.jpg`** — the group photo the markers sit on top of.
- **`README.md`** — this file.

## How the live page works

- **Live site:** https://jimjardine.github.io/lobster-boil-tagger/
- Click a numbered marker → a box pops up to enter that person's name + email → Save.
  Saves are shared and live: anyone else with the link sees them appear.
- **Toggle numbers** button hides/shows the markers (clean photo vs. numbered).
- **Print** button → photo + the full numbered name/email list (also "Save as PDF").
- **Setup mode:** add `?setup=1` to the URL
  (https://jimjardine.github.io/lobster-boil-tagger/?setup=1) to **drag markers**, add a
  missed person (click an empty spot), or remove one. Share the **plain** URL (no
  `?setup=1`) with everyone else so they can't move things around.

## Where the data lives

- **Supabase project:** `lobster-boil-tagger`
  (dashboard: https://supabase.com/dashboard/project/zmujamxgxrgxaqjxyxvt)
- **Table:** `lobster_boil_people` — columns: `number`, `name`, `email`, `x`, `y`,
  `updated_at`. `x`/`y` are percent positions (0–100) of each marker over the photo.
- This is its **own** Supabase project, deliberately separate from the distillery /
  Inventory Tracker project, so the page's public key can't touch any business data.
  Keep future photo tables in this same project (it's free and dedicated to this).

The page uses Supabase's **publishable (public) key** — safe to have in the code. It only
allows reading/adding/editing rows in this one table.

---

## Making a new one (for a different photo)

Each photo = a copy of the folder + its own table. Steps:

1. **Copy the folder** to a new name, e.g. `christmas-party-tagger/`, and drop the new
   photo in as `photo.jpg`. If it's a big photo, shrink it to ~2000px wide first:
   ```
   sips --resampleWidth 2000 ORIGINAL.png --out photo.jpg
   ```

2. **Make a new table** in the same Supabase project (SQL editor:
   https://supabase.com/dashboard/project/zmujamxgxrgxaqjxyxvt/sql). Use a new table name,
   e.g. `christmas_party_people`:
   ```sql
   create table public.christmas_party_people (
     number int primary key, name text default '', email text default '',
     x numeric not null, y numeric not null, updated_at timestamptz default now()
   );
   alter table public.christmas_party_people enable row level security;
   create policy "anon read"   on public.christmas_party_people for select to anon using (true);
   create policy "anon insert" on public.christmas_party_people for insert to anon with check (true);
   create policy "anon update" on public.christmas_party_people for update to anon using (true) with check (true);
   alter publication supabase_realtime add table public.christmas_party_people;
   ```

3. **Point the new page at the new table.** In the copied `index.html`, change one line
   near the top (the URL and key stay the same — same project):
   ```js
   const TABLE = "christmas_party_people";
   ```

4. **Seed the markers.** Open the page with `?setup=1`, click each person to drop a
   numbered marker, then drag to fine-tune. (Or ask Claude to pre-place them from the photo.)

5. **Publish** as a new GitHub Pages site (new repo), or ask Claude to do the deploy.

> Easiest path: just tell Claude "set up a tagger for this new photo" and point at the
> image — Claude can do all of the above (new table, new page, deploy).

---

## Notes

- Anyone with the link can view and edit — that's intentional for crowd-sourcing names.
- To deploy changes to the live Lobster Boil page: commit and `git push`; GitHub Pages
  rebuilds in a minute or two (hard-refresh to clear the cache).
