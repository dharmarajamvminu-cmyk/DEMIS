# පාසල් දත්ත කළමනාකරණ පද්ධතිය — School Database System

Firebase Authentication + Cloud Firestore මත ක්‍රියාත්මක වන, real-time, mobile-friendly School Database System එකකි. Login වූ පසු **Hub** පිටුවකින් පහත මොඩියුල තුන අතරින් එකක් තෝරාගත හැක:

- 👩‍🏫 **ගුරු දත්ත ගබඩාව** (Teacher Database)
- 🎓 **සිසුන් දත්ත ගබඩාව** (Student Database)
- 📝 **ලකුණු පත්‍රය** (Mark Sheet — සිසුන් දත්ත ගබඩාවට සම්බන්ධ වේ)

## Project ව්‍යුහය
```
school-database-system/
├── index.html            → Admin login පිටුව (site root — GitHub/Firebase Hosting default)
├── hub.html                → Login වූ පසු මොඩියුලය තෝරන පිටුව
├── teachers.html            → ගුරු දත්ත ගබඩාව
├── students.html             → සිසුන් දත්ත ගබඩාව
├── marks.html                  → ලකුණු පත්‍රය
├── css/style.css
├── js/firebase-config.js        → Firebase project configuration
├── js/auth.js                    → Login / Logout / route guard (සියලුම පිටු බෙදාගනී)
├── js/hub.js                      → hub.html bootstrap
├── js/teachers.js / teachers-page.js   → Teacher CRUD + bootstrap
├── js/students.js / students-page.js   → Student CRUD + bootstrap
├── js/marks.js / marks-page.js          → Mark Sheet CRUD + bootstrap
├── firestore.rules
└── README.md
```

## 1. Firebase Console Setup

1. **Authentication** → Sign-in method → *Email/Password* enable කරන්න.
2. **Authentication** → Users → admin ගිණුමක් (email + password) manually add කරන්න — self-signup නොමැත.
3. **Firestore Database** → create database (production mode).
4. Firestore → **Rules** → මෙම project එකේ `firestore.rules` අන්තර්ගතය paste කර **Publish** කරන්න. (collections තුනටම එකම authenticated-admin rule එක අදාළ වේ.)

`js/firebase-config.js` තුළ ඔබගේ project (`demis-9e3c8`) configuration එක දැනටමත් ඇතුළත් කර ඇත.

## 2. Locally පරීක්ෂා කිරීම

`type="module"` scripts නිසා ගොනු double-click කර open කළ නොහැක — local server එකක් ඕනේ:
```
npx serve school-database-system
```
හෝ VS Code "Live Server" extension එක. **Production** සඳහා Firebase Hosting:
```
npm install -g firebase-tools
firebase login
firebase init hosting     # public directory ලෙස "school-database-system" තෝරන්න
firebase deploy
```

## 3. මොඩියුල තුනේම ක්‍රියාකාරීත්වය

සෑම මොඩියුලයකම common pattern එකක් අනුගමනය කරයි: real-time stats cards, add/edit form, search, view/edit/delete, print.

### 👩‍🏫 Teacher Database (`teachers.html` → collection `teachers`)
NIC අංකය document ID ලෙස භාවිතා කර duplicate NIC structurally වළක්වයි. CSV export ද ඇත.

### 🎓 Student Database (`students.html` → collection `students`)
Admission No එක document ID ලෙස භාවිතා කරයි. Fields: පුද්ගලික තොරතුරු, භාරකරු/දෙමාපිය තොරතුරු, පාසල් තොරතුරු (ශ්‍රේණිය/අංශය/ඇතුළත් වූ දිනය), සෞඛ්‍ය තොරතුරු (රුධිර කාණ්ඩය, වෛද්‍ය සටහන්). CSV export ද ඇත.

### 📝 Mark Sheet (`marks.html` → collection `marksheets`)
- "සිසුවා තෝරන්න" dropdown එක **Student Database** එකෙන් සජීවීව (real-time) fill වේ — එබැවින් ලකුණු පත්‍රයක් සෑදීමට පෙර සිසුවා Student Database එකට එක් කර තිබිය යුතුය.
- එක් එක් විෂයයක් "විෂයක් එක් කරන්න" මගින් dynamic row එකක් ලෙස එක් කර ලකුණු ඇතුළත් කරයි (pre-defined විෂය ලැයිස්තුවක් සහ "වෙනත්" custom option එකක් ඇත).
- මුළු ලකුණු සහ සාමාන්‍යය ස්වයංක්‍රීයව ගණනය වේ.
- Document ID එක `admissionNo__term` ලෙස සාදන බැවින් එකම සිසුවාට එකම වාරයට ලකුණු පත්‍ර දෙකක් සෑදිය නොහැක (duplicate error පෙන්වයි).
- Edit mode තුළ සිසුවා/වාරය වෙනස් කළ නොහැක (Delete කර අලුතින් සාදන්න).
- View modal එක printable "report card" ආකෘතියකින් පෙන්වයි.

## 4. සැලකිය යුතුයි

- Firebase config values client-side app එකක් තුළ public වීම සාමාන්‍යයි — සත්‍ය ආරක්ෂාව සපයනු ලබන්නේ **Firestore Security Rules** සහ **Authentication** මගිනි.
- අලුත් admin ගිණුම් Firebase Console හරහා පමණක් සාදන්න.
- Mark Sheet එක Student Database මත රඳා පවතී — සිසුවෙකු delete කළ පසුත් ඔහුගේ පැරණි ලකුණු පත්‍ර වල නම/admission no snapshot එකක් ලෙස රැඳී තිබේ (historical record එක නැති නොවේ), නමුත් අලුත් ලකුණු පත්‍රයක් සෑදීමට එම සිසුවා තවත් නොපෙනේ.
