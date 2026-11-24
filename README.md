# DiRT 2 — Android ARM64 (native recompilation)

Русский | [English](#english)

---

## Русский

### Автор оригинала

**XITRIX** — [github.com/XITRIX/Dirt-2-Recomp](https://github.com/XITRIX/Dirt-2-Recomp)

Его готовые сборки
(macOS / iOS / Android / Switch) публикуются в GitHub Actions:
[Actions → Build](https://github.com/XITRIX/Dirt-2-Recomp/actions/workflows/build.yml).


### Что это

Игра не эмулируется и не запускается через Wine: её машинный код механически переведён
в C и собран нативно под ARM64. Windows, DirectX, OpenAL, Bink и Games for Windows Live
заменены реализациями под хост:

| Оригинал | Замена |
|---|---|
| Direct3D 11 | DXVK → Vulkan (драйвер Turnip на устройстве) |
| OpenAL / Rapture3D | OpenAL Soft |
| XInput / DirectInput | SDL3 + геймпад, клавиатура, автоматический тач-контроллер |
| Win32 окно и сообщения | SDL3 |
| Bink 1 видео | FFmpeg `bink` / `binkaudio` |
| GFWL (`xlive.dll`) | офлайн-заглушка |
| Winsock | BSD sockets |

Данные игры и `dirt2_game.exe` **не входят** в сборку: нужна своя копия игры.

### Устройство для теста и результат

| | |
|---|---|
| Устройство | HONOR MTN-NX1 (модель `MTN-NX1`) |
| ОС | Android 16 |
| GPU | Adreno 810 |
| Драйвер Vulkan | Turnip Gen8 **v32** (`turnip_v26.2.0_b5`), md5 `6a4f5c6ff8f8ae33cd232253c6b339c8` — только для Adreno 8xx; для 6xx/7xx нужен v26.3.0-R5 |
| Разрешение рендера | 2640 × 1200 (полный экран, альбомная ориентация) |
| Качество | preset «ultra» (детализация ultra, тени 2048, MSAA off, зеркала off) |

Результат:

| Метрика | Значение |
|---|---|
| Меню, титульный экран, загрузка | 59.8–59.9 fps |
| Гонка | 30.0 fps (игра сама держит 30 Гц — это её собственный темп, а не предел GPU) |
| Ошибки хеша шейдеров | 0 |
| Ошибки загрузки Vulkan | нет |
| Что проверено | гонки, меню, звук, тач-управление и геймпад — работают |

То есть на этом устройстве игра идёт стабильно на своём целевом темпе, без графических
артефактов и без падений во время тестов.

### Что отличается от сборки автора

- Собрано и отлажено под Android ARM64 с D3D11 → DXVK → Vulkan через `libadrenotools`
  и внешний драйвер Turnip (вендорский драйвер Adreno не даёт нужных DXVK возможностей).
- Автоматический тач-контроллер (газ/тормоз/руль/камера) — играть можно без геймпада;
  при подключении геймпада, клавиатуры или мыши оверлей прячется.
- Правки под Qualcomm: раскладки сэмплируемых render target'ов, load/store аттачментов,
  корректный `VK_SUBOPTIMAL_KHR`, FIFO-презентация вместо MAILBOX (иначе композитор
  задерживает кадры на секунды).
- Оптимизации темпа (pacing) и синхронизации гостевых потоков, счётчики
  производительности DXVK.

### Файлы релиза

| Файл | Что это |
|---|---|
| `DiRT2-Android-arm64.apk` | сборка (177 МБ распакованных библиотек, **без драйвера**) |
| `INSTALL.md` | инструкция: сборка / игра / драйвер |
| `SHA256SUMS.txt` | контрольные суммы |

Драйвера в APK нет намеренно: Turnip обновляется отдельно и не должен обновляться вместе
с игрой. Как его поставить — в `INSTALL.md`, раздел 3.

### Установка (кратко)

1. `adb install -r DiRT2-Android-arm64.apk`
2. Скопировать свою папку `Dirt 2` (с `dirt2_game.exe`) в `files/Dirt 2` приложения.
3. Положить драйвер Turnip в `files/driver/vulkan.ad07xx.so` — сборка зависит от
   поколения GPU: Adreno 8xx → Gen8 v32, Adreno 6xx/7xx → v26.3.0-R5
   (см. `INSTALL.md`, шаг 3).

Подробно, с командами и диагностикой: **[INSTALL.md](INSTALL.md)**.

### Известные ограничения

- Сборка отладочная (`debuggable`, подписана локальным ключом разработчика). Для
  обновлений «поверх» нужен тот же ключ, иначе сначала удалите приложение.
- Гонки идут на 30 fps: так устроен сам движок игры (он же и на ПК с vsync).
- MSAA в тестовом пресете выключен: на Turnip он даёт только просадку.
- Прогресс и настройки лежат в приватных данных приложения и удаляются вместе с ним.
- Полноценный геймпад удобнее тача; раскладка оверлея подстраивается под safe area экрана.

### Правовая часть

Сборка не содержит данных игры и не является её распространителем: нужна легальная
копия Colin McRae: DiRT 2 (PC v1.1). Рекомпилятор и рантайм — работа XITRIX и
контрибьюторов проекта; драйвер Turnip — Mesa / [StevenMXZ](https://github.com/StevenMXZ/Adreno-Tools-Drivers).

---

## English

### Original author

**XITRIX** — [github.com/XITRIX/Dirt-2-Recomp](https://github.com/XITRIX/Dirt-2-Recomp)

His prebuilt binaries
(macOS / iOS / Android / Switch) are published as GitHub Actions artifacts:
[Actions → Build](https://github.com/XITRIX/Dirt-2-Recomp/actions/workflows/build.yml).


### What this is

The game is not emulated and not run under Wine: its machine code is lifted mechanically
into C and compiled natively for ARM64. Windows, DirectX, OpenAL, Bink and Games for
Windows Live imports are replaced with host implementations:

| Original | Replacement |
|---|---|
| Direct3D 11 | DXVK → Vulkan (device-side Turnip driver) |
| OpenAL / Rapture3D | OpenAL Soft |
| XInput / DirectInput | SDL3 with gamepad, keyboard fallback and automatic touch controller |
| Win32 window and messages | SDL3 |
| Bink 1 video | FFmpeg `bink` / `binkaudio` |
| GFWL (`xlive.dll`) | offline stub |
| Winsock | BSD sockets |

Game data and `dirt2_game.exe` are **not** included: you need your own copy of the game.

### Test device and result

| | |
|---|---|
| Device | HONOR MTN-NX1 (model `MTN-NX1`) |
| OS | Android 16 |
| GPU | Adreno 810 |
| Vulkan driver | Turnip Gen8 **v32** (`turnip_v26.2.0_b5`), md5 `6a4f5c6ff8f8ae33cd232253c6b339c8` — Adreno 8xx only; Adreno 6xx/7xx needs v26.3.0-R5 |
| Render resolution | 2640 × 1200 (full screen, landscape) |
| Quality | “ultra” preset (detail ultra, shadows 2048, MSAA off, mirrors off) |

Result:

| Metric | Value |
|---|---|
| Menus, title screen, loading | 59.8–59.9 fps |
| Race | 30.0 fps (the game paces itself at 30 Hz — that is the engine's own rate, not a GPU limit) |
| Shader hash failures | 0 |
| Vulkan load errors | none |
| Play test | races, menus, audio, touch controls and gamepad all work |

On this device the game runs stably at its own target rate, with no visual artefacts and
no crashes during testing.

### Differences from the author's build

- Built and debugged for Android ARM64 with D3D11 → DXVK → Vulkan through
  `libadrenotools` and an external Turnip driver (the vendor Adreno driver lacks the
  Vulkan features DXVK needs).
- Automatic touch controller (throttle, brake, steering, camera), so no gamepad is
  required; connecting a gamepad, keyboard or mouse hides the overlay.
- Qualcomm fixes: sampled render-target and attachment load/store layouts, correct
  `VK_SUBOPTIMAL_KHR` handling, FIFO presentation instead of MAILBOX (otherwise the
  compositor holds frames for seconds).
- Guest thread pacing/synchronisation work and DXVK performance counters.

### Release contents

| File | What it is |
|---|---|
| `DiRT2-Android-arm64.apk` | the build (177 MB of unpacked libraries, **no driver**) |
| `INSTALL.md` | build / game / driver instructions |
| `SHA256SUMS.txt` | checksums |

There is deliberately no driver in the APK: Turnip is updated separately and should not
be tied to game updates. See `INSTALL.md`, section 3.

### Install (short version)

1. `adb install -r DiRT2-Android-arm64.apk`
2. Copy your own `Dirt 2` folder (the one with `dirt2_game.exe`) into the app's `files/Dirt 2`.
3. Put the Turnip driver at `files/driver/vulkan.ad07xx.so` — the build depends on the
   GPU generation: Adreno 8xx → Gen8 v32, Adreno 6xx/7xx → v26.3.0-R5
   (see `INSTALL.md`, step 3).

Full instructions, commands and troubleshooting: **[INSTALL.md](INSTALL.md)**.

### Known limitations

- Development build (`debuggable`, signed with a local developer key). In-place updates
  need the same key; otherwise uninstall first.
- Races run at 30 fps because the game engine paces itself that way (same on PC with vsync).
- MSAA is off in the tested preset: on Turnip it only costs frame rate.
- Saves and settings live in the app's private data and are removed with the app.
- A real gamepad is more comfortable than touch; the overlay layout follows the screen's safe area.

### Legal

This build contains no game data and does not distribute the game: you need a legal copy of
Colin McRae: DiRT 2 (PC v1.1). The recompiler and runtime are the work of XITRIX and the
project's contributors; the Turnip driver is Mesa / [StevenMXZ](https://github.com/StevenMXZ/Adreno-Tools-Drivers).
