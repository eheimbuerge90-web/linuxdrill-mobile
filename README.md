# Linux Drill — Mobile

Android app (Capacitor + Vite + React + TypeScript + Tailwind 4) version of the [Linux Drill](https://github.com/eheimbuerge90-web/htmlquizz) web quiz.

## Layout

- `src/` — React app (UI + logic). `App.tsx` is the quiz screen; `data/questions.ts` is the bank.
- `android/` — Generated Capacitor Android project. Edit only what you intend to ship.
- `capacitor.config.ts` — App id (`com.heimburge.linuxdrill`) and display name.
- `.github/workflows/android.yml` — CI that builds the APK on every push.

## Local development (web preview)

```bash
npm install
npm run dev            # Vite dev server at http://localhost:5173
npm run build          # Production web bundle in dist/
```

The web build is what Capacitor packs into the APK.

## Android build

Builds happen in GitHub Actions on `ubuntu-latest` (x86_64 Linux). Local APK builds on Asahi (16K-page aarch64) hit Android build-tools binary-compat issues and are not supported here.

### Get a debug APK

1. Push a commit (or open a PR), or trigger the workflow from the Actions tab via **Run workflow**.
2. When the run finishes, download `linuxdrill-debug-apk` from the run's **Artifacts**.
3. Install on a device: `adb install linuxdrill-debug.apk`.

### Get an unsigned release AAB (for Play Store upload)

1. Go to **Actions → Android build → Run workflow** (manual trigger only).
2. Download `linuxdrill-release-aab-unsigned`.
3. Sign locally (see next section) before uploading to the Play Console.

## Signing for Play Store release

Play Store requires a release-signed `.aab`. **Do not commit the keystore** — generate one and keep it safe (lose it = lose the app).

### Generate a keystore (one-time)

```bash
keytool -genkey -v \
  -keystore linuxdrill-release.keystore \
  -alias linuxdrill \
  -keyalg RSA -keysize 2048 -validity 10000
```

You will be asked for a keystore password, alias password, and identity info. Save the passwords in a password manager.

### Sign the AAB

```bash
jarsigner -verbose -sigalg SHA256withRSA -digestalg SHA-256 \
  -keystore linuxdrill-release.keystore \
  linuxdrill-release.aab linuxdrill
```

Or configure signing in `android/app/build.gradle` and rebuild — see Capacitor docs.

### CI-side signing (recommended later)

Store the keystore base64-encoded in `secrets.ANDROID_KEYSTORE` and the passwords in `secrets.KEYSTORE_PASSWORD` / `secrets.KEY_PASSWORD`. Decode + use in the workflow before `bundleRelease`. Not wired up yet — kept simple while iterating.

## Updating questions

Questions live in `src/data/questions.ts`, an independent copy of the web project's file. To pull in updates from `htmlquizz`:

```bash
cp ../htmlquizz/app/data/questions.ts src/data/questions.ts
```

Then commit and push — CI will produce a fresh APK.

## Bumping the version

Edit `android/app/build.gradle`:

```gradle
versionCode 2          # increment for every Play Store upload
versionName "1.0.1"    # human-readable
```

Play Store rejects an upload with a `versionCode` already used.
