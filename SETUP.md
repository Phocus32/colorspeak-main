# Getting ColorSpeak running

The Dart app logic was already complete: 18 named colors (red, orange, yellow,
lime, green, teal, cyan, blue, navy, purple, magenta, pink, brown, maroon,
olive, white, gray, black), live camera color sampling, speech announcements,
and vibration feedback. What was missing was the native Android/iOS project
wrapper, which I've now added under `android/` and (partially) `ios/`.

## 1. Install dependencies
```
flutter pub get
```

## 2. Android — ready to run
```
flutter run
```
The manifest already declares `CAMERA` and `VIBRATE` permissions.
`minSdk` is set to 23 (Android 6+), which the `camera` and `vibration`
plugins require.

## 3. iOS — one extra step
I added `ios/Runner/Info.plist` with the required
`NSCameraUsageDescription`, but a full Xcode project (`Runner.xcodeproj`)
is a generated binary-ish project file I can't safely hand-write. Generate
it once with:
```
flutter create --platforms=ios .
```
This will create the Xcode project *without* touching your existing
`lib/`, `pubspec.yaml`, or the Info.plist content — Flutter merges into
existing folders. After that, `flutter run` on a physical iPhone works
(the simulator has no camera).

## 4. Test on a real device
Camera + vibration only work on physical hardware, not emulators/simulators
(most emulators have no vibration motor and a fake camera feed).

## How detection works
- `ColorDetectionService` samples a 60×60 pixel box at the center of each
  camera frame, converts YUV/BGRA to RGB, and averages it.
- `ColorDatabase.findNearest` matches the averaged RGB to the closest of
  18 named colors using a luminance-weighted distance.
- When the matched color name changes, `HomeScreen` triggers
  `HapticService.colorDetected()` (vibration) and `SpeechService.speak()`
  (TTS), then re-announces the same color every 4 seconds if it's still
  in view.

## Adding more colors
Add entries to `ColorDatabase.colors` in `lib/models/color_match.dart` —
each just needs a name, an RGB reference value, and a display `Color`.
