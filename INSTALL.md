# DiRT 2 на Android — установка

Русский | [English](#english)

---

## Русский

### Что нужно

- Телефон или планшет на **ARM64** (ARMv8), Android 9+ и **Vulkan 1.3** (все свежие
  Adreno 6xx/7xx/8xx, Mali-G7xx/G9xx подходят; x86-устройства не поддерживаются).
  Драйвер Turnip подбирается по поколению GPU — см. шаг 3.
- Своя легальная копия **Colin McRae: DiRT 2** для ПК, версия **1.1** (Steam) — папка,
  в которой лежит `dirt2_game.exe`. В сборку игра не входит.
- ADB (platform-tools) и включённая отладка по USB, либо файловый менеджер на телефоне.
- ~2 ГБ свободного места под игру (зависит от вашей копии) плюс ~200 МБ под приложение.

Пакет приложения: `com.daniilvinogradov.dirt2`. Все дальнейшие команды — с ним.
Если ADB видит несколько устройств, добавьте `-s <серийный номер>` (`adb devices`).

---

### Шаг 1. Сборка (APK)

```powershell
adb install -r DiRT2-Android-arm64.apk
```

Без ADB: перекиньте APK на телефон и откройте его — установка из файлового менеджера
(разрешите установку из этого источника). APK подписан локальным ключом разработчика и
собран как отладочный, поэтому «Неизвестный источник» — это нормально.

Проверить, что установилось:

```powershell
adb shell pm path com.daniilvinogradov.dirt2
```

Лучше сразу пройти шаг 3: без Turnip игра падает на системный драйвер Adreno — это
непроверенная конфигурация, и вместо картинки легко получить артефакты, чёрный экран или
ошибку создания Vulkan-устройства.

---

### Шаг 2. Игра (ваша папка `Dirt 2`)

Нужна папка с `dirt2_game.exe` внутри (обычно `...\Steam\steamapps\common\Colin McRae DiRT 2\Dirt 2`).
Копировать надо **в приватные данные приложения**: `files/Dirt 2`.
Внешняя карта памяти и `Android/data` не подходят — игра читает приватный каталог.

Два важных уточнения:

- Нужна **установленная** папка игры, а не файлы установщика/репака. Если внутри лежат
  `setup.exe`, `Launcher.7z`, `fix repack rus.zip` и подобное — это инсталлятор: сначала
  установите игру на ПК, потом копируйте получившуюся папку `Dirt 2`, где рядом с
  `dirt2_game.exe` есть каталог `data`.
- Сообщение «Use the installation helper» внутри приложения относится к скрипту
  `tools/scripts/install_android.sh` на машине, где собирали APK. В самом APK установщика
  нет, поэтому для готовой сборки данные ставятся только через ADB (варианты ниже) или
  по беспроводному ADB.

**Вариант A — ADB (рекомендуется).** Копирование идёт через `/data/local/tmp`,
распаковка — от имени приложения, поэтому ограничения scoped storage не мешают.

```powershell
$adb  = "adb"
$game = "D:\Games\Dirt 2"        # ваша папка, в ней должен быть dirt2_game.exe
$pkg  = "com.daniilvinogradov.dirt2"

& $adb shell mkdir -p /data/local/tmp/dirt2-assets
& $adb push --sync -a "$game/." /data/local/tmp/dirt2-assets
& $adb shell "run-as $pkg mkdir -p 'files/Dirt 2'"
& $adb shell "toybox tar -cf - -C /data/local/tmp/dirt2-assets . | run-as $pkg toybox tar -xof - -C 'files/Dirt 2'"
& $adb shell rm -rf /data/local/tmp/dirt2-assets
```

То же самое в bash (macOS/Linux):

```bash
adb shell mkdir -p /data/local/tmp/dirt2-assets
adb push --sync -a "/path/to/Dirt 2/." /data/local/tmp/dirt2-assets
adb shell "run-as com.daniilvinogradov.dirt2 mkdir -p 'files/Dirt 2'"
adb shell "toybox tar -cf - -C /data/local/tmp/dirt2-assets . | run-as com.daniilvinogradov.dirt2 toybox tar -xof - -C 'files/Dirt 2'"
adb shell rm -rf /data/local/tmp/dirt2-assets
```

Проверка:

```powershell
adb shell "run-as com.daniilvinogradov.dirt2 ls 'files/Dirt 2'" | Select-String dirt2_game.exe
adb shell "run-as com.daniilvinogradov.dirt2 du -sh 'files/Dirt 2'"
```

(в bash — те же строки, `Select-String` заменить на `grep`)

**Вариант B — без ADB.** Копирование большого каталога в приватную папку приложения
без ADB неудобно (нужен файловый менеджер с доступом к `Android/data` через SAF,
и это работает не на всех прошивках). Надёжный путь — ADB по Wi-Fi: «Для разработчиков →
Отладка по Wi-Fi», `adb pair`, затем `adb connect <ip>:<порт>`, дальше как в варианте A.

Если папка игры уже лежит на телефоне (например, в `Download`), её можно переложить
через тот же staging-каталог (`shell` читает `/sdcard`, приложение — нет):

```powershell
adb shell mkdir -p /data/local/tmp/dirt2-assets
adb shell "cp -r '/sdcard/Download/Dirt 2/.' /data/local/tmp/dirt2-assets/"
adb shell "run-as com.daniilvinogradov.dirt2 mkdir -p 'files/Dirt 2'"
adb shell "toybox tar -cf - -C /data/local/tmp/dirt2-assets . | run-as com.daniilvinogradov.dirt2 toybox tar -xof - -C 'files/Dirt 2'"
adb shell rm -rf /data/local/tmp/dirt2-assets
```

---

### Шаг 3. Драйвер (обязательно)

В APK драйвера нет. Игра ищет его строго по пути:

```
/data/user/0/com.daniilvinogradov.dirt2/files/driver/vulkan.ad07xx.so
```

Встроенный драйвер Adreno не даёт DXVK нужных возможностей Vulkan, поэтому нужен
**Mesa Turnip**. Сборки Turnip не взаимозаменяемы: они собираются под конкретное
поколение GPU, и драйвер «не того» поколения обычно даёт чёрный экран или падение
сразу после логотипа. Возьмите строку для своего GPU:

| GPU | Драйвер | Что внутри |
|---|---|---|
| Adreno 8xx (a810, a825, a829, a830, a840) | **Turnip Gen8 v32** (`turnip_v26.2.0_b5`) | `libvulkan_freedreno.so`, 15 006 216 байт, md5 `6a4f5c6ff8f8ae33cd232253c6b339c8` |
| Adreno 6xx/7xx (a6xx/a7xx; например Adreno 650 в Poco F3) | **Turnip v26.3.0-R5** | `libvulkan_freedreno.so`, 14 188 536 байт, md5 `15382fed0aa6407ff7f46eec5a2189b0` |

`Gen8 v32` собран только под Adreno 8xx — на a6xx/a7xx он не заработает.

Откуда взять (GitHub, [StevenMXZ/Adreno-Tools-Drivers](https://github.com/StevenMXZ/Adreno-Tools-Drivers/releases)):

- Adreno 8xx: релиз [v32](https://github.com/StevenMXZ/Adreno-Tools-Drivers/releases/tag/v32) →
  `Turnip_Gen8_V32.zip` → внутри `libvulkan_freedreno.so`.
- Adreno 6xx/7xx: релиз [v26.3.0-R5](https://github.com/StevenMXZ/Adreno-Tools-Drivers/releases/tag/v26.3.0-R5) →
  `Turnip_v26.3.0-R5.zip` (не `...-Oneui.zip`) → внутри `libvulkan_freedreno.so`.
- Либо из GameHub (Xiaoji/GameSir): список драйверов → сборка для своего поколения →
  «Экспорт». Для тестового устройства это `turnip_v26.2.0_b5`, md5 совпадает.

Проверить поколение GPU: `adb shell dumpsys SurfaceFlinger | grep GLES` (строка вида
`Adreno (TM) 650`) или любое приложение вроде Device Info HW.

Драйвер в Android — это HAL, а не ICD, поэтому имя файла обязательно
`vulkan.ad07xx.so` (переименуйте `libvulkan_freedreno.so`).

Установка (Windows PowerShell):

```powershell
$adb  = "adb"
$pkg  = "com.daniilvinogradov.dirt2"
$drv  = "C:\drivers\Turnip_Gen8_V32\libvulkan_freedreno.so"   # переименуем при копировании

& $adb shell rm -rf /data/local/tmp/dirt2-driver
& $adb shell mkdir -p /data/local/tmp/dirt2-driver
& $adb push "$drv" /data/local/tmp/dirt2-driver/vulkan.ad07xx.so
& $adb shell chmod 644 /data/local/tmp/dirt2-driver/vulkan.ad07xx.so
& $adb shell "run-as $pkg mkdir -p files/driver"
& $adb shell "run-as $pkg cp /data/local/tmp/dirt2-driver/vulkan.ad07xx.so files/driver/vulkan.ad07xx.so"
& $adb shell rm -rf /data/local/tmp/dirt2-driver
& $adb shell "run-as $pkg ls -l files/driver"
```

bash-вариант — те же команды без `&` и `$adb` → `adb`.

Шимы `libd2cut.so` и `libd2hw.so` (Turnip использует `libcutils`/`libhardware`/
`libsync`/`libnativewindow`) **уже лежат в APK** и подхватываются оттуда, отдельно
их ставить не нужно. В проверенной конфигурации они дополнительно скопированы в
`files/driver` — на работу это не влияет.

Проверка md5 на телефоне:

```powershell
adb shell "run-as com.daniilvinogradov.dirt2 md5sum files/driver/vulkan.ad07xx.so"
# 6a4f5c6ff8f8ae33cd232253c6b339c8  files/driver/vulkan.ad07xx.so
```

Можно держать несколько драйверов и переключаться без замены файла:

```powershell
adb push other_turnip.so /data/local/tmp/other.so
adb shell "run-as com.daniilvinogradov.dirt2 cp /data/local/tmp/other.so files/driver/other.so"
# затем в files/Dirt2Data/env.txt: DIRT2_VULKAN_DRIVER=/data/user/0/com.daniilvinogradov.dirt2/files/driver/other.so
```

---

### Шаг 4. Первый запуск

```powershell
adb shell am start -n com.daniilvinogradov.dirt2/.Dirt2Activity
```

- Первый запуск долгий: DXVK компилирует шейдеры «на лету», вверху экрана виден
  счётчик `fps`/`compiler`. Загрузка меню может занять минуту-две, подгрузки во время
  первой гонки — тоже. Дальше компиляция идёт из кэша и всё быстрее.
- Управление: тач-оверлей появляется автоматически, если не подключены геймпад,
  клавиатура или мышь. Геймпад (XInput-совместимый, Bluetooth/USB) удобнее.
- Настройки графики — в меню игры (Options → Graphics). Проверенный пресет: разрешение
  2640×1200, детализация ultra, тени 2048, MSAA off, зеркала off.
- Экран входа/меню и раскладка оверлея подстраиваются под safe area и ориентацию.

---

### Обновление и удаление

```powershell
# обновление поверх (данные игры сохраняются)
adb install -r DiRT2-Android-arm64.apk

# полное удаление (вместе с данными игры, сохранениями и настройками)
adb uninstall com.daniilvinogradov.dirt2
```

`adb install -r` сработает только с APK, подписанным тем же ключом. Чужая сборка
(например, из CI автора) потребует сначала удалить приложение — и вместе с ним
скопированную игру.

---

### Данные, сохранения и диагностика

| Что | Где |
|---|---|
| Игра | `files/Dirt 2` (`dirt2_game.exe` внутри) |
| Настройки, сохранения, логи | `files/Dirt2Data` |
| Драйвер | `files/driver/vulkan.ad07xx.so` |
| Переменные окружения | `files/Dirt2Data/env.txt` (строки `KEY=VALUE`) |
| Лог игры | `files/Dirt2Data/dirt2.log`, предыдущий — `dirt2.prev.log` |

Сохранения повторяют ПК-раскладку:
`files/Dirt2Data/C/Users/Player/Documents/My Games/DiRT2/savegame`, поэтому их можно
перенести с ПК и обратно. Конфиг качества —
`.../My Games/DiRT2/hardwaresettings/hardware_settings_config.xml` (его можно
вытащить, поправить и закинуть назад, игра не запущена).

Полезные переменные в `env.txt`:

```
DXVK_HUD=fps,compiler            # по умолчанию включено; пустое значение убирает HUD
DXVK_CONFIG=dxgi.syncInterval = 1
TU_DEBUG=...                     # отладка Turnip
DIRT2_VULKAN_DRIVER=/data/user/0/com.daniilvinogradov.dirt2/files/driver/other.so
```

Записать `env.txt`:

```powershell
adb shell "run-as com.daniilvinogradov.dirt2 sh -c 'cat > files/Dirt2Data/env.txt'" < env.txt
```

Забрать лог и падения:

```powershell
adb exec-out run-as com.daniilvinogradov.dirt2 cat files/Dirt2Data/dirt2.log > dirt2.log
adb logcat -b crash -d > crash.log
```

Типичные проблемы:

| Симптом | Причина |
|---|---|
| Сообщение «game data missing» с путём | нет `files/Dirt 2/dirt2_game.exe` — повторите шаг 2 |
| Игра скопирована, но снова «game data missing» | папка вложена (`files/Dirt 2/Dirt 2/...`) или данные лежат в `Android/data` — нужен ровно `files/Dirt 2/dirt2_game.exe` |
| Чёрный экран, в логе `Vulkan: cannot load external driver` | драйвера нет или он не переименован в `vulkan.ad07xx.so` |
| Падение или артефакты сразу после логотипа | драйвер не того поколения: `Gen8 v32` на Adreno 6xx/7xx (нужен `v26.3.0-R5`) или наоборот, см. шаг 3 |
| Чёрный экран после сворачивания | перезапустите игру (жизненный цикл Android) |
| «App not installed» при обновлении | другая подпись — удалите приложение и поставьте заново |

---

### Контрольные суммы

```powershell
certutil -hashfile DiRT2-Android-arm64.apk SHA256
```

Сверьте с `SHA256SUMS.txt` из этого релиза.

---

## English

### Requirements

- **ARM64** phone or tablet (ARMv8), Android 9+ and **Vulkan 1.3** (any recent
  Adreno 6xx/7xx/8xx or Mali-G7xx/G9xx; x86 devices are not supported).
  The Turnip driver depends on the GPU generation — see step 3.
- Your own legal copy of **Colin McRae: DiRT 2** for PC, version **1.1** (Steam) — the
  folder that contains `dirt2_game.exe`. The game is not part of this release.
- ADB (platform-tools) with USB debugging, or a file manager on the device.
- ~2 GB of free space for the game (depends on your copy) plus ~200 MB for the app.

App package: `com.daniilvinogradov.dirt2`. All commands below use it. With several
devices attached, add `-s <serial>` (`adb devices`).

---

### Step 1. The build (APK)

```powershell
adb install -r DiRT2-Android-arm64.apk
```

Without ADB: copy the APK to the device and open it — install from the file manager
(allow that source). The APK is a development build signed with a local key, so
"unknown source" is expected.

Verify:

```powershell
adb shell pm path com.daniilvinogradov.dirt2
```

Do step 3 right away: without Turnip the game falls back to the system Adreno driver,
which is not a tested configuration — expect artefacts, a black screen or a failure to
create a Vulkan device instead of a picture.

---

### Step 2. The game (your `Dirt 2` folder)

You need the folder containing `dirt2_game.exe` (usually
`...\Steam\steamapps\common\Colin McRae DiRT 2\Dirt 2`). It must go into the app's
**private** storage: `files/Dirt 2`. An SD card or `Android/data` will not work — the
game reads the private directory.

Two things that catch people out:

- It must be the **installed** game folder, not installer/repack files. If it contains
  `setup.exe`, `Launcher.7z`, `fix repack rus.zip` and the like, that is the installer:
  install the game on a PC first, then copy the resulting `Dirt 2` folder that has a
  `data` directory next to `dirt2_game.exe`.
- The "Use the installation helper" text inside the app refers to
  `tools/scripts/install_android.sh` on the machine that built the APK. The APK has no
  installer of its own, so with a ready build the data goes in over ADB (options below)
  or wireless ADB.

**Option A — ADB (recommended).** Files are staged in `/data/local/tmp` and unpacked as
the app UID, so scoped storage is not an issue.

```powershell
$adb  = "adb"
$game = "D:\Games\Dirt 2"        # your folder; must contain dirt2_game.exe
$pkg  = "com.daniilvinogradov.dirt2"

& $adb shell mkdir -p /data/local/tmp/dirt2-assets
& $adb push --sync -a "$game/." /data/local/tmp/dirt2-assets
& $adb shell "run-as $pkg mkdir -p 'files/Dirt 2'"
& $adb shell "toybox tar -cf - -C /data/local/tmp/dirt2-assets . | run-as $pkg toybox tar -xof - -C 'files/Dirt 2'"
& $adb shell rm -rf /data/local/tmp/dirt2-assets
```

The same in bash (macOS/Linux):

```bash
adb shell mkdir -p /data/local/tmp/dirt2-assets
adb push --sync -a "/path/to/Dirt 2/." /data/local/tmp/dirt2-assets
adb shell "run-as com.daniilvinogradov.dirt2 mkdir -p 'files/Dirt 2'"
adb shell "toybox tar -cf - -C /data/local/tmp/dirt2-assets . | run-as com.daniilvinogradov.dirt2 toybox tar -xof - -C 'files/Dirt 2'"
adb shell rm -rf /data/local/tmp/dirt2-assets
```

Verify:

```powershell
adb shell "run-as com.daniilvinogradov.dirt2 ls 'files/Dirt 2'" | Select-String dirt2_game.exe
adb shell "run-as com.daniilvinogradov.dirt2 du -sh 'files/Dirt 2'"
```

(In bash use the same lines with `grep` instead of `Select-String`.)

**Option B — no ADB.** Copying a large folder into app-private storage without ADB is
awkward (needs a file manager with SAF access to `Android/data`, which not every ROM
provides). The reliable route is wireless ADB: Developer options → Wireless debugging,
`adb pair`, then `adb connect <ip>:<port>`, then option A.

If the game folder is already on the device (e.g. in `Download`), move it through the same
staging directory (`shell` can read `/sdcard`, the app cannot):

```powershell
adb shell mkdir -p /data/local/tmp/dirt2-assets
adb shell "cp -r '/sdcard/Download/Dirt 2/.' /data/local/tmp/dirt2-assets/"
adb shell "run-as com.daniilvinogradov.dirt2 mkdir -p 'files/Dirt 2'"
adb shell "toybox tar -cf - -C /data/local/tmp/dirt2-assets . | run-as com.daniilvinogradov.dirt2 toybox tar -xof - -C 'files/Dirt 2'"
adb shell rm -rf /data/local/tmp/dirt2-assets
```

---

### Step 3. The driver (required)

The APK ships no driver. The game looks for it at exactly:

```
/data/user/0/com.daniilvinogradov.dirt2/files/driver/vulkan.ad07xx.so
```

The vendor Adreno driver lacks the Vulkan features DXVK needs, so **Mesa Turnip** is
required. Turnip builds are not interchangeable: each is compiled for a GPU generation,
and a driver for the wrong generation usually means a black screen or a crash right after
the logo. Pick the row for your GPU:

| GPU | Driver | Contents |
|---|---|---|
| Adreno 8xx (a810, a825, a829, a830, a840) | **Turnip Gen8 v32** (`turnip_v26.2.0_b5`) | `libvulkan_freedreno.so`, 15 006 216 bytes, md5 `6a4f5c6ff8f8ae33cd232253c6b339c8` |
| Adreno 6xx/7xx (a6xx/a7xx; e.g. Adreno 650 in Poco F3) | **Turnip v26.3.0-R5** | `libvulkan_freedreno.so`, 14 188 536 bytes, md5 `15382fed0aa6407ff7f46eec5a2189b0` |

`Gen8 v32` is built for Adreno 8xx only — it will not run on a6xx/a7xx.

Where to get it (GitHub, [StevenMXZ/Adreno-Tools-Drivers](https://github.com/StevenMXZ/Adreno-Tools-Drivers/releases)):

- Adreno 8xx: release [v32](https://github.com/StevenMXZ/Adreno-Tools-Drivers/releases/tag/v32) →
  `Turnip_Gen8_V32.zip` → contains `libvulkan_freedreno.so`.
- Adreno 6xx/7xx: release [v26.3.0-R5](https://github.com/StevenMXZ/Adreno-Tools-Drivers/releases/tag/v26.3.0-R5) →
  `Turnip_v26.3.0-R5.zip` (not `...-Oneui.zip`) → contains `libvulkan_freedreno.so`.
- Or from GameHub (Xiaoji/GameSir): driver list → the build for your generation →
  export. For the test device that is `turnip_v26.2.0_b5`, same md5.

Check the GPU generation: `adb shell dumpsys SurfaceFlinger | grep GLES` (a line such as
`Adreno (TM) 650`), or any app like Device Info HW.

On Android the driver is a HAL, not an ICD, so the file name must be
`vulkan.ad07xx.so` (rename `libvulkan_freedreno.so`).

Install (Windows PowerShell):

```powershell
$adb  = "adb"
$pkg  = "com.daniilvinogradov.dirt2"
$drv  = "C:\drivers\Turnip_Gen8_V32\libvulkan_freedreno.so"   # renamed while copying

& $adb shell rm -rf /data/local/tmp/dirt2-driver
& $adb shell mkdir -p /data/local/tmp/dirt2-driver
& $adb push "$drv" /data/local/tmp/dirt2-driver/vulkan.ad07xx.so
& $adb shell chmod 644 /data/local/tmp/dirt2-driver/vulkan.ad07xx.so
& $adb shell "run-as $pkg mkdir -p files/driver"
& $adb shell "run-as $pkg cp /data/local/tmp/dirt2-driver/vulkan.ad07xx.so files/driver/vulkan.ad07xx.so"
& $adb shell rm -rf /data/local/tmp/dirt2-driver
& $adb shell "run-as $pkg ls -l files/driver"
```

For bash use the same commands without `&` and with `adb` instead of `$adb`.

The `libd2cut.so` and `libd2hw.so` shims (Turnip needs `libcutils`/`libhardware`/
`libsync`/`libnativewindow`) are **already inside the APK** and are loaded from there, so
you do not have to copy them. In the tested setup they are additionally present in
`files/driver`; that does not change behaviour.

Check the hash on the device:

```powershell
adb shell "run-as com.daniilvinogradov.dirt2 md5sum files/driver/vulkan.ad07xx.so"
# 6a4f5c6ff8f8ae33cd232253c6b339c8  files/driver/vulkan.ad07xx.so
```

You can keep several drivers and switch without replacing files:

```powershell
adb push other_turnip.so /data/local/tmp/other.so
adb shell "run-as com.daniilvinogradov.dirt2 cp /data/local/tmp/other.so files/driver/other.so"
# then in files/Dirt2Data/env.txt: DIRT2_VULKAN_DRIVER=/data/user/0/com.daniilvinogradov.dirt2/files/driver/other.so
```

---

### Step 4. First launch

```powershell
adb shell am start -n com.daniilvinogradov.dirt2/.Dirt2Activity
```

- The first launch takes a while: DXVK compiles shaders on first sight and the `fps` /
  `compiler` HUD shows it. The menus can take a minute or two, as can loading during the
  first race. Afterwards the cache makes everything faster.
- Controls: the touch overlay appears automatically when no gamepad, keyboard or mouse is
  connected. An XInput-compatible gamepad (Bluetooth or USB) is more comfortable.
- Graphics settings: in-game Options → Graphics. Tested preset: 2640×1200, detail ultra,
  shadows 2048, MSAA off, mirrors off.
- The overlay and menus follow the safe area and orientation.

---

### Update and uninstall

```powershell
# update in place (game data is kept)
adb install -r DiRT2-Android-arm64.apk

# full removal (including game data, saves and settings)
adb uninstall com.daniilvinogradov.dirt2
```

`adb install -r` only works with an APK signed by the same key. A different build (for
example the author's CI artifact) requires uninstalling first — which also deletes the
copied game.

---

### Data, saves and diagnostics

| What | Where |
|---|---|
| Game | `files/Dirt 2` (contains `dirt2_game.exe`) |
| Settings, saves, logs | `files/Dirt2Data` |
| Driver | `files/driver/vulkan.ad07xx.so` |
| Environment overrides | `files/Dirt2Data/env.txt` (`KEY=VALUE` lines) |
| Game log | `files/Dirt2Data/dirt2.log`, previous run in `dirt2.prev.log` |

Saves follow the PC layout:
`files/Dirt2Data/C/Users/Player/Documents/My Games/DiRT2/savegame`, so they can be moved
from a PC and back. The quality config is
`.../My Games/DiRT2/hardwaresettings/hardware_settings_config.xml` (pull it, edit it,
push it back while the game is not running).

Useful variables in `env.txt`:

```
DXVK_HUD=fps,compiler            # on by default; an empty value hides the HUD
DXVK_CONFIG=dxgi.syncInterval = 1
TU_DEBUG=...                     # Turnip debugging
DIRT2_VULKAN_DRIVER=/data/user/0/com.daniilvinogradov.dirt2/files/driver/other.so
```

Write `env.txt`:

```powershell
adb shell "run-as com.daniilvinogradov.dirt2 sh -c 'cat > files/Dirt2Data/env.txt'" < env.txt
```

Collect the log and crashes:

```powershell
adb exec-out run-as com.daniilvinogradov.dirt2 cat files/Dirt2Data/dirt2.log > dirt2.log
adb logcat -b crash -d > crash.log
```

Common problems:

| Symptom | Cause |
|---|---|
| "game data missing" message with a path | `files/Dirt 2/dirt2_game.exe` is missing — redo step 2 |
| Copied the game, still "game data missing" | nested folder (`files/Dirt 2/Dirt 2/...`) or the data is in `Android/data` — it must be exactly `files/Dirt 2/dirt2_game.exe` |
| Black screen, log says `Vulkan: cannot load external driver` | driver missing or not named `vulkan.ad07xx.so` |
| Crash or artefacts right after the logo | driver for the wrong generation: `Gen8 v32` on Adreno 6xx/7xx (needs `v26.3.0-R5`) or the other way round, see step 3 |
| Black screen after backgrounding | restart the game (Android lifecycle) |
| "App not installed" on update | different signature — uninstall and install again |

---

### Checksums

```powershell
certutil -hashfile DiRT2-Android-arm64.apk SHA256
```

Compare with `SHA256SUMS.txt` in this release.
