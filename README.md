# TikTok Watch-History Donation Page

Static page for the *Algorithmic Attitudes* study, hosted free on GitHub Pages.

What a participant does: **select their TikTok file → tap Send → enter the raffle.**

What the page does, all in the participant's browser:

1. Reads `rid` and `sid` from the link (`?rid=...&sid=12345`), or asks for the 5-digit study ID.
2. Opens the TikTok export (`.json` or `.zip`) **on their device**.
3. Keeps only video IDs and watch times from the watch-history section. Messages, searches, favorites, profile, and username are never kept or sent.
4. Shows a count and a **Send** button. "Review or remove videos" opens an optional list.
5. Sends the filtered rows to a Google Apps Script receiver, which saves a CSV into a private Google Drive folder. Large histories go in chunks of 50,000 rows, with up to 3 automatic retries.

GitHub stores no data. The receiver can only **add** files to one folder; nobody can read through it.

## Files

| File | Purpose |
| --- | --- |
| `index.html` | The donation page |
| `extract.js` | Extraction logic |
| `apps-script/Code.gs` | The Google receiver (paste into Apps Script) |
| `vendor/` | JSZip 3.10.1 (MIT), bundled so no outside server sees visitors |
| `test/` | `node test/test.js`, `node test/apps-script-test.js`, and `node test/browser-test.js` (full page; needs `npm i playwright jszip`) |

Saved files are named `<sid>_<rid>_<UTC time>_part1of1.csv`, with columns `rid, sid, video_id, watched_at`.

## 1. Set up the Google receiver

1. In Google Drive (personal account, with 2-Step Verification on), create a folder **TikTok Donations**. Don't share it.
2. Open the folder. Copy the ID from the address bar: the part after `/folders/`.
3. Go to script.google.com → **New project**. Delete the sample code and paste all of `apps-script/Code.gs`.
4. Replace `PASTE_FOLDER_ID_HERE` with the folder ID. Save.
5. **Deploy → New deployment** → gear icon → **Web app**.
   - Execute as: **Me**
   - Who has access: **Anyone**
6. Click Deploy, then authorize: choose your account → **Advanced** → **Go to project (unsafe)** → Allow. (The warning appears for every personal script; it's your own code.)
7. Copy the **Web app URL** (ends in `/exec`). Open it in a browser: it should say `"receiver running"`.

If you edit `Code.gs` later: **Deploy → Manage deployments → Edit → Version: New version → Deploy**. The URL stays the same.

## 2. Publish on GitHub Pages

1. Create a **public** repository named `tiktok_donation`.
2. Upload these files (Add file → Upload files), keeping the folders.
3. In `index.html`, set `SEND_URL` to the Web app URL and `RAFFLE_URL` to the raffle form. Fill in the IRB protocol number and Meyer's email.
4. Settings → Pages → Source: **Deploy from a branch** → `main` / `root` → Save.
5. The page is live at `https://<your-username>.github.io/tiktok_donation/` within a minute or two.

The repo is public, so it must contain **no participant data**, ever. Only code.

## Before launch

- [ ] Run your own TikTok export through the live page and tap Send. Check that a CSV appears in the Drive folder and the row count matches your watch history.
- [ ] Point the Qualtrics yes-donate end message at
  `https://<your-username>.github.io/tiktok_donation/?rid=${e://Field/source_id}&sid=${e://Field/StudyID}`
- [ ] Test on an iPhone and an Android phone, including a `.zip` file.
- [ ] Weekly during collection: move new CSVs from Drive to the encrypted drive, then delete them from Drive (30-day limit). Delete any junk files.
