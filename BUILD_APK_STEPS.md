# Build APK Steps

## Fixed Issues ✅
1. Fixed AndroidManifest.xml meta-data tag formatting
2. Added network security configuration
3. Configured proper HTTPS support

## Steps to Build APK

### 1. Clean and Sync
```bash
cd mobile

# Install dependencies (if needed)
npm install

# Build the Vue/Ionic app
npm run build

# Sync Capacitor
npx cap sync android

# Copy web assets
npx cap copy android
```

### 2. Open in Android Studio
```bash
npx cap open android
```

### 3. In Android Studio

#### Option A: Run on Device/Emulator (for testing)
1. Click the green "Run" button (or Shift+F10)
2. Select your device/emulator
3. Wait for build and installation

#### Option B: Build APK (for distribution)
1. Build → Clean Project
2. Build → Rebuild Project
3. Build → Generate Signed Bundle / APK
4. Select "APK"
5. Click "Next"
6. Create or select your keystore
7. Enter keystore password
8. Select build variant: "release"
9. Click "Finish"

### 4. Find Your APK
```
mobile/android/app/build/outputs/apk/release/app-release.apk
```

## Troubleshooting

### If build fails with Gradle errors:
1. File → Invalidate Caches → Invalidate and Restart
2. Build → Clean Project
3. Build → Rebuild Project

### If meta-data errors occur:
- Already fixed in AndroidManifest.xml
- Ensure all meta-data tags use self-closing format: `<meta-data ... />`

### If network/API doesn't work:
- Check network_security_config.xml exists
- Verify AndroidManifest references it
- Ensure INTERNET permission is present

## Testing the API

After installing APK:
1. Fill all required fields (marked with *)
2. Click SUBMIT
3. Should show loading spinner
4. Should show success modal if submission works
5. Should show error toast if network fails

## Production Checklist
- ✅ API endpoint configured
- ✅ Network security config added
- ✅ HTTPS enforced
- ✅ Internet permission granted
- ✅ Form validation working
- ✅ Error handling implemented
- ✅ La Salle green theme applied

