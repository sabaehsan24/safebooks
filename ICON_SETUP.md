# App Icon Setup Instructions

The SafeBooks logo has been provided for use as the app icon throughout the project.

## iOS Icon Setup

iOS icons are located in: `ios/App/App/Assets.xcassets/AppIcon.appiconset/`

### Required iOS Icon Sizes:
- **1024x1024** (required for App Store)
- **512x512** (for watch OS)
- **180x180** (iPhone)
- **167x167** (iPad Pro)
- **152x152** (iPad)
- **120x120** (iPhone)
- **87x87** (watchOS)
- **80x80** (iPad, iPhone)
- **58x58** (watchOS)
- **40x40** (iPhone, iPad)
- **29x29** (iPhone, iPad)
- **16x16** (Mac)

Currently, only AppIcon-512@2x.png (1024x1024) is configured. 

To add more sizes:
1. Generate PNG files from the SafeBooks logo for each size listed above
2. Update `Contents.json` to include all icon sizes
3. Add the PNG files to the appiconset folder

## Android Icon Setup

Android icons are located in: `android/app/src/main/res/`

### Required Android Icon Sizes (by density):

| Density | Folder | Size | DPI |
|---------|--------|------|-----|
| ldpi | mipmap-ldpi | 36x36 | 120 |
| mdpi | mipmap-mdpi | 48x48 | 160 |
| hdpi | mipmap-hdpi | 72x72 | 240 |
| xhdpi | mipmap-xhdpi | 96x96 | 320 |
| xxhdpi | mipmap-xxhdpi | 144x144 | 480 |
| xxxhdpi | mipmap-xxxhdpi | 192x192 | 640 |
| anydpi | mipmap-anydpi-v26 | adaptive icons |  |

Create icon files named:
- `ic_launcher.png` (standard launcher icon)
- `ic_launcher_round.png` (round launcher icon, optional)
- Optionally add adaptive icon support with:
  - `ic_launcher_foreground.png`
  - `ic_launcher_background.png`

## Tools to Generate Icons

### Option 1: Using an Online Icon Generator
1. Visit https://www.appicon.co/ or https://makeappicon.com/
2. Upload the SafeBooks logo image
3. Download the generated icon package
4. Extract and place files in respective folders

### Option 2: Using ImageMagick
```bash
# Install ImageMagick
# Windows: choco install imagemagick
# macOS: brew install imagemagick
# Linux: apt-get install imagemagick

# Generate iOS icons
convert safeBooks-logo.png -resize 1024x1024 ios/App/App/Assets.xcassets/AppIcon.appiconset/AppIcon-512@2x.png
convert safeBooks-logo.png -resize 180x180 ios/App/App/Assets.xcassets/AppIcon.appiconset/AppIcon-180.png

# Generate Android icons
convert safeBooks-logo.png -resize 192x192 android/app/src/main/res/mipmap-xxxhdpi/ic_launcher.png
convert safeBooks-logo.png -resize 144x144 android/app/src/main/res/mipmap-xxhdpi/ic_launcher.png
```

### Option 3: Using Cordova Icon Generator
```bash
npm install -g cordova
cordova plugin add cordova-plugin-splashscreen

# Place your icon at: icon.png (1024x1024 or larger)
cordova-icon --icon=icon.png --platforms=ios,android
```

### Option 4: Manual with Design Software
1. Open the SafeBooks logo in Figma, Adobe XD, or Photoshop
2. Create an artboard for each required size
3. Export as PNG for each size
4. Place in corresponding folders

## Files to Update After Icon Installation

Once you generate and place the icons:

1. **iOS** - Update Contents.json in `ios/App/App/Assets.xcassets/AppIcon.appiconset/`
2. **Android** - No additional configuration needed if icons are named `ic_launcher.png`

## Recommended Next Steps

1. Generate all required icon sizes from the SafeBooks logo
2. Copy generated icons to their respective platform directories
3. Test on both iOS and Android by building and running the app
4. Verify icons appear correctly on home screen and in app settings

---

**Note:** The app name "SafeBókhald" has been updated throughout the project in:
- Android: `android/app/src/main/res/values/strings.xml`
- iOS: `ios/App/App/Info.plist`
- Capacitor configs: `capacitor.config.json` and `capacitor_config_json_ios`
- README.md documentation
