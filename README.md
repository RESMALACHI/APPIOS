# APPIOS

אפליקציית iOS שנבנתה עם **Expo** (SDK 57, TypeScript) ונפרסת ל-**TestFlight** באמצעות **EAS Build** ו-**EAS Submit**.
לא צריך Mac או Xcode, כי הבנייה רצה בענן של Expo.

## הרצה מקומית

```bash
npm install
npx expo start
```

סרקו את קוד ה-QR עם אפליקציית **Expo Go** באייפון.

## פריסה ל-TestFlight

### דרישות מוקדמות (פעם אחת)

1. **Apple Developer Program**: חשבון בתשלום (99$ לשנה) ב-https://developer.apple.com/programs/
2. **חשבון Expo**: חינמי ב-https://expo.dev/signup
3. **Bundle Identifier**: ב-`app.json` מוגדר `com.resmalachi.appios`. המזהה חייב להיות ייחודי בכל ה-App Store, ואי אפשר לשנות אותו אחרי ההעלאה הראשונה. אם רוצים מזהה אחר, צריך לשנות אותו לפני ההעלאה.

### פריסה ראשונה (מהמחשב, אינטראקטיבית)

```bash
npm install
npx eas-cli@latest login          # התחברות לחשבון Expo
npx eas-cli@latest init           # מקשר את הפרויקט ל-Expo ומוסיף projectId ל-app.json
npx eas-cli@latest build --platform ios --profile production --auto-submit
```

בזמן הפקודה האחרונה, EAS:
- מבקש להתחבר עם ה-Apple ID ויוצר לבד את תעודת ההפצה (Distribution Certificate) ואת ה-Provisioning Profile
- מקים את האפליקציה ב-App Store Connect ומפיק מפתח App Store Connect API
- בונה את קובץ ה-`.ipa` בענן ומעלה אותו ל-TestFlight

אחרי העלאה, Apple מעבדת את ה-build במשך 5 עד 30 דקות. אחר כך הוא מופיע ב-App Store Connect ← האפליקציה ← **TestFlight**.

> אחרי `eas init` צריך לבצע commit לשינוי ב-`app.json` (השדה `extra.eas.projectId`).

### הוספת בודקים

ב-App Store Connect ← TestFlight:
- **Internal Testing**: עד 100 חברי צוות, זמינים מיד
- **External Testing**: עד 10,000 בודקים דרך מייל או קישור ציבורי. ה-build הראשון עובר Beta App Review קצר

הבודקים מתקינים את האפליקציה **TestFlight** מה-App Store ומקבלים ממנה את הגרסה.

### פריסה אוטומטית מ-GitHub Actions

אחרי שהפריסה הראשונה הצליחה:

1. יוצרים טוקן ב-https://expo.dev/accounts/[account]/settings/access-tokens
2. מוסיפים אותו כ-Secret בשם `EXPO_TOKEN` ב-GitHub: Settings ← Secrets and variables ← Actions
3. מוסיפים ל-`eas.json` את מזהה האפליקציה מ-App Store Connect (App Information ← Apple ID), כי submit לא-אינטראקטיבי צריך אותו:
   ```json
   "submit": { "production": { "ios": { "ascAppId": "1234567890" } } }
   ```
4. מפעילים את ה-workflow **iOS → TestFlight** ידנית מלשונית Actions, או דוחפים תגית:
   ```bash
   git tag v1.0.0 && git push origin v1.0.0
   ```

מספר ה-build עולה אוטומטית בכל בנייה (`autoIncrement` + `appVersionSource: remote`). את גרסת האפליקציה (`version` ב-`app.json`) מעדכנים ידנית כשרוצים גרסה חדשה.

## מבנה

| קובץ | תפקיד |
|------|--------|
| `App.tsx` | מסך הפתיחה |
| `app.json` | הגדרות האפליקציה: שם, Bundle ID, אייקון |
| `eas.json` | פרופילי בנייה והגשה של EAS |
| `.github/workflows/testflight.yml` | בנייה והעלאה ל-TestFlight |
| `.github/workflows/ci.yml` | בדיקת טיפוסים בכל push |
