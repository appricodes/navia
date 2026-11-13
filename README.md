# Navia — a touchable offline map for blind and low-vision people

Navia is an Android app that turns the phone screen into a map you explore with your finger. Slide a finger across the screen and Navia speaks the street, shop or place under it: its name, what it is, and how far away it is, with directions given as clock positions ("2 o'clock, 40 meters"). It is built around TalkBack from the start.

[Get it on Google Play](https://play.google.com/store/apps/details?id=com.wikiponder.touchmap)

## Features

- **Touch exploration**: streets, house numbers, shops, bus stops, crossings, benches and more are announced as your finger passes over them. Points take priority over roads so small things are easy to find.
- **Aim**: pick any place and get live distance and direction updates while walking toward it.
- **Offline**: once your area is downloaded, everything works without a network connection.
- **AI place recognition (experimental)**: film a familiar spot (such as your front door) for a few seconds; later, point the camera ahead and Navia tells you when it sees it and how far to turn. Runs fully on-device.
- **Sound simulation**: hear a 3D (HRTF) soundscape of any point on the map (traffic, trams, parks, water), built from the surrounding map data.
- **Make it yours**: add your own points, rename places, save favourites, and filter announcements by category or keyword.
- **Voice input**: search, add and rename by dictation.
- **Low vision support**: an optional drawn map with distinct marker shapes per category, street labels, and light/dark themes.
- **47 languages**, no ads, no account, no tracking.

> Navia is an extra aid only. Always use a white cane, guide dog or another safe method first. GPS, map data and software can be wrong.

## How it works

- **Map data** comes from [OpenStreetMap](https://www.openstreetmap.org/). Instead of parsing OSM extracts on the device, the app downloads pre-built SQLite "box" databases for the user's area from a server manifest (`classes/boxes`, `classes/importer/BoxImporter`). Maps can be updated when a new map version is published.
- **Place search** uses a GeoNames-based `world_places.db`, downloaded on first run (`classes/worldplaces`).
- **Rendering** (`classes/ui/MapView`) uses a cached map-space path layer drawn off the UI thread, with screen-space labels.
- **Announcements and live guidance** run in `BackgroundService` with `AnnouncementManager`, `AngleManager` and a Kalman-filtered location (`classes/location`).
- **Sound simulation** (`classes/soundscape`) renders a clip bank through [OpenAL Soft](https://github.com/kcat/openal-soft) with HRTF (native code in `app/src/main/cpp`, included as a git submodule).
- **Place recognition** uses an int8-quantised MegaLoc model run with ONNX Runtime (Navia flavor only). The model is downloaded the first time the feature is opened.

## Project layout

```
app/
  src/main/java/com/wikiponder/touchmap/   Activities, services and core classes
  src/main/cpp/                            Native audio (OpenAL Soft submodule + JNI)
  src/navia/                               Navia-flavor code (place recognition)
  src/test, src/androidTest*               Unit and on-device tests (incl. TalkBack smoke tests)
docs/                                      Design notes (place recognition, soundscape rules, ideas)
tools/                                     Python/shell tools: map box builder, world places DB,
                                           soundscape bank generation, VPR model export/quantisation
```

## Building

Requirements: Android Studio (recent), JDK 17+, Android SDK 36, and the NDK/CMake components (for the native audio code). The app runs on Android 8.0 (API 26) and later.

```bash
git clone --recurse-submodules https://github.com/appricodes/navia.git
cd navia
./gradlew assembleNaviaDebug
```

If you cloned without submodules, run `git submodule update --init` first.

Files that are not in the repository and must be supplied locally:

- `app/google-services.json`: Firebase configuration (used for opt-in crash reporting). Create your own Firebase project and add an Android app with id `com.wikiponder.touchmap` to get one.
- `keystore.properties`: release signing. Only needed for signed release builds; see `keystore.properties.example`.

The release bundle for Google Play is built with `./gradlew bundleNaviaRelease`.

## Feedback

Bug reports and suggestions are welcome via GitHub issues. Feedback from blind and low-vision users is especially valuable.

## Acknowledgements

- Map data © [OpenStreetMap contributors](https://www.openstreetmap.org/copyright), available under the ODbL.
- Place names from [GeoNames](https://www.geonames.org/) (CC BY 4.0).
- [OpenAL Soft](https://github.com/kcat/openal-soft) for 3D audio, [ONNX Runtime](https://onnxruntime.ai/) for on-device inference, and the [MegaLoc](https://github.com/gmberton/MegaLoc) place-recognition model.
