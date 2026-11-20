# GDG On Campus Zagazig · Robotics Portal

Instructor/student portal where every student and instructor has a **Software** tab and a **Hardware** tab, backed by Google Sheets through Google Apps Script.

```
gdg-robotics-portal/
├── index.html      the whole website (HTML + CSS + JS)
├── Code.gs         Google Apps Script backend
└── assets/         optional: gdg-logo.png, robotics-logo.png
```

## 1. Try it now (demo mode)
Open `index.html` in a browser. With `CONFIG.API_URL` empty, the site stores sample data in your browser, so every feature works instantly. Demo logins are shown on the login screen.

## 2. Connect Google Sheets
1. Create a new Google Sheet.
2. **Extensions → Apps Script**. Delete the default code and paste in `Code.gs`.
3. Select the `setup` function and click **Run**. Approve the permissions. This creates the sheets **Users, Tasks, Progress, Feedback, Announcements** with sample rows.
4. **Deploy → New deployment → Web app**
   - Execute as: **Me**
   - Who has access: **Anyone**
5. Copy the **Web app URL** and paste it into `index.html`:
   ```js
   const CONFIG = { API_URL: 'https://script.google.com/macros/s/XXXX/exec', POLL_MS: 8000 };
   ```
6. Host `index.html` anywhere static (GitHub Pages, Netlify, or open it locally).
7. After any later change to `Code.gs`, use **Deploy → Manage deployments → Edit → New version**.

### Sheet columns
| Sheet | Columns |
|---|---|
| Users | id, name, role (`student`/`instructor`), username, password (hashed) |
| Tasks | id, department, title, description, deadline (`YYYY-MM-DD`) |
| Progress | taskId, studentId, status (`To do`/`In progress`/`Done`) |
| Feedback | id, department, taskId, studentId, text, date, author |
| Announcements | id, department, title, body, date, author |

**Add users** by running `addUser('s9','Name','student','username','password')` in the Apps Script editor. Passwords are stored as salted SHA-256 hashes, so don't type plain passwords into the sheet. Keep usernames lowercase.

### Live updates
The page re-reads the sheet every 8 seconds (`POLL_MS`) and when the tab regains focus. Edits made directly in the sheet appear without a manual refresh.

## 3. Real logos
Drop the exported logos into `assets/gdg-logo.png` and `assets/robotics-logo.png`. The page uses them automatically and falls back to a CSS version if they are missing.

## Security and limits of a public project
- **No API keys are in the frontend.** The only public value is the Web App URL, which is an endpoint, not a credential. The sheet itself stays private; only the script can read it.
- Anyone who knows the URL can call it, so the script checks every request: login with a hashed password, a session token (expires after 6 hours), role checks (students can't post or delete), and department checks (users only see their own department).
- This is lightweight auth, not enterprise-grade. There is no rate limiting or password reset, and passwords are sent to Apps Script over HTTPS. Don't store sensitive personal data. Use it for course management, not private records.
- Apps Script has quotas (daily executions and about 30 simultaneous executions) and requests take around 1 second. That is fine for a club-sized group.
- Writes are serialized with a script lock. If two people edit at the same moment, one waits briefly.
- Text is escaped in the page, and values that start with `=`, `+`, `-`, or `@` are neutralized before being written to the sheet.
- Demo mode stores everything in the visitor's browser and is not secure. Use it only for previews.
