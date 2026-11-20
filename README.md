 tvhub

A small family of Android apps for TVs, projectors and phones. Built and tested on a 1 GB RAM projector, a 4 GB Android TV box and a phone with Android 16. Apps are developed directly on a phone with *Code On The Go*.

**Languages:** Русский · English · Қазақша · Українська · Türkçe · Français (Settings → Language / Язык / Тіл)

| App | Package | What it does |
|---|---|---|
| **Launcher** (Лаунчер) | `com.example.tvlauncher` | Android TV style home screen: rows of apps (1–4 lines per row), wallpapers, spectrometer, built-in IPTV player, screensaver, sleep timer, one-tap memory clean. |
| **Pult** (Пульт) | `com.example.pult` | Phone as a remote: trackpad, keyboard, TV-remote pad, screen mirroring both ways between phone, Android TV and PC. |
| **Peredacha** (Передача) | `com.example.peredacha` | Send files and text between devices over Wi‑Fi or Bluetooth, even without a router (phone hotspot works). |

## Install

1. Download the APKs from the latest [Release](../../releases/latest): `Launcher.apk`, `Pult.apk`, `Peredacha.apk`.
2. Allow *Install unknown apps* for your browser / file manager and install.
3. Launcher: Settings → *Home screen* → make it the default. On projectors that ignore it:
   `adb shell cmd package set-home-activity com.example.tvlauncher/.MainActivity`
4. Pult / Launcher: enable the accessibility service when asked (needed for remote control and the Home button).

## Updates over the internet

All three apps can update themselves. They read `updates.json` from this repository:

```json
{"apps":[
 {"pkg":"com.example.pult","code":12,"name":"2.0","url":"https://github.com/<user>/tvhub/releases/latest/download/Pult.apk","notes":""}
]}
```

* Launcher: Settings → *Updates*. Pult: Settings → *Updates*. Peredacha: button at the bottom.
* To publish a new version: upload new APKs to a Release, raise `code` in `updates.json`.
* The update must be signed with the same key as the installed app.

## Pult on a PC

```
pip install pynput
python3 pult_pc.py
```

`ffmpeg` / `ffplay` must be in `PATH` for screen mirroring. The program prints a PIN and an address; in Pult choose *Control a computer*.

## Build

Android Studio or *Code On The Go*, Gradle, Kotlin, no external libraries. Each app is a separate Gradle project (`app/`).

## Translations

All UI strings in the code are Russian: `L.t("Текст")`. Translations live in `app/src/main/assets/lang/<code>.json` as `{"Русский текст": "Translation"}`. To add a language, add a JSON file and a code to `L.CODES` / `L.NAMES`.

## Status

Hobby project, code is shared as is. Bug reports are welcome: in each app open *Errors and log* (Pult) and attach the text.

---

# tvhub (по-русски)

Семейство приложений для Android: **Лаунчер** (главный экран для ТВ и проекторов с IPTV), **Пульт** (телефон как пульт, трекпад, клавиатура, показ экрана) и **Передача** (файлы и текст между устройствами по Wi‑Fi и Bluetooth). Языки: русский, English, Қазақша, Українська, Türkçe, Français.

Скачайте APK из последнего релиза, разрешите установку из неизвестных источников, включите службу «Специальные возможности» для Пульта и Лаунчера. Обновление по интернету: Настройка → Обновления.
