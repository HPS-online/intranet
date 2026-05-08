# Highbury Primary School — Staff Intranet Handover Document

*Last updated: May 2026 — keep this file updated when anything changes*

---

## What This Project Is

A Firebase-hosted staff intranet for Highbury Primary School. It replaces OneNote for day-to-day staff operations. Staff log in with their `@schools.sa.edu.au` email and can access:

- **Home** — landing page with quick links, today's daybook snapshot, notices, and open tickets
- **Daybook** — daily staff absences, relievers, duties, NIT coverage, and daily notices
- **Tickets** — maintenance and IT issue tracking with photo attachments and progress notes
- **Reports** — ticket summaries by category, status, and priority
- **Admin** — user account management (admin-only)

---

## Key Accounts & Access

### Firebase (hosts the database and website)
- **URL:** https://console.firebase.google.com
- **Project name:** hps-net
- **Project ID:** hps-net
- **Live site URL:** https://hps-net.web.app
- **Owner account:** danielbotten@gmail.com
- **To add a new owner:** Firebase Console → gear icon → Project settings → Users and permissions → Add member → set role to Owner

### GitHub (stores all the code)
- **Owner account:** danielbotten@gmail.com (or associated GitHub username)
- **Repository:** contains `index.html`, `HANDOVER.md`
- **To deploy changes:** Edit `index.html` → open GitHub Desktop → commit → push → Firebase auto-deploys within ~60 seconds

### EmailJS (sends email notifications for new/updated tickets)
- **Account:** linked to school Gmail (check with Business Manager)
- **Service ID:** stored in `index.html` — search for `EMAILJS_SERVICE_ID`
- **Note:** Free tier allows 200 emails/month — sufficient for school use

---

## How to Deploy an Update

1. Open the project folder on your computer
2. Edit `index.html` using any text editor (Notepad, VS Code, etc.)
3. Open **GitHub Desktop**
4. You'll see `index.html` listed as a changed file
5. Type a short commit message (e.g. "Update staff list")
6. Click **Commit to main**
7. Click **Push origin**
8. Wait ~60 seconds, then hard-refresh the site with **Ctrl + Shift + R**

---

## How to Add a New Staff Member

New staff can self-register at the live site using their `@schools.sa.edu.au` email. However, accounts need admin approval before they can log in.

**To approve a new account:**
1. Log into the intranet as admin
2. Click **Admin** in the top nav
3. Find the pending account and approve it

**To add someone manually (e.g. TRTs, casuals):**
1. Log into the intranet as admin
2. Click **Admin** → **Add User**
3. Enter their school email and set a temporary password
4. They can reset their password from the login screen

---

## How to Add an Admin Account

1. Log into the intranet as an existing admin
2. Click **Admin** → find the user → click **Make Admin**

OR via Firebase directly:
1. Go to Firebase Console → Firestore Database → Data
2. Find the `allowed_users` collection
3. Find the user's document (their email address)
4. Change the `role` field from `staff` to `admin`

---

## Firestore Security Rules

Current rules (as of May 2026) — found in Firebase Console → Firestore → Rules:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    match /allowed_users/{email} {
      allow read: if request.auth != null && request.auth.token.email.matches('.*@schools\\.sa\\.edu\\.au');
      allow write: if request.auth != null &&
        get(/databases/$(database)/documents/allowed_users/$(request.auth.token.email)).data.role == 'admin';
    }

    match /tickets/{ticketId} {
      allow read, write: if request.auth != null &&
        exists(/databases/$(database)/documents/allowed_users/$(request.auth.token.email));
    }

    match /daybook/{dateId} {
      allow read, write: if request.auth != null &&
        exists(/databases/$(database)/documents/allowed_users/$(request.auth.token.email));
    }

  }
}
```

---

## Firestore Data Structure

| Collection | Document ID | Fields |
|---|---|---|
| `allowed_users` | staff email address | `role` (staff/admin), `name`, `createdAt` |
| `tickets` | auto-generated ID | `title`, `description`, `category`, `priority`, `status`, `assignedTo`, `createdBy`, `createdAt`, `photoURL`, `notes[]` |
| `daybook` | date string `YYYY-MM-DD` | `rows[]` (absent/reliever/duty/nit), `dutyTeam`, `pblFocus`, `assembly`, `notices`, `updatedAt`, `updatedBy` |

---

## Firebase Free Tier Limits

The project runs on Firebase's **Spark (free) plan**. Limits that matter for a school:

| Resource | Free limit | Expected usage |
|---|---|---|
| Firestore reads | 50,000/day | Well within limits |
| Firestore writes | 20,000/day | Well within limits |
| Hosting bandwidth | 10 GB/month | Well within limits |
| Authentication | Unlimited | No issue |

No billing is expected unless usage grows dramatically (e.g. storing large video files).

---

## If Something Breaks

**Site won't load at all:**
- Check Firebase Console → Hosting — look for a failed deployment
- Re-deploy from GitHub Desktop

**Staff can't log in:**
- Check Firebase Console → Authentication — make sure the account exists
- Check Firestore → allowed_users — make sure their email document exists

**Daybook not saving:**
- Check Firestore → Rules — make sure the daybook rule is published
- Check browser console (F12) for specific error messages

**Ticket emails not sending:**
- Check EmailJS dashboard for error logs
- EmailJS free tier resets monthly — may have hit the 200 email limit

---

## Contacts & Handover Notes

| Role | Name | Contact |
|---|---|---|
| Deputy Principal (project owner) | Daniel Botten | daniel.botten315@schools.sa.edu.au |
| ICT Support | Nouman | — |
| Business Manager | — | — |

---

## Version History

| Date | Change |
|---|---|
| May 2026 | Initial build — ticketing system with Firebase Auth, Firestore, EmailJS |
| May 2026 | Added Home landing page with quick tiles and daily notices |
| May 2026 | Added Daybook module (absences, duties, NIT, daily info) |

---

*This document lives in the GitHub repository alongside `index.html`. Update it whenever you make a significant change to the system.*
