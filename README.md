# Red Sun Studios: My details page

Members sign in with Google and see only their own studio, door code, wifi, rent, open invoices, and recent payments. Nobody can see anyone else's row.

How it works: the page (`index.html`, on GitHub Pages) shows a "Sign in with Google" button. Google gives the page a signed token that says which email signed in. The page sends that token to a small Apps Script in the Studio Management sheet. The script checks the token with Google, finds that email in the Members tab, and sends back that one member's details. The sheet itself is never shared with members.

Files:
- `index.html`: the member page
- `Code.gs`: the Apps Script that goes inside the Studio Management sheet

## 1. Make the management sheet a Google Sheet

Apps Script only works on a native Google Sheet.
1. In Drive, open `Red Sun Studios - Studio Management.xlsx`.
2. File > Save as Google Sheets. Use the new Google Sheet from now on, and delete or archive the .xlsx.
3. Keep it shared with Co-Green St people only.

## 2. Add two columns to the Members tab

Column I is the Lamartine + Inside Door checkbox. The script finds the other columns by their header name, so add these in J and K (row 3):
- `Door code`: each member's keypad code. Leave it blank if you connect RemoteLock (step 5).
- `Agreement link`: a Drive link to their signed agreement (optional).

Check the `Email` column: it must hold the address each person will sign in with. Several addresses can share one cell, separated by `;`. Members with no email on file (Niara, Sopheak, Deb right now) can't sign in until one is added.

Only members whose Status is Current or Starting see the door code and wifi. Departed members can still see their balance and payments.

## 3. Create a Google sign-in client ID (free, about 10 minutes)

1. Go to console.cloud.google.com and create a project, for example "Red Sun Studios".
2. Google Auth Platform (or APIs & Services) > OAuth consent screen: User type External. App name "Red Sun Studios", support email ellie@coeverything.co. Add no extra scopes (sign-in only uses name and email). Publish the app so it isn't limited to test users. Sign-in with only basic scopes doesn't need Google verification.
3. Credentials > Create credentials > OAuth client ID > Web application.
   - Authorized JavaScript origins: `https://YOUR-GITHUB-USERNAME.github.io`
   - No redirect URIs needed.
4. Copy the client ID (it ends in `.apps.googleusercontent.com`).

## 4. Install the script

1. In the Studio Management Google Sheet: Extensions > Apps Script. Delete what's there and paste in `Code.gs`. Save.
2. Project Settings (gear icon) > Script properties. Add:
   - `GOOGLE_CLIENT_ID`: the client ID from step 3
   - `WIFI_NETWORK`: `Co-Green St Guest`
   - `WIFI_PASSWORD`: the password
   - `NICKEL_URL` (optional): defaults to https://94greenst.nickelpayments.com/
3. In `testLookup()`, put a member's email between the quotes, select `testLookup`, and click Run. Approve the permissions. The log shows exactly what that member would see. Clear the email afterwards.
4. Deploy > New deployment > type Web app.
   - Execute as: Me
   - Who has access: Anyone
   - Deploy, then copy the Web app URL (ends in `/exec`).

"Anyone" means anyone can send a request to the script, but it only answers with data for a Google-verified email that's on the Members tab, and only that person's row.

When you change `Code.gs` later: Deploy > Manage deployments > edit > Version: New version. That keeps the same URL.

## 5. Optional: read door codes live from RemoteLock

RemoteLock turned on API access for the co-green@coeverything.co account on May 12, 2026 (Jess Dion's email: "The same credentials to the RemoteLock account will give you access to the Developer Portal").
1. Sign in at developer.remotelock.com with the co-green@coeverything.co RemoteLock login, and open https://developer.remotelock.com/oauth/applications/new
2. Name: `Red Sun Studios My Details`. Redirect URI: `urn:ietf:wg:oauth:2.0:oob` (the form requires one; this script never uses it). Submit.
3. The next page shows an Application ID and a Secret. Keep the secret out of email and chat.
4. Add Script properties `REMOTELOCK_CLIENT_ID` (the Application ID), `REMOTELOCK_CLIENT_SECRET` (the Secret), and `REMOTELOCK_DOOR_NAME` (`Green St`).
5. Select `testRemoteLock` and click Run. The log should say "Token OK", then list each access person's name, email and whether they have a PIN (it never prints the PIN), with "NO MATCH" next to anyone whose RemoteLock email isn't on the Members tab. Fix those emails in RemoteLock or the sheet so they match.

The script then finds the member's access person in RemoteLock by email and shows that PIN, plus the name of every door it currently opens (for example Green St, Lamartine and both inside locks for someone in Coop Social Club too). Codes don't need to live in the sheet. Door names come from what each lock is called in RemoteLock, so rename a lock there if you want a friendlier label. If a person has access through a lock group, the page shows the group's name. The member's email in RemoteLock must match the sheet. If RemoteLock doesn't answer, the page falls back to the `Door code` column and labels it with `REMOTELOCK_DOOR_NAME`. This part hasn't been tested against your account; run `testLookup()` after adding the keys to confirm the code comes through.

## Managing members and door codes

Once `Code.gs` is pasted in and saved, reload the sheet. A **Red Sun Studios** menu appears next to Help. (The first time you use it, Google asks you to approve permissions.)

- **Add a new member…** Fill in name, email, studio, whole or half share, their monthly share, start date, and optionally a pro-rated first month, deposit and license. It writes one row to Members and one to Studio History, in the first empty row of each, and only in the typed-in columns, so no formula is touched. It won't add a duplicate name or email, or studio 2.
- **Change studio, share or rate…** For someone already on the sheet who's moving studios, going from a share to the whole studio, or changing rate. It ends their current Studio History row the day before the change and adds the new one.
- **Someone is leaving…** Pick the member and their last day. It sets the end date on their open Studio History rows. Their payments and balance stay on the sheet; the Status column switches to Departed once the report month passes their last day.
- **Preview door code changes** lists what Apply would do in RemoteLock and changes nothing:
  - people in the building today with no code: create one, with access to Green St
  - people in the building with a code but no studio door: add Green St
  - people whose last day has passed: remove their studio doors (Green St and Co-Green Street). If that leaves them no doors, their code is turned off; if they still have Coop Social Club doors, those stay.
  - people with no email on the Members tab are listed as skipped
- **Apply door code changes** asks you to confirm, makes those changes, and records each one on a **Door Log** tab.
- **Turn on door sync (1st, 15th, last day)** runs Apply around 6am on the 1st, the 15th and the last day of each month. Someone whose last day is the 31st loses access on the morning of the 1st; someone starting on the 1st gets their code that morning. Turn it on once you are comfortable with what Preview shows. You can still run Apply by hand any time.

The script only ever changes people whose email is on the Members tab. Staff, Coop Social Club members and the shared Red Sun Studios codes are never touched.

**Lamartine + Inside Door checkbox (Members column I).** Tick it for studio members who are also in Coop Social Club. The next Apply gives them Lamartine and the Inside Door group on top of Green St (only the ones they don't already have). The Add a new member form has the same checkbox. Unticking does not remove those doors, because Coop Social Club access may come from elsewhere; remove them in RemoteLock if needed.

Optional Script properties for this part: `GRANT_DOOR` (default `Green St`), `STUDIO_DOORS` (default `Green St, Co-Green Street`) and `EXTRA_DOORS` (default `Lamartine, Inside Door`), comma separated, using the exact lock or group names in RemoteLock.

The My details page shows a member's code only while their Studio History says they're in the building today, so it stops showing the code the day after their last day.

**Members can change their own code.** On the My details page, "Change my code" lets a current member pick a new code. It has to be 6 numbers, and not one digit repeated or a straight run like 123456. New codes made by Apply are also 6 digits (change the Script property `CODE_LENGTH` if your locks need a different length). RemoteLock refuses a code someone else already has. Each member gets 3 tries an hour. The new code goes straight to RemoteLock and replaces the old one on every door they have; it is never written to the sheet. The Door Log gets a "code change" line with the member's name only.

## 6. Put the page on GitHub Pages

1. In `index.html`, fill in the settings near the top:
   - `GOOGLE_CLIENT_ID`: from step 3
   - `SCRIPT_URL`: from step 4
   - `MEMBERS_DOC_URL`: the members-only Google Doc, once it exists
2. Upload `index.html` to a GitHub repo, for example `red-sun-studios`, in a folder called `my-details`. If the studio map is in the same repo, it can stay at the root.
3. Settings > Pages > Deploy from a branch > `main`, `/ (root)`.
4. The page will be at `https://YOUR-GITHUB-USERNAME.github.io/red-sun-studios/my-details/`. The origin in step 3 has to match the `https://YOUR-GITHUB-USERNAME.github.io` part.

The page has no secrets in it; everything private comes from the script after sign-in.

## Keeping it accurate

The page shows what's in the sheet, so a payment that isn't in the Payment Log yet shows as owed. Log payments before telling members about the page.

## When someone leaves

Use Red Sun Studios > Someone is leaving… and then apply door code changes (or let the daily sync do it).
