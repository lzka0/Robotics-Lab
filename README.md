# GDG On Campus Zagazig · Robotics Portal (Supabase version)

Students and instructors each get **Software**, **Hardware** and **Mechanical** tabs (tasks with links, announcements, feedback) plus a **Sessions** tab for lecture links. Data lives in Supabase (free tier), and changes appear live without refreshing.

```
index.html           the whole website
assets/              gdg-logo.png, robotics-logo.png (replace with your real logos)
supabase/schema.sql  tables, security rules and live updates
```

## 0. Try it first (demo mode)
Open `index.html` with the Supabase settings empty. Demo logins: `sara` / `inst123` (instructor), `ahmed` / `stud123` (student). Data stays in your browser only.

## 1. Create the Supabase project
1. Sign up at supabase.com, then **New project**. Pick a region close to your users (for Egypt, an EU region is usually closest).
2. Open **SQL Editor → New query**, paste all of `supabase/schema.sql`, and click **Run**. It should finish without errors.

## 2. Lock down sign-ups (important)
In **Authentication** settings:
- Turn **off** "Allow new users to sign up". Otherwise anyone with your public key could create an account.
- Turn **off** "Confirm email", because the portal's users do not have real mailboxes.

## 3. Create users
The portal logs in with a username. Behind the scenes the username becomes the email `username@gdg-portal.app` (set by `EMAIL_DOMAIN` in `index.html`).
1. **Authentication → Users → Add user → Create new user**.
2. Email: `sara@gdg-portal.app` (lowercase username), set a strong password, tick **Auto Confirm User**, and create it.
3. Every new account starts as a **student**. To set the name and role, run this in the SQL Editor:
   ```sql
   update public.profiles set name = 'Eng. Sara Hassan', role = 'instructor' where username = 'sara';
   ```
   For students, only the name needs changing: `update public.profiles set name = 'Ahmed Samir' where username = 'ahmed';`
4. Create your own instructor account first and test the login before adding anyone else.

If Supabase rejects the email domain, use another made-up domain, and change `EMAIL_DOMAIN` in `index.html` to match. All users must use the same one.

## 4. Connect the website
In **Project Settings → API** copy the **Project URL** and the **anon / publishable key** into the top of the script in `index.html`:
```js
const CONFIG={SUPABASE_URL:'https://xxxx.supabase.co',SUPABASE_ANON_KEY:'eyJ...',EMAIL_DOMAIN:'gdg-portal.app',POLL_MS:30000};
```
**Never** paste the `service_role` key anywhere in the website. It bypasses all security.

## 5. Publish
Upload `index.html` and `assets/` to a GitHub repo, then Settings → Pages → Deploy from branch → `main` → `/ (root)`.

## Security model
- The anon key is meant to be public. Safety comes from **row level security** in `schema.sql`:
  - Only logged-in users can read anything. Anonymous visitors get nothing.
  - Only instructors can create or delete tasks, announcements, sessions and feedback.
  - Students can read their own feedback and progress and change only their own progress.
  - Names shown as authors are set by the database, not by the browser.
  - Roles can only be changed from the Supabase dashboard (a student cannot promote themselves).
- Passwords are handled and hashed by Supabase Auth. They never appear in tables or in the website.
- Attempts to log in are rate limited by Supabase, but use strong passwords anyway.

## Limits and notes
- To save data transfer, the portal loads tasks, feedback and announcements from the **last 90 days**. A "Show older tasks" button at the bottom of the task list loads everything. Live updates refresh only the table that changed, and there is no constant polling while live updates work.
- A free Supabase project may pause after a period of no activity. Restore it from the dashboard.
- Adding a track (e.g. AI) means editing the CHECK lists in `schema.sql` (or running `alter table ... drop/add constraint`) and the `DEPTS` list in `index.html`.
- To edit a task, delete and recreate it (editing in the UI is not built yet). You can also edit rows in Supabase's Table Editor.
- The page loads the Supabase library from the jsDelivr CDN (`@supabase/supabase-js@2`). For a stricter setup, pin an exact version.
