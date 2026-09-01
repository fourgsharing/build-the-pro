# Build The Pro — Growth Tracker

A prototype web app for the **BUILD THE PRO** 90-day mentorship program. Mentees fill
out a self-assessment questionnaire at 4 checkpoints (Before / Mid / After / Final);
mentors and program leadership can see the results without editing them.

## What this is right now

**A working prototype, not the finished product.** It's a single self-contained
`index.html` file — open it in any browser and it runs, no build step, no server.

- All three roles (Mentee / Mentor / Leadership) live in one page. Since there's no
  real backend yet, the top-right tabs let you preview all three views in one
  browser — in the real version each person would get their own personal link that
  opens straight into just their own view.
- All data (submitted answers, added mentors/mentees, edits) is saved in the
  browser's **local storage** — private to whichever browser/device it's opened in.
  Nothing syncs between people or devices at this stage. Opening the file fresh
  always starts from the same small set of hardcoded demo people (Sari, Bagus,
  Melati, Intan, Yoga under mentors Rania and Dimas).
- The questionnaire content (35 skill questions, 20 drive questions, plus several
  open-ended reflection sections) is transcribed from the client's own workbook and
  has gone through several rounds of wording edits — check with the client before
  changing question text.

## What's NOT built yet (the real next step)

Nothing about the design, the questions, or the three-role structure needs to
change. What's missing is the real plumbing underneath — this prototype's job was
to settle the product decisions first, before spending engineering time on that.
Roughly in build order:

1. **A real database** (Supabase is the natural pick — free tier, Postgres). Two
   tables cover it: a *people* table (name, role, `mentor_id` if they're a mentee,
   a unique token) and a *responses* table (person, round, skill/drive scores,
   reflection answers). This replaces the `DB` object and every `localStorage` call
   in `index.html`.
2. **Real personal links ("magic links")** — each person's row gets a random,
   unguessable token. Opening `yoursite.com/link/<token>` looks up that token and
   decides what to render: a mentee's own form, a mentor's filtered dashboard, or
   leadership's full view. No passwords, no accounts.
3. **Server-side permission enforcement** — right now "a mentor only sees their own
   mentees" is just JavaScript filtering in the browser; someone with devtools open
   could technically see everyone's data. The real version needs that enforced
   where the data lives (e.g. Supabase Row Level Security), not just hidden in the UI.
4. **Rewire the frontend to call the database instead of localStorage.** This is
   the mechanical part, not a redesign — every place the code currently does
   `getResponse(...)`, `saveDB()`, `addMentor(...)`, `addMentee(...)` etc. becomes a
   network call to Supabase instead. The data shapes already match what's needed.
5. **Hosting** (Vercel or similar) so it's a permanent public URL instead of a file
   someone has to open locally.
6. **Link delivery** — when leadership or a mentor adds someone, the system needs to
   hand that person their link somehow. Simplest first version: just show it
   on-screen to copy and send manually (WhatsApp, email, whatever) — no need to
   build automated email sending to ship a working v1.

## Project structure

Just one file: `index.html` — HTML, CSS, and JS all inline, no dependencies, no
build tooling. Open it directly in a browser to run it.

## Recent history

See `git log` for the detailed change history. At a glance: started as a prototype
covering the skill/drive questionnaire and 3-role dashboards, then added a 4th
"Final" checkpoint, per-round locking after submit, required-field validation, the
ability to add/rename/remove mentors and mentees, and a series of content/wording
fixes to match the client's exact phrasing.
