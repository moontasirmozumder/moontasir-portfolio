# Bus Tracker — Native Android App

Kotlin native app, feature-parity with the web version (script.js + index.html):
live GPS marker on an OpenStreetMap view, follow/center buttons, and an info
card (status, bus ID, speed, lat/lng, last update) synced from Firebase
Realtime Database in real time.

## Setup (do this before building)

1. **Add `google-services.json`**
   - Go to Firebase Console → your project (`real-time-bus-tracker-1c9b8`) →
     Project settings → Add app → Android.
   - Package name: `com.moontasir.bustracker`
   - Download `google-services.json` and place it at:
     `app/google-services.json`
   - This file is required — the project won't build without it.

2. **Open in Android Studio**
   - Open the `BusTracker/` folder as a project.
   - Let Gradle sync (it will download osmdroid, Firebase, etc. — needs internet).

3. **Firebase Database Rules**
   - Same warning as before: if your Realtime Database rules are still
     `.read: true, .write: true` (test mode), lock them down before shipping
     this app publicly, since the app will be reading the same `/bus` node.

4. **Run**
   - Connect a device/emulator with internet access and hit Run.
   - The map centers on Dhaka by default, then jumps to the bus location
     once Firebase sends the first valid `latitude`/`longitude`.

## Structure

```
app/src/main/java/com/moontasir/bustracker/
  MainActivity.kt   → map setup, Firebase listener, follow/center logic
  BusData.kt        → data model matching the Firebase /bus JSON node
app/src/main/res/
  layout/activity_main.xml  → info card + map + buttons (matches web UI)
  values/colors.xml         → same blue theme as style.css (#2563eb)
  drawable/ic_bus_marker.xml → vector bus icon for the map marker
```

## Getting a ready-to-install APK (no Android Studio needed)

This project includes a GitHub Actions workflow (`.github/workflows/build.yml`)
that builds the APK automatically on GitHub's servers every time you push.

1. **Create a GitHub repo** (private is fine) and push this whole folder:
   ```bash
   cd BusTracker
   git init
   git add .
   git commit -m "Initial commit"
   git branch -M main
   git remote add origin https://github.com/<your-username>/<repo-name>.git
   git push -u origin main
   ```

2. **Add `google-services.json` before pushing** (see step 1 above) — put it
   at `app/google-services.json`. It's client-side config (same category as
   the Firebase web API key already in your project), so it's fine to commit
   to a private repo.

3. **Watch it build**: go to your repo → **Actions** tab → you'll see
   "Build APK" running. Takes ~2-3 minutes.

4. **Download the APK**: once it finishes (green checkmark), click into that
   run → scroll to **Artifacts** → download `BusTracker-debug-apk` (a zip
   containing `app-debug.apk`).

5. **Install on your phone**: transfer the APK to your phone (or upload it to
   your cPanel and download via browser), enable "Install from unknown
   sources" if prompted, and install.

Every time you push new changes to `main`, a fresh APK builds automatically —
no local Android Studio setup required.

## Notes / things to double check

- `lastUpdate` is read as a `Double` in `BusData.kt` because the ESP32 writes
  it as a numeric epoch-millis value — this avoids a Firebase deserialization
  crash that happens if you declare it as `Long` directly.
- osmdroid caches map tiles on device storage; the `WRITE_EXTERNAL_STORAGE`
  permission is capped at `maxSdkVersion=28` since newer Android versions
  don't need it (scoped storage handles this automatically).
- No Google Maps API key needed — this uses the same OpenStreetMap tile
  source as your web app.
- If you later want push notifications for offline/online status changes,
  that would need Firebase Cloud Messaging added on top of this — let me
  know if you want that.
