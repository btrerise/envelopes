# Trerise Envelopes: setup

Total time: about 30 minutes, done once on a computer. Everything here is free: Firebase's free "Spark" plan (no credit card), GitHub Pages, and Google Apps Script.

**What's in this folder**

| File | Where it goes |
|---|---|
| `index.html`, `manifest.webmanifest`, `icon.svg`, `icon-180.png`, `icon-192.png`, `icon-512.png` | GitHub (the app itself) |
| `firestore.rules` | Pasted into Firebase (who's allowed in) |
| `apps-script/Code.gs`, `apps-script/appsscript.json` | Pasted into the Weekly Budget sheet (the sync) |
| `test/` | Not needed. These are the automated tests used to check the sync. |

---

## 1. Create the Firebase project

1. Go to **console.firebase.google.com** and sign in with your Google account.
2. Click **Create a project** and name it `trerise-envelopes`. If that name is taken, any name works.
3. Turn **Google Analytics** off. Click **Create project**.

## 2. Turn on sign-in and create the three accounts

1. In the left menu, go to **Build → Authentication → Get started**.
2. Choose **Email/Password**, switch the first toggle on, and click **Save**.
3. Open the **Users** tab and click **Add user** three times:

| Email | Password |
|---|---|
| `bentontrerise@gmail.com` | your choice |
| `kimberlee.trerise@gmail.com` | Kim's choice; she can reset it later |
| `envelopes-sync@example.com` | a long random password, used only by the sheet. Keep it for step 6. |

> These are app passwords, separate from your Google passwords. Creating all three now also stops anyone else from claiming those emails.
> To use different emails, change them in **three places**: here, `firestore.rules`, and the `PEOPLE` list in `index.html`.

## 3. Get the app's Firebase settings

1. Click the gear icon, then **Project settings**.
2. Scroll to **Your apps** and click the **`</>`** (Web) icon.
3. Name it `Envelopes`. Leave "Firebase Hosting" **unchecked**, and click **Register app**.
4. You'll see a block like `const firebaseConfig = { apiKey: "...", ... }`. Open `index.html` in VS Code, find `FIREBASE_CONFIG` near the bottom, and paste in these four values:
   - `apiKey`
   - `authDomain`
   - `projectId`
   - `appId`

> These values are meant to be public. They identify the project, and the rules in step 4 are what protect it.

## 4. Create the database and lock it down

1. In the left menu, go to **Build → Firestore Database → Create database**.
2. Pick a location close to you, such as `us-west2 (Los Angeles)`. **You can't change this later.**
3. Choose **Start in production mode**, then **Create**.
4. Open the **Rules** tab. Replace everything with the contents of `firestore.rules` and click **Publish**.

## 5. Put the app on GitHub Pages

1. On GitHub, create a new **public** repository, e.g. `envelopes`. Free accounts can only use Pages on public repos.
   - The code will be visible, but your budget won't be, because it lives in Firebase behind the sign-in.
2. Click **Add file → Upload files** and upload these six files:
   - `index.html`
   - `manifest.webmanifest`
   - `icon.svg`
   - `icon-180.png`
   - `icon-192.png`
   - `icon-512.png`
3. Go to **Settings → Pages**. Under **Source**, choose **Deploy from a branch**, then **main** and **/ (root)**, and click **Save**.
4. After a minute or two the app is live at `https://<your-username>.github.io/envelopes/`. It shows a sign-in screen. Leave it there for now.

## 6. Add the sync to the Weekly Budget sheet

Your account needs **edit** access to the sheet for this step. Kim owns it, so she may need to make you an editor first.

1. Open **Weekly Budget**, then go to **Extensions → Apps Script**. Rename the project (top left) to `Envelopes sync`.
2. Replace everything in `Code.gs` with the contents of `apps-script/Code.gs`.
3. Show the manifest:
   - Click the gear (**Project Settings**) and tick **Show "appsscript.json" manifest file in editor**.
   - Back in the editor, open `appsscript.json` and replace its contents with `apps-script/appsscript.json`.
4. Add the script properties:
   - In **Project Settings**, scroll to **Script Properties** and click **Add script property** for each row below.
   - Keep the names exactly as written.

| Property | Value |
|---|---|
| `FIREBASE_API_KEY` | the `apiKey` from step 3 |
| `FIREBASE_PROJECT_ID` | the `projectId` from step 3 |
| `SYNC_EMAIL` | `envelopes-sync@example.com` |
| `SYNC_PASSWORD` | the sync account's password from step 2 |

5. Deploy the web app, which lets the phones trigger an instant sync:
   - Click **Deploy → New deployment**. Click the gear next to "Select type" and choose **Web app**.
   - Set **Execute as: Me** and **Who has access: Anyone**, then click **Deploy**.
   - Google asks for permission:
     - Click **Authorize access** and choose your account.
     - If you see "Google hasn't verified this app", click **Advanced → Go to Envelopes sync**, then **Allow**. It's your own script, so that warning is expected.
   - Copy the **Web app URL**. It ends in `/exec`.
   - Back in **Script Properties**, add `WEB_APP_URL` with that URL as its value.
6. Run the first sync:
   - In the editor, choose **`setup`** in the function dropdown at the top and click **Run**.
   - The log should end with something like `Setup finished. {"ok":true,"tab":"October 2026","boxes":7,...}`.

> "Anyone" only means anyone with that long URL can ask the sheet to sync. They can't see or change any data. The app reads the URL from your locked-down database, so it never appears in the public GitHub code.

## 7. Put it on both phones

1. Open the GitHub Pages link in **Safari** (iPhone) or **Chrome** (Android) and sign in.
2. Add it to the home screen:
   - **iPhone:** tap **Share → Add to Home Screen**.
   - **Android:** tap **⋮ → Add to Home screen**.
3. Kim does the same on her phone, with her own email and password.

You'll stay signed in. From then on, tap the Envelopes icon like any other app.

---

## How it works day to day

- **The sheet is the boss.**
  - The app shows the boxes on the current month's tab (Eating Out, Unplanned, Vehicle Gas, Meds, Groceries, Planned Gifts, Date Night).
  - Budgets come from Kim's formulas, so if she changes a budget or adds a new box, the app follows.
  - A box is any label followed by a formula ending in `SUM(G3:G34)`-style ranges, the way she already builds them.
- **Recording in the app:**
  - The purchase goes into the first empty row of that box, usually within a few seconds.
  - It gets a small note (the triangle in the corner of the amount cell) saying who added it.
- **Kim typing in the sheet:**
  - Her entry appears in the app the next time either of you opens it, or within 10 minutes.
  - Edits and deletions sync both ways.
- **Double counting:** if you record an amount that's already in that box, the app shows what's there and asks before adding it again.
- **New month:**
  - When Kim creates the next tab, named like `November 2026`, the app switches to it on the 1st.
  - Until that tab exists, it stays on the newest one.
  - Tap the month name at the top of the app to pick a month manually.
- **Full box:**
  - If a box runs out of empty rows, the purchase waits in the app with a "Box full in sheet" tag.
  - Lengthen that box's `SUM(...)` range in the sheet and it fills in on the next sync.

**Safety rules the sync never breaks**
- It only writes into rows where both cells are completely empty.
- It never writes outside a box's SUM range.
- When you delete something in the app, it only clears that row in the sheet if the row still shows exactly what you deleted.

## If something looks wrong

| You see | Do this |
|---|---|
| **"Add your Firebase settings"** | `index.html` still has the `PASTE_...` placeholders (step 3). |
| **"…isn't on the household list"** | The signed-in email isn't in `firestore.rules`. Fix it and publish the rules again (step 4). |
| **"Waiting for the first sheet sync"** that never ends | In Apps Script, run `setup` again and read the log. Usually a Script Property is missing or misspelled. |
| **Red "sheet sync problem" banner** | The message says what's wrong, e.g. "The sync account could not sign in" means `SYNC_PASSWORD` doesn't match step 2. |
| **App edits reach the sheet only every 10 minutes** | `WEB_APP_URL` is missing. Add it and run `setup` again. |

## Moving over from the Claude version

The earlier Claude-hosted version had one purchase recorded: **Chili's with Mom and Dad, $124.43 (restaurants)**. Record it again in the new app under Eating Out, or have Kim type it into the sheet. You can delete the old Claude page whenever you like.
