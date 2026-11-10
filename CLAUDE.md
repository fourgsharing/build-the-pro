# Build The Pro — Growth Tracker

Web app for the BUILD THE PRO 90-day mentorship program. Mentees fill in a self-assessment (35 skill questions, 15 drive questions, reflection sections) at 4 checkpoints; mentors and leadership review results. Questionnaire text is in Indonesian and was transcribed from the client's workbook. **Don't change question wording without checking with the client.**

## Stack and layout

- **One file: `index.html`.** HTML, CSS and JS are all inline. No build step, no package.json, no framework. Open it in a browser to run it.
- **Firebase (compat SDK v10.12.2, loaded from the gstatic CDN):** Auth (email/password) and Cloud Firestore. Firebase project ID: `build-the-pro`. The config is in `index.html` (search `firebaseConfig`). The web API key is public by design; security comes from the Firestore rules.
- **Hosting is on Vercel**, connected to GitHub `fourgsharing/build-the-pro`:
  - `master` deploys to production at https://build-the-pro.vercel.app
  - `testing` deploys to a preview at https://build-the-pro-git-testing-fourgsharing.vercel.app
- Other files: `firestore.rules` (the security rules, which must match what's live), `logo.png`, `commitment.png` (signature/commitment template).

## Roles and access

| Who | How they get in | What they see |
|---|---|---|
| Leadership | Signs in (email/username + password) | Leadership tab plus Mentor tab; manages people, edits detail views, Download Recap |
| Mentor | Signs in | Mentor tab only, limited to their own mentees |
| Mentee | Personal link, no sign-in: `?mentee=<menteeId>&round=<before\|mid\|after\|final>` | Only their own form for that round |

- **Usernames become emails.** `toEmail()` turns input without an `@` into `<name, lowercased, no spaces>@buildthepro.app`.
- **Role lookup.** After sign-in the app reads `users/{uid}` and uses its `role` field (`leadership` or `mentor`, plus `mentorId` for mentors). If that doc is missing, the app shows "Account setup incomplete" and stops.
- **First admin.** The "Create Admin Account" setup form only appears if the `users` collection is empty. Because of the rules, signed-out visitors can't read `users`, so in practice the first admin must be created by hand: create the Auth user in the Firebase console, then add a `users/<uid>` document with `role: "leadership"`.
- **Adding a mentor** (`addMentor`) also creates their login through a secondary Firebase app, with username = lowercased name without spaces and **default password `12345678`**. This is a known weakness: mentors should change it, or it should be replaced with something better.

## Firestore data model

- `users/{uid}`: `{ fullName, role, mentorId?, createdAt }`
- `mentors/{mentorId}`: `{ name, order?, createdAt }`. IDs look like `mentor-<slug>`.
- `mentees/{menteeId}`: `{ name, mentorId, order? }`
- `responses/{id}`: `{ menteeId, round, skill, drive, reflections, submittedAt }`. Saved through `scheduleAutoSave()`.
- `invites/{code}`: used during signup.

Loading: `loadAllFromFirestore()` (signed-in users) and `loadMenteeDataFromFirestore(menteeId)` (mentee links). The in-memory shapes are `MENTORS`, `MENTEES` and `DB.responses[menteeId][round]`.

Rounds (`ROUNDS`): `before`, `mid`, `after`, `final`.

## Firestore security rules (important history)

- **Incident on 9 Nov 2026:** nobody could get past sign-in, and the app showed "Error loading data: Missing or insufficient permissions." The cause was that the live rules were still Firebase's 30-day *test-mode* rules (`allow read, write: if request.time < timestamp.date(2026, 10, 1)`), which expired on 1 Oct 2026 and from then on denied every request. Sign-in itself kept working.
- **Fix:** published the rules now in `firestore.rules`. They have no expiry date. They differ from the earlier repo version in two ways:
  1. `mentors` is readable without sign-in, so mentee links can load their mentor's name.
  2. Leadership can create `users/{uid}` docs, so `addMentor` can create the new mentor's user doc.
- **Keep `firestore.rules` in the repo identical to the live rules** (Firebase console → Firestore → Rules). If you change one, change the other. There's no `firebase.json` yet, so to use `firebase deploy --only firestore:rules` you'd first run `firebase init firestore`. Otherwise paste the rules into the console.
- Rules use the `isLeadership()` helper, which reads `users/{auth.uid}.role`.
- **Known open items, to tighten before wider use:** anyone can create or update `responses` and update `invites` without signing in (needed for mentee links, but it lets anyone write). Mentee docs can be modified by any signed-in user, including mentors who don't own them.

## Admin / accounts

- The Firebase project is owned by a separate Google account (the second signed-in account in the owner's Chrome, `/u/1/` in console URLs), not the primary one. Firebase requires 2-Step Verification on it.
- Firebase plan: Spark (free). Daily limits are about 50k reads and 20k writes, far above current usage.

## Conventions

- Keep everything in `index.html`. Don't add a build system or framework unless asked.
- Commit messages have been short and imperative ("Fix …", "Add …", "Allow …").
- Normal flow: work on `testing`, check the preview URL, then merge to `master` through a PR.
- After any change that touches Firestore reads or writes, check it against `firestore.rules` for all three roles (leadership, mentor, signed-out mentee link).
- Don't commit real passwords or personal data.
