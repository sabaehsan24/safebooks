# Safe Books Mobile App - Capacitor Setup Guide

Complete guide for wrapping the Safe Books web app into native Android/iOS apps using Capacitor.

---

## Prerequisites

- Node.js installed
- Android Studio (for Android builds)
- Xcode + macOS (for iOS builds - optional)
- Your web app deployed and accessible via HTTPS

---

## Step 1: Create a New Capacitor Project

Create a new empty folder **outside** your web project:
```bash
mkdir safe-books-mobile
cd safe-books-mobile
npm init -y
npm install @capacitor/core @capacitor/cli
npx cap init
```

### During `npx cap init` prompts:

- **App name:** `Safe Books`
- **App ID:** `com.safebooks.app` (reverse domain format)
- **Web directory:** `dist` (required but not used for live web apps)
- **Create free Ionic account?** → Type `n` (not needed unless using Ionic cloud services)

---

## Step 2: Configure Capacitor to Use Live Web App

Edit `capacitor.config.ts`:
```typescript
import { CapacitorConfig } from '@capacitor/cli';

const config: CapacitorConfig = {
  appId: 'com.safebooks.app',
  appName: 'Safe Books',
  webDir: 'dist',
  server: {
    url: 'https://your-deployed-app.vercel.app', // Your live web app URL
    cleartext: true
  }
};

export default config;
```

✅ **This is the key part** - Capacitor will load your deployed web app, not local files.

---

## Step 3: Add Mobile Platforms
```bash
npx cap add android
npx cap add ios  # Only if you have macOS + Xcode
```

This creates:
- `android/` folder with Android Studio project
- `ios/` folder with Xcode project

---

## Step 4: Install Required Plugins

Install essential Capacitor plugins:
```bash
npm install @capacitor/status-bar
npm install @capacitor/assets
npx cap sync
```

---

## Step 5: Configure Status Bar & Safe Areas

### Install Status Bar Plugin

Already installed in Step 4, now configure it.

### A. Create Status Bar Setup Component

Create `src/components/CapacitorSetup.tsx`:
```tsx
'use client';

import { Capacitor } from '@capacitor/core';
import { StatusBar, Style } from '@capacitor/status-bar';
import { useEffect } from 'react';

export function CapacitorSetup() {
  useEffect(() => {
    if (Capacitor.isNativePlatform()) {
      const setupStatusBar = async () => {
        try {
          // Show the status bar
          await StatusBar.show();
          
          // Style.Light = Dark/black icons (for light backgrounds)
          // Style.Dark = White icons (for dark backgrounds)
          await StatusBar.setStyle({ style: Style.Light });
          
          // Set background color (works on Android 14 and below)
          await StatusBar.setBackgroundColor({ color: '#ffffff' });
          
          console.log('Status bar configured');
        } catch (error) {
          console.error('Status bar error:', error);
        }
      };

      setupStatusBar();
    }
  }, []);

  return null;
}
```

**Important - Status Bar Style Names:**
- ✅ `Style.Light` = **Dark/black icons** for light backgrounds
- ✅ `Style.Dark` = **White icons** for dark backgrounds
- ✅ `Style.Default` = Follows device theme (dark mode = white icons, light mode = black icons)

### B. Add to Your Root Layout

In `src/app/layout.tsx` or your main layout:
```tsx
import { CapacitorSetup } from '@/components/CapacitorSetup';

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        <CapacitorSetup />
        {children}
      </body>
    </html>
  );
}
```

### C. Add Safe Area CSS

Add to your global CSS file (e.g., `globals.css`):
```css
/* Add padding to account for safe areas (notches, status bar, etc.) */
body {
  padding-top: env(safe-area-inset-top);
  padding-bottom: env(safe-area-inset-bottom);
}

/* Fill the status bar area with matching background color */
body::before {
  content: '';
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  height: env(safe-area-inset-top, 0px);
  background: #ffffff; /* Match your app's background color */
  z-index: 50; /* Keep below modals but above content */
}

/* Adjust header position to account for status bar */
header {
  top: env(safe-area-inset-top, 0px) !important;
}

/* Fix drawer/sheet components to account for status bar */
[role='dialog'][class*='inset-y-0'] {
  top: env(safe-area-inset-top, 0px) !important;
  height: calc(100vh - env(safe-area-inset-top, 0px)) !important;
}

/* Fix close button position in drawer/sheet */
[role='dialog'] button[class*='absolute'][class*='top-4'] {
  top: 1rem !important; /* Keep original position, drawer is already offset */
}
```

### D. Color Coordination Guide

**For Light Backgrounds (white, light gray):**
```tsx
await StatusBar.setStyle({ style: Style.Light }); // Black icons
await StatusBar.setBackgroundColor({ color: '#ffffff' });
```
```css
body::before {
  background: #ffffff; /* Match status bar color */
}
```

**For Dark Backgrounds (dark gray, black):**
```tsx
await StatusBar.setStyle({ style: Style.Dark }); // White icons
await StatusBar.setBackgroundColor({ color: '#1a1f2e' });
```
```css
body::before {
  background: #1a1f2e; /* Match status bar color */
}
```

**Key Rule:** Status bar background color and `body::before` background must match for seamless appearance.

---

## Step 6: Generate App Icons & Splash Screens

### Prepare Assets

Create these files in your project root:
- `icon.png` - 1024x1024px (app icon)
- `splash.png` - 2732x2732px (splash screen)

### Generate Assets
```bash
npx capacitor-assets generate
```

⚠️ **Note:** The command is `capacitor-assets generate`, NOT `cap assets` (old command).

This will automatically generate:
- Android icons (all densities)
- iOS icons (all sizes)
- Splash screens for both platforms

---

## Step 7: Run & Build

### Android
```bash
npx cap open android
```

From Android Studio:
1. Select device/emulator
2. Click **Run** (green play button)
3. For release: **Build** → **Generate Signed Bundle/APK**

### iOS (macOS only)
```bash
npx cap open ios
```

From Xcode:
1. Select simulator/device
2. Click **Run** (▶️ button)
3. For release: **Product** → **Archive** → Submit to TestFlight/App Store

---

## Step 8: Sync Changes

Whenever you update plugins or configuration:
```bash
npx cap sync
```

This updates both Android and iOS projects with latest changes.

---

## Android 15+ Important Notes

⚠️ **Breaking Changes in Android 15+:**

Starting with Android 15 (API 35+), the following are **no longer available**:
- `setBackgroundColor()` - Status bar background is always transparent
- `setOverlaysWebView()` - Status bar always overlays the webview

**Solution:** Use CSS `body::before` to fill the safe area with your desired color (already included in Step 5C).

---

## Why This Approach is Best

✅ **Web code stays untouched** - No modifications to your existing web app  
✅ **No Firebase SSR issues** - App loads from live URL, not static files  
✅ **Auth & permissions work correctly** - Firebase Auth works like in browser  
✅ **Faster delivery** - No rebuilding app for UI changes  
✅ **Auto-updates** - Web app updates reflect immediately in mobile app  
✅ **Single codebase** - Maintain one web app for all platforms  

---

## Important Notes for Client

📱 **App requires internet connection** - Loads from live web URL  
🔄 **No App Store resubmission for UI fixes** - Only web deploy needed  
🔐 **Firebase Auth works exactly like browser** - Same authentication flow  
⚡ **Instant updates** - Changes to web app appear immediately in mobile app  

---

## Troubleshooting

### Status bar icons not visible
- Check if `Style.Light` matches background (light bg = Style.Light)
- Verify `body::before` color matches status bar color
- On Android 15+, `setBackgroundColor` won't work, rely on CSS only

### Drawer/Sheet positioning issues
- Ensure safe area CSS is applied (Step 5C)
- Check z-index conflicts with `body::before`
- Verify drawer has `inset-y-0` class for proper offset

### App not loading web content
- Verify `server.url` in `capacitor.config.ts` is correct
- Check if web app is accessible via HTTPS
- Run `npx cap sync` after config changes

### Permission errors
- Add required permissions in `AndroidManifest.xml` / `Info.plist`
- Request permissions using Capacitor plugins, not web APIs

---

## Next Steps (Optional)

📦 **Add offline fallback screen** - Show message when no internet  
🔗 **Setup deep linking** - Open specific pages from links/notifications  
📊 **Add analytics** - Track mobile app usage separately  
🔔 **Push notifications** - Engage users with notifications  
📱 **App Store optimization** - Prepare screenshots, descriptions, keywords  

---

## Store Submission Checklist

### Before Submission:
- [ ] App icon (1024x1024) designed and generated
- [ ] Splash screen created
- [ ] Privacy policy URL added
- [ ] App description written
- [ ] Screenshots prepared (multiple device sizes)
- [ ] Test on physical devices (Android & iOS)
- [ ] Version number set correctly
- [ ] Signing keys configured (Android)
- [ ] Provisioning profiles set (iOS)

### Android:
- [ ] Generate signed AAB (Android App Bundle)
- [ ] Fill Google Play Console listing
- [ ] Add feature graphic, screenshots
- [ ] Complete questionnaire

### iOS:
- [ ] Archive in Xcode
- [ ] Upload to TestFlight
- [ ] Add App Store metadata
- [ ] Submit for review

---

## Useful Commands Reference
```bash
# Sync changes to platforms
npx cap sync

# Open platform projects
npx cap open android
npx cap open ios

# Copy web assets
npx cap copy

# Update Capacitor
npm install @capacitor/core@latest @capacitor/cli@latest
npx cap sync

# Check Capacitor doctor
npx cap doctor
```

---

## Additional Resources

- [Capacitor Documentation](https://capacitorjs.com/docs)
- [Status Bar Plugin Docs](https://capacitorjs.com/docs/apis/status-bar)
- [Android Safe Areas Guide](https://developer.android.com/develop/ui/views/layout/edge-to-edge)
- [iOS Safe Areas Guide](https://developer.apple.com/design/human-interface-guidelines/layout)

---

**Setup complete! 🎉**

Your web app is now wrapped in a native mobile app shell and ready for deployment to app stores.