# ගුරු දත්ත ගබඩාව — Teacher Database Management System

Firebase Authentication + Cloud Firestore මත ක්‍රියාත්මක වන, real-time, mobile-friendly Teacher Database System එකකි.

## Project ව්‍යුහය
```
teacher-database/
├── index.html          → auth state අනුව login/dashboard වෙත redirect කරයි
├── login.html           → Admin login පිටුව
├── dashboard.html        → ප්‍රධාන dashboard (stats + add/edit form + list)
├── css/style.css
├── js/firebase-config.js → Firebase project configuration (දැනටමත් ඇතුළත් කර ඇත)
├── js/auth.js            → Login / Logout / route guard
├── js/teachers.js         → Teacher CRUD, real-time list, search, CSV export, print
├── js/dashboard.js        → Dashboard bootstrap
├── firestore.rules
└── README.md
```

## 1. Firebase Console Setup

1. **Authentication** → Sign-in method → *Email/Password* enable කරන්න.
2. **Authentication** → Users → admin ගිණුමක් (email + password) manually add කරන්න — මෙම system එකේ self-signup නොමැත, admin ගිණුම console එකෙන් සාදන්න.
3. **Firestore Database** → create database (production mode).
4. Firestore → **Rules** tab → මෙම project එකේ `firestore.rules` ගොනුවේ අන්තර්ගතය paste කර **Publish** කරන්න.

## 2. Firebase Configuration

`js/firebase-config.js` ගොනුව තුළ ඔබගේ project (`demis-9e3c8`) configuration එක දැනටමත් ඇතුළත් කර ඇත. වෙනස් project එකකට මාරු වන්නේ නම් පමණක් එය සංස්කරණය කරන්න.

## 3. Hosting (locally හෝ Firebase Hosting)

**Locally පරීක්ෂා කිරීමට** — `type="module"` scripts CORS restriction නිසා සරලව ගොනුව double-click කර open කළ නොහැක. VS Code "Live Server" extension එක හෝ:
```
npx serve teacher-database
```
වැනි local server එකක් හරහා open කරන්න.

**Production සඳහා** (Firebase Hosting):
```
npm install -g firebase-tools
firebase login
firebase init hosting     # public directory ලෙස "teacher-database" තෝරන්න
firebase deploy
```

## 4. NIC Duplicate වැළැක්වීම

සෑම ගුරුවරයෙකුගේම Firestore document ID එක ලෙස ඔවුන්ගේ **NIC අංකය** භාවිතා කර ඇත. මෙමගින්:
- එකම NIC එකකින් record දෙකක් සෑදීම structurally වළක්වයි.
- Save කිරීමට පෙර පවතින document එකක් ඇත්දැයි පරීක්ෂා කර user-friendly Sinhala error message එකක් පෙන්වයි.
- Edit mode තුළ NIC field එක disable කර ඇත (NIC වෙනස් කිරීමට අවශ්‍ය නම් record එක delete කර අලුතින් සාදන්න).

## 5. විශේෂාංග සාරාංශය

- Firebase Authentication හරහා admin login/logout
- Cloud Firestore `onSnapshot()` real-time updates
- Add / Edit / Delete (confirmation dialog සමඟ)
- Name / NIC / Phone / Position / Grade අනුව real-time search
- Sinhala Unicode සමඟ නිවැරදිව පෙන්වන CSV export (filtered results පමණක් export වේ)
- Teacher list සහ තනි teacher details print කිරීම
- Desktop (2-column), Tablet සහ Mobile (single-column, horizontal-scroll table) සඳහා සම්පූර්ණ responsive design

## 6. සැලකිය යුතුයි

- Firebase config values (`apiKey` ඇතුළුව) client-side app එකක් තුළ public වීම සාමාන්‍යයි — සත්‍ය ආරක්ෂාව සපයනු ලබන්නේ **Firestore Security Rules** සහ **Authentication** මගිනි, config එක සැඟවීමෙන් නොවේ.
- මෙම system එකේ admin registration UI එකක් නොමැත — අලුත් admin ගිණුම් Firebase Console හරහා පමණක් සාදන්න.
