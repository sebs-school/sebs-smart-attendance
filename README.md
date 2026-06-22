# SEBS Smart Attendance — Setup Guide

## Files in this package:
- `index.html`  → Main web app (your frontend)
- `Code.gs`     → Google Apps Script backend
- `README.md`   → This guide

---

## STEP 1: Google Sheet Setup

1. Open https://sheets.google.com
2. Create a **new spreadsheet** called `SEBS Attendance`
3. Note the Spreadsheet ID from the URL:
   `https://docs.google.com/spreadsheets/d/SPREADSHEET_ID/edit`
4. **Do NOT add any sheets manually** — the script creates them automatically.

---

## STEP 2: Google Apps Script

1. In your Google Sheet, click **Extensions → Apps Script**
2. Delete all existing code in `Code.gs`
3. Paste the entire contents of `Code.gs` from this package
4. Save (Ctrl+S)

---

## STEP 3: Deploy as Web App

1. Click **Deploy → New Deployment**
2. Click the gear ⚙️ icon next to "Type" → Select **Web App**
3. Set:
   - **Description**: `SEBS Attendance API`
   - **Execute as**: `Me`
   - **Who has access**: `Anyone`
4. Click **Deploy**
5. **Copy the Web App URL** — it looks like:
   `https://script.google.com/macros/s/XXXXXX.../exec`

---

## STEP 4: Update Frontend

1. Open `index.html` in a text editor
2. Find this line near the top of the `<script>` section:
   ```
   const API = 'https://script.google.com/macros/s/AKfycbx.../exec';
   ```
3. Replace it with **your Web App URL** from Step 3
4. Save the file

---

## STEP 5: Face Recognition Models

The app loads face-api.js models from GitHub. You need to host the model
files OR use an existing CDN. The current config points to:
```
https://raw.githubusercontent.com/sebs-school/sebs-smart-attendance/main/models
```

**To host your own models:**
1. Download models from: https://github.com/justadudewhohacks/face-api.js/tree/master/weights
2. You need these files:
   - ssd_mobilenetv1_model-weights_manifest.json
   - ssd_mobilenetv1_model-shard1
   - face_landmark_68_model-weights_manifest.json
   - face_landmark_68_model-shard1
   - face_recognition_model-weights_manifest.json
   - face_recognition_model-shard1
   - face_recognition_model-shard2
3. Upload to your GitHub repo or any web server
4. Update the `MODELS` constant in `index.html`:
   ```
   const MODELS = 'https://your-domain.com/models';
   ```

**Alternatively**, use the public CDN:
```
const MODELS = 'https://cdn.jsdelivr.net/gh/justadudewhohacks/face-api.js@0.22.2/weights';
```

---

## STEP 6: Deploy the Frontend

You can host `index.html` using any of:

### Option A: GitHub Pages (Free)
1. Create a repo on GitHub
2. Upload `index.html` and `logo.png`
3. Go to Settings → Pages → Deploy from branch `main`
4. Your URL: `https://yourusername.github.io/repo-name`

### Option B: Netlify (Free, Recommended)
1. Go to https://netlify.com
2. Drag & drop your folder with `index.html` and `logo.png`
3. Get instant URL

### Option C: Local use
Just open `index.html` directly in Chrome/Edge.
⚠️ Camera works only on HTTPS or localhost.

---

## STEP 7: Logo

Place your school logo as `logo.png` in the same folder as `index.html`.
Recommended size: 128×128px or 256×256px, transparent PNG.

---

## HOW IT WORKS

### First-time Student (New Face):
1. Student stands in front of camera
2. Face detected → circular ring appears
3. Blink or move head → Liveness verified ✅
4. Registration form pops up
5. Fill Name, Class → Save
6. Face embedding saved to Google Sheets
7. Attendance marked as Present

### Returning Student (Known Face):
1. Student stands in front of camera
2. Face detected → recognized instantly
3. Blink or move head → Liveness verified ✅
4. ✅ Attendance auto-marked
5. 30-second cooldown prevents duplicates
6. Already-marked check prevents double-marking per day

### Anti-Spoofing (Liveness Detection):
- Photo / screenshot will NOT work
- Requires either: eye blink OR head movement
- eSewa-style verification

---

## ATTENDANCE STATUS RULES

| Time          | Status     |
|---------------|------------|
| Before 7:30   | Present    |
| 7:30 – 8:00   | Late       |
| After 8:00    | Very Late  |

Adjust in `Code.gs`:
```javascript
const ON_TIME_HOUR = 7;  // 7:30
const ON_TIME_MIN  = 30;
const LATE_HOUR    = 8;  // 8:00
const LATE_MIN     = 0;
```

---

## GOOGLE SHEETS STRUCTURE (auto-created)

### Students Sheet
| StudentID | StudentName | AdmissionNumber | DateOfBirth | Class | Section | RollNumber | Gender | FatherName | MotherName | GuardianPhone | Address | RegisteredAt |

### Attendance Sheet
| AttendanceID | StudentID | StudentName | Date | DayOfWeek | Time | Status | Confidence | MarkedBy | Remarks |

### Embeddings Sheet
| StudentID | StudentName | Embedding | UpdatedAt |

---

## TROUBLESHOOTING

**Camera not working:**
- Must use HTTPS (or localhost)
- Grant camera permission in browser
- External webcam: should auto-detect; if not, try in Chrome

**Models not loading:**
- Check your MODELS URL is accessible
- Try the jsdelivr CDN option above
- Check browser console for 404 errors

**API not responding:**
- Redeploy with "New Deployment" (not update)
- Check that "Who has access" is set to "Anyone"
- Make sure you saved the Code.gs before deploying

**Face not recognized:**
- Ensure good lighting
- Look directly at camera
- Try re-registering the student
- Adjust threshold (0.48 → 0.52 for more lenient matching):
  `const FACE_THRESHOLD = 0.48;` in the HTML

---

## SECURITY NOTE

This system uses password-free login for school internal use.
If needed, add a proper login by replacing `doLogin()` with:
```javascript
function doLogin() {
  const password = prompt('Enter admin password:');
  if (password !== 'your-password-here') {
    alert('Wrong password!');
    return;
  }
  // ... rest of login
}
```
