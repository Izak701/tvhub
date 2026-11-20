# Setup guide: ADB, permissions, accessibility / Подробная инструкция

English first, русская версия ниже ([jump / перейти](#русская-версия)).

Package names used below:

| App | Package |
|---|---|
| Launcher | `com.example.tvlauncher` |
| Pult (remote) | `com.example.pult` |
| Peredacha (transfer) | `com.example.peredacha` |

---

## 0. What you need ADB for (and what you don't)

Most things are done inside the apps. ADB is only needed for:

1. making the Launcher the permanent Home app without the "Always / Once" chooser;
2. enabling an Accessibility service when your TV firmware has no Accessibility screen;
3. (optional) granting permissions from a computer.

If the Launcher already starts when you press Home, you do not need ADB at all.

## 1. Get ADB

### Option A: from a PC (Windows / macOS / Linux)
1. Download **Android SDK Platform-Tools** from developer.android.com/tools/releases/platform-tools and unzip.
2. Windows: open the folder, click the address bar, type `cmd`, Enter. macOS/Linux: open a terminal in the folder (use `./adb` instead of `adb`).
3. Check: `adb version`.

### Option B: from another Android phone (no PC)
1. Install **Termux** (from F-Droid) and run `pkg install android-tools`.
2. Use the same `adb` commands.

## 2. Turn on debugging on the TV / projector

1. **Settings → About** (or *Device info*) → tap **Build number** 7 times → "You are now a developer".
2. **Settings → Developer options** → turn on **USB debugging**. If you see **Network / Wireless debugging**, turn that on instead.
3. Connect the PC and the TV to the **same Wi-Fi**.

### Connect over Wi-Fi
- Find the TV IP: **Settings → Network → your Wi-Fi → IP address** (example `192.168.1.50`).
- Classic network debugging (port 5555):
  ```
  adb connect 192.168.1.50:5555
  ```
- Android 11+ "Wireless debugging" with pairing code:
  ```
  adb pair 192.168.1.50:PAIRING_PORT      (enter the 6-digit code)
  adb connect 192.168.1.50:CONNECT_PORT
  ```
  The two ports are shown on the Wireless debugging screen and differ.
- The TV shows **"Allow USB debugging?"** → tick *Always allow* → **OK**.
- Check: `adb devices` must list the device as `device` (not `unauthorized`).

## 3. Make the Launcher the Home app

Command (works on most devices, the chooser never appears again):

```
adb shell cmd package set-home-activity com.example.tvlauncher/.MainActivity
```

Inside the Launcher the same command is in **Settings → Make default launcher** (it can be copied there).

If it says an error:
- Install the Launcher first and open it once.
- Make sure only one device is connected (`adb devices`).
- Some firmwares block it: use **Settings → Apps → Default apps → Home app** instead.

To undo: **Settings → Apps → Default apps → Home app** and choose the old one, or `adb shell pm clear com.example.tvlauncher`.

## 4. Accessibility services

The apps use accessibility for:
- **Launcher**: fully closing apps ("Free memory" presses *Force stop* in Android settings) and restarting itself after boot.
- **Pult**: moving the focus on the TV with the phone remote.

### 4.1 Normal way (on-screen)
**Settings → Accessibility → Installed / Downloaded services → Launcher (or Pult) → turn On → Allow.**

### 4.2 Android 13+: "Restricted setting" / the switch is grey
For apps installed from a file (not from a store) Android blocks the switch:
1. **Settings → Apps → [Launcher / Pult] → ⋮ (top right) → Allow restricted settings.**
2. Then repeat 4.1.
(If there is no ⋮, first try to turn the service on once, then the menu appears.)

### 4.3 If your device has no Accessibility screen: use ADB
First see what is already enabled:
```
adb shell settings get secure enabled_accessibility_services
```
- Output `null` or empty → nothing enabled yet. Enable the Pult service:
  ```
  adb shell settings put secure enabled_accessibility_services com.example.pult/com.example.pult.PultAccessibility
  adb shell settings put secure accessibility_enabled 1
  ```
- Output has something → **append** with a colon, otherwise you switch the others off:
  ```
  adb shell settings put secure enabled_accessibility_services "OLD_VALUE:com.example.pult/com.example.pult.PultAccessibility"
  ```
  (paste the old output instead of `OLD_VALUE`).

Safe one-liner that does both cases (Linux/macOS/Termux):
```
adb shell 'cur=$(settings get secure enabled_accessibility_services); case "$cur" in ""|null) new=com.example.pult/com.example.pult.PultAccessibility;; *) new="$cur:com.example.pult/com.example.pult.PultAccessibility";; esac; settings put secure enabled_accessibility_services "$new"; settings put secure accessibility_enabled 1'
```
For the Launcher service replace the part `com.example.pult/com.example.pult.PultAccessibility` with `com.example.tvlauncher/com.example.tvlauncher.LauncherAccessibilityService`.

Windows `cmd`: run the first two commands separately (the one-liner uses single quotes).

To switch off: `adb shell settings put secure enabled_accessibility_services null` (turns off **all** services).

## 5. Other permissions

| What | Where |
|---|---|
| Install updates from the internet | When asked: **Allow from this source**. Or **Settings → Apps → [app] → Install unknown apps** |
| Launcher autostart after boot | In Launcher: **Settings → Launcher autostart**. Also enable the Accessibility service (4.1). On some firmwares also allow *Autostart* in the system app list. |
| Show over other apps | **Settings → Apps → Special access → Display over other apps → Launcher** |
| Notifications (Pult) | **Settings → Apps → Pult → Notifications → Allow** |
| Storage / photos (Peredacha) | Allow when asked |

## 6. Updates over the internet

1. Open any app → **Check updates** (or just start it: it asks every launch).
2. Press **Update** to install, **Later** to skip.
3. The new APK must be signed with the **same key** as the installed one. If Android says "app not installed" you installed a build signed differently: uninstall once and install the new one.

## 7. Pult on a PC

See the main README. Short version: run `Run_Pult.bat` (Windows) or `python3 pult_pc.py`; it installs `pynput` automatically; phone and PC on the same Wi-Fi; the port is `47474`.

## 8. Troubleshooting

| Problem | Fix |
|---|---|
| `adb: command not found` | Open the terminal in the platform-tools folder, or add it to PATH |
| `unauthorized` | Look at the TV screen, accept the dialog; if absent: Developer options → *Revoke USB debugging authorizations*, reconnect |
| `more than one device/emulator` | `adb -s 192.168.1.50:5555 shell ...` |
| `cannot connect` | Same Wi-Fi? Wireless debugging on? Reboot the TV and try again; IP could have changed |
| Service turns itself off | Battery optimisation: **Settings → Apps → [app] → Battery → Unrestricted** |
| "Free memory" does not close apps | The Launcher accessibility service is off (4.1), or the firmware has no *Force stop* button |

---

# Русская версия

Имена пакетов:

| Приложение | Пакет |
|---|---|
| Лаунчер | `com.example.tvlauncher` |
| Пульт | `com.example.pult` |
| Передача | `com.example.peredacha` |

## 0. Когда нужен ADB

Почти всё делается внутри приложений. ADB нужен только чтобы:
1. сделать Лаунчер главным экраном без окна «Всегда / Только сейчас»;
2. включить службу специальных возможностей, если в прошивке нет такого экрана;
3. (по желанию) выдать разрешения с компьютера.

Если по кнопке «Домой» уже открывается Лаунчер, ADB не нужен.

## 1. Как получить ADB

**С компьютера:** скачайте *Platform-Tools* с developer.android.com/tools/releases/platform-tools, распакуйте. В Windows откройте папку, в адресной строке напишите `cmd`, Enter. Проверка: `adb version`.

**С другого телефона:** установите Termux (из F-Droid), выполните `pkg install android-tools`.

## 2. Включить отладку на проекторе / приставке

1. **Настройки → О устройстве** → 7 раз нажмите **Номер сборки**.
2. **Настройки → Для разработчиков** → включите **Отладка по USB** (или **Отладка по сети / Беспроводная отладка**).
3. ПК и ТВ в **одной Wi-Fi сети**.

**Подключение по Wi-Fi:**
- IP ТВ: **Настройки → Сеть → ваш Wi-Fi** (например `192.168.1.50`).
- Обычный способ: `adb connect 192.168.1.50:5555`
- Android 11+ «Беспроводная отладка»: `adb pair 192.168.1.50:ПОРТ_СОПРЯЖЕНИЯ` (введите 6-значный код), затем `adb connect 192.168.1.50:ПОРТ_ПОДКЛЮЧЕНИЯ`. Порты показаны на экране, они разные.
- На ТВ появится «Разрешить отладку?» → галочка «Всегда» → **OK**.
- Проверка: `adb devices` — устройство должно быть `device`, а не `unauthorized`.

## 3. Сделать Лаунчер главным

```
adb shell cmd package set-home-activity com.example.tvlauncher/.MainActivity
```

Эта же команда есть в Лаунчере: **Настройки → Сделать главным** (её можно скопировать).

Если ошибка: сначала установите и один раз откройте Лаунчер; проверьте, что подключено одно устройство; либо через **Настройки → Приложения → Приложения по умолчанию → Главный экран**.

Отменить: выбрать прежний лаунчер в тех же настройках.

## 4. Специальные возможности (службы)

Зачем:
- **Лаунчер**: полностью закрывать приложения («Освободить память» нажимает *Остановить* в настройках Android) и запускаться после включения.
- **Пульт**: управлять фокусом на ТВ с телефона.

**4.1 Обычный способ:** **Настройки → Специальные возможности → Загруженные/Установленные службы → Лаунчер (или Пульт) → Включить → Разрешить.**

**4.2 Android 13+, переключатель серый:** **Настройки → Приложения → [приложение] → ⋮ (справа сверху) → Разрешить ограниченные настройки**, потом повторите 4.1. Если ⋮ нет, один раз попробуйте включить службу — пункт появится.

**4.3 Если экрана нет — через ADB.** Сначала посмотрите, что уже включено:
```
adb shell settings get secure enabled_accessibility_services
```
- `null` или пусто → включаем:
  ```
  adb shell settings put secure enabled_accessibility_services com.example.pult/com.example.pult.PultAccessibility
  adb shell settings put secure accessibility_enabled 1
  ```
- Что-то есть → **дописываем через двоеточие**, иначе выключатся остальные:
  ```
  adb shell settings put secure enabled_accessibility_services "СТАРОЕ_ЗНАЧЕНИЕ:com.example.pult/com.example.pult.PultAccessibility"
  ```

Универсальная команда (Linux/macOS/Termux), подходит в обоих случаях:
```
adb shell 'cur=$(settings get secure enabled_accessibility_services); case "$cur" in ""|null) new=com.example.pult/com.example.pult.PultAccessibility;; *) new="$cur:com.example.pult/com.example.pult.PultAccessibility";; esac; settings put secure enabled_accessibility_services "$new"; settings put secure accessibility_enabled 1'
```
Для службы Лаунчера замените `com.example.pult/com.example.pult.PultAccessibility` на `com.example.tvlauncher/com.example.tvlauncher.LauncherAccessibilityService`.

В Windows `cmd` выполняйте первые две команды по отдельности.

Выключить все службы: `adb shell settings put secure enabled_accessibility_services null`.

## 5. Остальные разрешения

| Что | Где |
|---|---|
| Установка обновлений | Разрешить при запросе, либо **Настройки → Приложения → [приложение] → Установка неизвестных приложений** |
| Автозапуск Лаунчера | В Лаунчере: **Настройки → Автозапуск лаунчера**; плюс служба из 4.1; на некоторых прошивках разрешить «Автозапуск» в списке приложений |
| Поверх других окон | **Настройки → Приложения → Особый доступ → Поверх других приложений → Лаунчер** |
| Уведомления (Пульт) | **Настройки → Приложения → Пульт → Уведомления** |
| Память / фото (Передача) | Разрешить при запросе |

## 6. Обновление по интернету

1. При каждом запуске приложение спрашивает про обновления (или **Проверить обновления** вручную).
2. **Обновить** — установить, **Позже** — пропустить.
3. Новый APK должен быть подписан **тем же ключом**. Если «приложение не установлено» — один раз удалите старое и поставьте новое.

## 7. Пульт на ПК

Запустите `Run_Pult.bat` (Windows) или `python3 pult_pc.py`; телефон и ПК в одной Wi-Fi сети; порт `47474`.

## 8. Если что-то не работает

| Проблема | Решение |
|---|---|
| `adb не найден` | Откройте терминал в папке platform-tools |
| `unauthorized` | Подтвердите окно на ТВ; иначе «Для разработчиков → Отозвать разрешения отладки» и подключитесь снова |
| `more than one device` | `adb -s 192.168.1.50:5555 shell ...` |
| Не подключается | Одна ли сеть? Включена ли отладка? Перезагрузите ТВ, IP мог смениться |
| Служба сама выключается | **Приложения → [приложение] → Батарея → Без ограничений** |
| «Освободить память» не закрывает | Выключена служба Лаунчера (4.1) или в прошивке нет кнопки «Остановить» |
