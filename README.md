# SafeBrowse

A privacy-first Android browser with a **multi-layer content filter** that
survives cache wipes and can't be bypassed from inside the browser.

> Browser thoda slow chale par filter bypass na ho — slow is OK, leaky is not.

## Features

- Hardened WebView with JS enabled but no file/content access, no DOM storage,
  no on-disk cache, cookies disabled, mixed content blocked.
- **9 filter layers** running in sequence. A bypass has to defeat every layer
  that's enabled.
- **Block list** — exact domains or `*.example.com` wildcards.
- **White list** — always wins over block list and categories.
- **Categories** — adult / porn, social media, gaming, gambling, video
  streaming, dating, news, shopping, drugs, weapons. Toggle per category.
- **Per-site timers** — allow a site only between two times of day, optionally
  on selected weekdays, with optional daily quota in seconds.
- **On-device NSFW image scoring** — TFLite MobileNet over every image the
  WebView tries to render.
- **HTML content keyword scan** — strip script/style, sliding-window keyword
  match in the rendered text.
- **DNS over HTTPS** filter via Cloudflare for Families (1.1.1.3).
- **System-wide VPN sinkhole** (opt-in) — captures DNS queries from any app on
  the device and refuses to resolve blocked hosts.
- **SafeSearch / Restricted Mode** forced on Google, Bing, DuckDuckGo, YouTube.
- **Google Sign-In** with end-to-end encrypted Firestore sync of all settings.
- **Boot-time watchdog service** restarts after device reboot so the user
  cannot defeat the filter by rebooting.
- **Accessibility watchdog** logs attempts to open another browser app.

## Why multi-layer?

A single filter (e.g. just a host blocklist) is trivially bypassed: clear app
cache, use a different DNS, type the IP, use a shortener, use a web proxy,
open the URL in another browser, etc. We run nine independent layers so the
attacker has to defeat every one of them. See `docs/FILTER_LAYERS.md`.

## Architecture

See `docs/ARCHITECTURE.md` for the full module map, the data flow, and the
cryptographic model.

## Quick start (developer)

1. Install Android Studio Hedgehog (or newer).
2. Create a Firebase project and add an Android app with the package
   `com.safebrowse.app`.
3. Copy the generated `google-services.json` into `app/`.
4. Enable Google Sign-In in the Firebase console (Authentication → Sign-In
   Method → Google).
5. Enable Firestore in Native mode.
6. Open the `SafeBrowse` folder in Android Studio, sync Gradle, run.

A complete setup walk-through is in `docs/SETUP.md`.

## Build APK

```bash
cd SafeBrowse
./gradlew :app:assembleRelease
```

For the release APK you also need a release keystore. Put it in
`local.properties`:

```properties
signing.keystore=/absolute/path/keystore.jks
signing.storePassword=...
signing.keyAlias=...
signing.keyPassword=...
```

The release build also expects a `google-services.json` (not bundled here).
Drop it into `app/google-services.json` before building.

## Tests

```bash
./gradlew :app:test
```

Unit tests cover URL pattern matching, wildcard rules, category lookup, timer
window evaluation (including wrap-around midnight), keyword scanning, and
filter precedence.

## License

Source provided under MIT. The bundled NSFW model weights (TFLite) come from
the yahoo/open_nsfw project — Apache 2.0.
