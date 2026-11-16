# رادار العائلة — دليل الإعداد

## 1) Firebase
1. console.firebase.google.com → Add project → Build → **Realtime Database** → Create.
2. Build → **Authentication** → Sign-in method → فعّل **Anonymous**.
3. Project settings → Your apps → Web (</>) → انسخ `firebaseConfig` إلى أعلى `index.html`.
4. Realtime Database → Rules (لا قراءة للجذر، ولا يصل إلا من يعرف الهاش):
```json
{ "rules": { "families": { "$hash": {
  ".read": "auth != null", ".write": "auth != null",
  "members": { "$uid": { ".validate": "newData.hasChildren(['name'])" } } } } } }
```
> الأمان يعتمد على أن الرمز السري طويل وغير قابل للتخمين (SHA-256 يُحسب على الجهاز، والرمز نفسه لا يُرسل).

## 2) الاستضافة (HTTPS إلزامي للموقع وSW)
`firebase init hosting` ثم `firebase deploy` (أو Netlify / GitHub Pages).

## 3) APK
**الأسرع (TWA):** افتح pwabuilder.com ← أدخل الرابط ← Android ← نزّل الحزمة. (يلزم تحويل icon.svg إلى PNG 512×512 و192×192.)

**الأفضل للخلفية (Capacitor):**
```bash
npm init -y && npm i @capacitor/core @capacitor/cli @capacitor/android
npx cap init FamilyRadar com.family.radar --web-dir=www
mkdir www && cp index.html manifest.json sw.js icon.svg www/
npm i @capacitor-community/background-geolocation
npx cap add android && npx cap sync && npx cap open android   # Build > Build APK
```
في `AndroidManifest.xml` أضف: `ACCESS_FINE_LOCATION`, `ACCESS_BACKGROUND_LOCATION`, `FOREGROUND_SERVICE`, `FOREGROUND_SERVICE_LOCATION`, `POST_NOTIFICATIONS`.
ثم استبدل `navigator.geolocation.watchPosition` بـ `BackgroundGeolocation.addWatcher({backgroundMessage:"التتبع يعمل", requestPermissions:true, distanceFilter:20}, (loc)=>push({coords:loc}))` — هذا يشغّل Foreground Service يبقي التتبع حياً مع إغلاق الشاشة.

## حدود الخلفية في المتصفح/PWA
المتصفح يوقف JavaScript عند إخفاء الصفحة؛ لا حل حقيقي بدون غلاف Capacitor أعلاه. في PWA: يُستخدم Wake Lock (الشاشة مفتوحة)، ويُرسل آخر موقع عند الإخفاء. على الجهاز: عطّل "توفير البطارية" للتطبيق وفعّل الموقع "السماح دائماً".
