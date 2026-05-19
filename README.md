# BloodConnect (MVP)

Real-time blood donation matching app for emergencies.

## What this MVP includes

- **Native Android app**: Kotlin + Jetpack Compose
- **Firebase backend**: Phone OTP Auth, Firestore database, FCM push notifications
- **Two roles in one account**:
  - **Donor**: registers blood group, age, phone, current location, and **travel radius (km)**; can toggle **Available**.
  - **Receiver**: creates a blood request with blood group, units, hospital location, notes.
- **Matching & notifications**
  - When a receiver creates a request, a Firebase **Cloud Function** finds matching nearby donors and sends **push notifications**.
  - Notification opens the request in-app and provides a **Call receiver** button (uses dialer intent).

## Assumptions (changeable)

- “Connect” = **phone call** (one-tap open dialer), because it’s fastest in emergencies.
- Donor location is taken from the phone (with permission). Donor can refresh/update it.
- Radius is donor-defined (km) and used for matching.
- This MVP does **not** include admin verification, donation history validation, or in-app chat (can be added next).

---

## How to run (Android Studio)

> Firebase setup note:
> I can’t directly create/configure your Firebase project from here (it requires your account access),
> but the steps below are exactly what you need to do once in Firebase Console.

1. Create a Firebase project.
2. Add an Android app with package name:
   - `com.bloodconnect`
3. Download `google-services.json` and place it here:
   - `app/google-services.json`
4. In Firebase Console enable:
   - **Authentication → Phone**
   - **Firestore Database**
   - **Cloud Messaging**
5. Deploy Cloud Functions (for push matching):
   - From the `functions/` folder, run:
     - `npm install`
     - `firebase deploy --only functions`
6. Open the project in Android Studio and run on a device.

> Notes
> - Phone OTP requires a real device (emulators can work but are less reliable).
> - For notifications on Android 13+, the app will request the **POST_NOTIFICATIONS** permission.

## Build a debug APK (Android Studio)

- Android Studio → **Build** → **Build Bundle(s) / APK(s)** → **Build APK(s)**
- APK output is shown in the Build window; you can also find it under:
  - `app/build/outputs/apk/debug/`

---

## Firestore collections (schema)

### `users/{uid}`

```json
{
  "name": "Asha",
  "age": 23,
  "phoneNumber": "+911234567890",
  "bloodGroup": "O+",
  "isDonor": true,
  "isReceiver": true,
  "isAvailable": true,
  "radiusKm": 10,
  "location": { "lat": 12.9716, "lng": 77.5946 },
  "fcmTokens": ["token1", "token2"],
  "lastDonationAt": 1710000000000,
  "blockedUids": [],
  "updatedAt": 1710000000000
}
```

### `requests/{requestId}`

```json
{
  "createdByUid": "uid",
  "receiverPhoneNumber": "+911234567890",
  "bloodGroup": "O+",
  "unitsNeeded": 2,
  "hospitalName": "City Hospital",
  "hospitalLocation": { "lat": 12.97, "lng": 77.59 },
  "notes": "Accident emergency",
  "status": "OPEN",
  "createdAt": 1710000000000
}
```

---

## Next upgrades (recommended)

- Stronger donor cooldown rules (configurable days, server-enforced availability)
- In-app chat improvements (read receipts, attachments, moderation)
- Stronger verification (admin review, abuse detection, rate limits)
- Better geo-querying via geohash indexes (already scaffolded in the Cloud Function)
