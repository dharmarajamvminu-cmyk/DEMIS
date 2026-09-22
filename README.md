# ධර්මරාජ විද්‍යාලය — ශිෂ්‍ය ලියාපදිංචි පද්ධතිය

Files දෙකකින් සමන්විතයි:

- **`demis-form.html`** — දෙමව්පියන් සඳහා public ලියාපදිංචි ෆෝරමය. ඕනෑම කෙනෙකුට open කර පුරවන්න පුළුවන්, login අවශ්‍ය නැත.
- **`admin.html`** — කාර්ය මණ්ඩලය සඳහා පුවරුව. **login අවශ්‍යයි** — ලියාපදිංචි වූ email/password එකකින් පමණයි ඇතුල් විය හැක්කේ.

දෙකම එකිනෙකට වෙනම, self-contained HTML files. Firebase project එකම දෙකෙන්ම භාවිත වේ.

---

## 1 වන පියවර — Firebase Project එකක් හදන්න

1. https://console.firebase.google.com වෙත ගොස් login වෙන්න.
2. **Add project** → නමක් දෙන්න → Create.
3. **Build → Firestore Database** → **Create database** → **Test mode** තෝරන්න (පසුව rules වෙනස් කරමු).
4. **Build → Authentication** → **Get started** → **Sign-in method** → **Email/Password** enable කරන්න.
5. **Build → Storage** → **Get started** → default location එකෙන්ම **Done** (ශිෂ්‍යයන්ගේ ඡායාරූප ගබඩා කරන්නේ මෙතනයි — පසුව rules වෙනස් කරමු).
6. Project overview → **</> (Web)** → app එකකට නමක් දී **Register app**.
7. පෙන්වන `firebaseConfig` object එක copy කරගන්න.

## 2 වන පියවර — Config එක files දෙකටම දාන්න

`demis-form.html` සහ `admin.html` දෙකේම `firebaseConfig` object එක සොයාගෙන (ctrl+F කරන්න "YOUR_API_KEY"), copy කරගත් අගයන් වලින් replace කරන්න. **files දෙකේම එකම config එක දාන්න.**

## 3 වන පියවර — පළමු පරිපාලක (admin) ගිණුම හදන්න

`admin.html` එකෙන් කෙනෙකුට ඇතුල් වෙන්න කලින්, අවම වශයෙන් එක් admin ගිණුමක්වත් manual විදිහට හදන්න ඕනෑ:

1. Firebase Console → **Authentication → Users → Add user** → email + password එකක් දෙන්න (උදා: ඔබේ email එක).
2. එතනින් හැදුණු user ගේ **User UID** එක copy කරගන්න.
3. **Firestore Database → Data → Start collection** → collection ID: `staff`
4. Document ID තැන ඒ **User UID** එකම paste කරන්න → fields මෙසේ දාන්න:
   - `name` (string) — ඔබේ නම
   - `email` (string) — දුන් email එකම
   - `role` (string) — `admin`
5. **Save**.

දැන් ඒ email/password එකෙන් `admin.html` වෙත login විය හැක. **මින්පසු අලුත් teacher/admin ගිණුම් admin dashboard එකෙන්ම හදාගත හැක** — Firestore Console එකට යළි යාමට අවශ්‍ය නැත.

## 4 වන පියවර — Firestore Security Rules සකසන්න

**Firestore Database → Rules** වෙත ගොස් පහත rules දාන්න:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    function isStaff() {
      return request.auth != null &&
        exists(/databases/$(database)/documents/staff/$(request.auth.uid));
    }
    function isAdmin() {
      return isStaff() &&
        get(/databases/$(database)/documents/staff/$(request.auth.uid)).data.role == 'admin';
    }

    match /demis_registrations/{docId} {
      allow create: if true;               // public ෆෝරමයට ඕනෑම කෙනෙකුට ලියන්න පුළුවන්
      allow read:   if isStaff();          // කියවීම කාර්ය මණ්ඩලයට පමණයි
      allow update, delete: if isAdmin();  // මකා දැමීම admin ට පමණයි
    }

    match /staff/{uid} {
      allow read:  if request.auth != null && request.auth.uid == uid;
      allow write: if isAdmin();           // staff ගිණුම් හැදීම/ඉවත් කිරීම admin ට පමණයි
    }
  }
}
```

**Publish** click කරන්න.

## 4.5 වන පියවර — Storage Rules සකසන්න (ඡායාරූප සඳහා)

**Storage → Rules** වෙත ගොස් පහත rules දාන්න — ඕනෑම කෙනෙකුට (public form එකෙන්) 3MB දක්වා ඡායාරූප upload කළ හැක, නමුත් ඒවා බැලිය හැක්කේ පද්ධතියේ getDownloadURL() token එක දන්නා අයට පමණි (admin panel එකෙන් පෙන්වන URL එක token-protected):

```
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /student-photos/{fileName} {
      allow read: if true;
      allow write: if request.resource.size < 3 * 1024 * 1024
                   && request.resource.contentType.matches('image/.*');
      allow delete: if false;
    }
  }
}
```

**Publish** click කරන්න.

> **වැදගත්:** `admin.html` එකේ "PDF ලෙස බාගන්න" බටනය ඡායාරූපය PDF එකට ඇතුළත් කිරීමට browser එකෙන්ම කෙලින්ම ඡායාරූපය කියවීමට උත්සාහ කරයි. සමහර browsers වල මෙය CORS දෝෂයක් නිසා අසාර්ථක විය හැක. එය නිරාකරණය කිරීමට Firebase Storage bucket එකට CORS සැකසුමක් දාන්න අවශ්‍ය විය හැක ([gsutil cors](https://firebase.google.com/docs/storage/web/download-files#cors_configuration) හරහා) — නමුත් ඡායාරූපය නැතිව වුවත් PDF එක සාදනු ලැබේ.

## 5 වන පියවර — GitHub වෙත upload කර Pages enable කරන්න

1. GitHub → **New repository** → Public → Create.
2. `demis-form.html` සහ `admin.html` upload කරන්න (**Commit changes**).
3. **Settings → Pages** → Branch: `main`, `/ (root)` → Save.
4. විනාඩි කිහිපයකින්:
   - ලියාපදිංචි ෆෝරමය: `https://YOUR_USERNAME.github.io/REPO/demis-form.html`
   - කාර්ය මණ්ඩල පුවරුව: `https://YOUR_USERNAME.github.io/REPO/admin.html`

---

## admin.html එකෙන් කරන්න පුළුවන් දේවල්

**Admin (පරිපාලක):**
- ලියාපදිංචි වූ සියලුම ශිෂ්‍යයන් බලන්න, නම/ශ්‍රේණිය/දුරකථනයෙන් සොයන්න, පන්තියක් අනුව filter කරන්න
- එක් එක් ලියාපදිංචියේ සම්පූර්ණ විස්තර හා ඡායාරූපය බලන්න, අවශ්‍ය නම් මකන්න
- එක් ශිෂ්‍යයෙකුගේ විස්තර PDF ලෙස බාගන්න
- "පන්ති අනුව" tab එකෙන් සෑම පන්තියකම ශිෂ්‍ය සංඛ්‍යාව බලන්න
- සියලුම දත්ත (හෝ තෝරාගත් පන්තියක දත්ත පමණක්) CSV හෝ Excel (.xlsx) file එකක් ලෙස බාගත කරගන්න
- අලුත් teacher හෝ admin ගිණුම් සෑදීම
- පවතින කාර්ය මණ්ඩල ගිණුම්වල පද්ධති ප්‍රවේශය ඉවත් කිරීම

**Teacher (ගුරුවරයා):**
- ලියාපදිංචි වූ සියලුම ශිෂ්‍යයන් බලන්න, සොයන්න, පන්තියක් අනුව filter කරන්න
- එක් ශිෂ්‍යයෙකුගේ විස්තර PDF ලෙස බාගන්න, CSV/Excel බාගත කරන්න
- (staff ගිණුම් කළමනාකරණය කළ නොහැක)

### වැදගත් සටහනක්
"ප්‍රවේශය ඉවත් කරන්න" click කළ විට, ඒ පුද්ගලයාට **admin.html** එකට ඇතුල් විය නොහැකි වේ (Firestore එකේ staff record එක ඉවත් වන නිසා). නමුත් ඒ පුද්ගලයාගේ Firebase Authentication ගිණුමම මකා දැමීමට අවශ්‍ය නම්, Firebase Console → Authentication → Users වෙතින් manual විදිහට මකන්න.

---

## අලුතින් එකතු කළ දේවල්

- **ශිෂ්‍ය ඡායාරූපය (කැමරාව හෝ ගොනුව)** — ලියාපදිංචි ෆෝරමයේ 1 වන පියවරේදී දෙමව්පියන්ට "කැමරාවෙන් ගන්න" බටනයෙන් දුරකථනයේ කැමරාව කෙලින්ම භාවිත කර ඡායාරූපයක් ගත හැක, නැතහොත් "ගොනුවක් තෝරන්න" බටනයෙන් දැනටමත් ඇති ඡායාරූපයක් upload කළ හැක (JPG/PNG/WebP, උපරිම 3MB). තෝරාගත් ඡායාරූපය ඉවත් කර නැවත තෝරාගැනීමටත් හැක. එය Firebase Storage වෙත upload වී, admin panel එකේ table එකේ සහ විස්තර පෙනුමේ පෙන්වනු ලැබේ.
- **එක් එක් ශිෂ්‍යයාට අනන්‍ය QR කේතයක්** — admin panel එකේ එක් එක් ශිෂ්‍යයෙකුගේ විස්තර පෙනුමේ (modal) ඒ ශිෂ්‍යයාටම අනන්‍ය QR කේතයක් (ඇතුළත්වීමේ අංකය, නම, පන්තිය සහ ලියාපදිංචි ID එක encode කර ඇත) පෙන්වයි — "QR කේතය බාගන්න" බටනයෙන් PNG ලෙසත්, "PDF ලෙස බාගන්න" බටනයෙන් ලැබෙන එක් ශිෂ්‍යයෙකුගේ PDF එකේත් එය ඇතුළත් වේ. එසේම toolbar එකේ **"QR කාඩ්පත් (PDF)"** බටනයෙන් (Class Filter එකක් තෝරා ඇත්නම් ඒ පන්තියේ පමණක්, නැත්නම් සියලුම) ශිෂ්‍යයන්ගේ QR කේත + නම + පන්තිය සහිත කපා හැඳුනුම්පත් ලෙස පාවිච්චි කළ හැකි printable PDF එකක් එකවර සාදාගත හැක.
- **ශ්‍රේණිය + පන්තියේ නම වෙන වෙනම** — දැන් ශ්‍රේණිය (1-13) dropdown එකකින් හා පන්තියේ නම වෙනම text field එකකින් ලබා ගැනේ (උදා: "4" + "රත්නජිත්" → "4 - රත්නජිත්"), පසුව නිවැරදිව පන්ති අනුව වර්ග කිරීමට හැකි වන පරිදි.
- **Search + Class Filter** — admin panel එකේ toolbar එකේ නම/ශ්‍රේණිය/දුරකථනය අනුව සෙවීමට අමතරව, දැනට ලියාපදිංචි වී ඇති පන්ති ලැයිස්තුවෙන් එකක් තෝරා filter කිරීමටත් හැක.
- **"පන්ති අනුව" tab එක** — සෑම පන්තියකම ලියාපදිංචි වූ ශිෂ්‍ය සංඛ්‍යාව එකවර බැලිය හැක. පන්තියක් click කළ විට එම පන්තියේ ශිෂ්‍යයන් පමණක් පෙන්වන පරිදි ප්‍රධාන tab එකට filter කර යොමු කරයි.
- **Excel බාගැනීම** — CSV එකට අමතරව, "Excel" බටනයෙන් `.xlsx` file එකක් ලෙස බාගත හැක. Class Filter එකක් තෝරා ඇත්නම්, ඒ පන්තියේ ශිෂ්‍යයන් පමණක් ඇතුළත් වූ වෙනම Excel file එකක් ලැබේ — එනම් එක් එක් පන්තියේ ගුරුවරයාට තමන්ගේම ලැයිස්තුව වෙන වෙනම ලබාගත හැක.
- **PDF (එක් එක් ශිෂ්‍යයා)** — විස්තර පෙනුමේ (modal) "PDF ලෙස බාගන්න" බටනයෙන් එක් ශිෂ්‍යයෙකුගේ සම්පූර්ණ ලියාපදිංචි විස්තර සහිත PDF එකක් බාගත හැක (ඡායාරූපය සමගින්).

### පන්ති අනුව සංවිධානය කිරීම සඳහා අදහසක්

සෑම ශිෂ්‍යයෙකුටම දැන් නිශ්චිත **ශ්‍රේණියක්** සහ **පන්තියේ නමක්** තිබෙන නිසා:
1. "පන්ති අනුව" tab එකෙන් සෑම පන්තියකම මුළු ශිෂ්‍ය සංඛ්‍යාව ලේසියෙන් බැලිය හැක.
2. පන්තියක් click කර, ඒ පන්තියේ ශිෂ්‍යයන් පමණක් ලැයිස්තුගත කර, "Excel" බටනයෙන් ඒ පන්තියටම වෙනම attendance/register list එකක් සාදාගත හැක.
3. අනාගතයේදී තවත් ඉදිරියට ගෙන යාමට අවශ්‍ය නම්: teacher ගිණුම් සෑදීමේදී (කාර්ය මණ්ඩල පුවරුවේ) එක් එක් teacher ට එක් පන්තියක් "පවරා" (staff document එකට `assignedClass` field එකක් එකතු කර) ඒ teacher ට තමන්ගේ පන්තියේ ශිෂ්‍යයන් පමණක් පෙනෙන පරිදි Firestore rules සකසාගත හැක.

## Submit වූ දත්ත Firebase Console එකෙන් සෘජුවම බලන්නේ කෙසේද?

Firebase Console → **Firestore Database → Data** → `demis_registrations` collection එක යටතේ එක් එක් submission එකක් document එකක් විදිහට පෙනේ.
