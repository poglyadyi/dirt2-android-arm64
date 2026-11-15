Statically recompiled Colin McRae: DiRT 2 for Android ARM64 — native ARM64 code, D3D11 → DXVK → Vulkan, no emulator and no Wine.

**Original project and author:** [XITRIX/Dirt-2-Recomp](https://github.com/XITRIX/Dirt-2-Recomp) — the recompiler, runtime, shims and the port itself are his work; his prebuilt binaries are in [GitHub Actions](https://github.com/XITRIX/Dirt-2-Recomp/actions/workflows/build.yml). This build is his code (commit `cce3bf8`) plus Android-specific changes.

**Tested on:** HONOR MTN-NX1, Android 16, Adreno 810, Turnip Gen8 v32 (`turnip_v26.2.0_b5`), 2640×1200, ultra preset — menus 59.9 fps, races 30.0 fps (the engine's own 30 Hz pacing), 0 shader-hash failures.

**No driver in the APK** — install Turnip separately (see `INSTALL.md`, step 3). No game data is included: you need your own copy of DiRT 2 (PC v1.1).

---

Статически рекомпилированная Colin McRae: DiRT 2 для Android ARM64 — нативный код ARM64, D3D11 → DXVK → Vulkan, без эмулятора и без Wine.

**Автор оригинала:** [XITRIX/Dirt-2-Recomp](https://github.com/XITRIX/Dirt-2-Recomp) — рекомпилятор, рантайм, шимы и сам порт сделаны им; его готовые сборки лежат в [GitHub Actions](https://github.com/XITRIX/Dirt-2-Recomp/actions/workflows/build.yml). Эта сборка — его код (коммит `cce3bf8`) плюс правки под Android.

**Проверено на:** HONOR MTN-NX1, Android 16, Adreno 810, Turnip Gen8 v32 (`turnip_v26.2.0_b5`), 2640×1200, пресет ultra — меню 59.9 fps, гонки 30.0 fps (собственный темп движка 30 Гц), 0 ошибок хеша шейдеров.

**Драйвера в APK нет** — Turnip ставится отдельно (см. `INSTALL.md`, шаг 3). Данных игры в релизе нет: нужна своя копия DiRT 2 (PC v1.1).

### Files

| File | |
|---|---|
| `DiRT2-Android-arm64.apk` | the build, 177 MB (no driver) |
| `README.md` | what it is, author credit, test device and results (RU/EN) |
| `INSTALL.md` | install: build, game, driver (RU/EN) |
| `SHA256SUMS.txt` | checksums |
