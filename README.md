# Android QA Toolkit

**Open-source Android QA utilities for device discovery, UI smoke testing, diagnostics, carrier configuration validation, boot-animation comparison, and Excel-driven E2E checks.**

The toolkit is manufacturer-neutral: it targets Android through ADB and Appium UiAutomator2. It does not require a Motorola-, Samsung-, Xiaomi-, or other OEM-specific SDK.

## Features

- **Device Manager:** discovers all ADB devices, reports `device`, `offline`, and `unauthorized` states, and collects model, manufacturer, Android version/API level, build, ABI, screen, and battery metadata.
- **Safe device selection:** auto-selects only when exactly one device is online; refuses to guess when multiple devices are online.
- **Appium Test Engine:** creates sessions pinned to the chosen ADB serial (`udid`), with optional target app configuration.
- **Failure evidence:** attaches screenshots and Logcat excerpts to pytest HTML reports when Appium tests fail.
- **Log Analyzer:** captures Logcat and classifies common signals such as fatal Java exceptions, ANRs, AndroidRuntime messages, and native crashes.
- **Excel E2E runner:** reads test cases from `.xlsx`, validates package installation and app version, captures screenshots, and writes results/evidence links.
- **Carrier Validator:** validates generic carrier configuration JSON using editable rules.
- **Boot Animation Comparator:** compares `bootanimation.zip` metadata, frame hashes, and file changes; includes a CLI and a CustomTkinter GUI.
- **CI:** GitHub Actions runs the hardware-independent test suite on multiple Python versions.

## Requirements

- Python 3.10+
- Android SDK Platform-Tools (`adb`) on `PATH`
- JDK and Android SDK (including Build-Tools and Platform-Tools)
- Node.js `^20.19.0 || ^22.12.0 || >=24.0.0` and npm 10+
- Appium 3 server
- Appium UiAutomator2 driver 5.0.7 (recommended pinned version)
- Android device or emulator with USB/Wireless debugging enabled

All software dependencies are free/open source. No paid cloud test service is required. See `docs/DEVICE_SUPPORT.md` for the supported Android-version caveats.

## Install on Windows (PowerShell)

```powershell
py -3 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
Copy-Item .env.example .env
```

## Install on Linux/macOS

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
pip install -r requirements.txt
cp .env.example .env
```

## Install Appium and Android driver

```bash
npm install -g appium@^3
appium driver install uiautomator2@5.0.7
appium driver doctor uiautomator2
```

The server and driver are separate from the Python package. Start Appium in a dedicated terminal:

```bash
appium
```

The default server URL is `http://127.0.0.1:4723`. Override it with `APPIUM_SERVER_URL` in `.env` or with pytest's `--appium-url` option.

## Connect and inspect Android devices

Enable Developer options and USB debugging, connect the device, and accept the RSA authorization prompt.

```bash
adb devices -l
python -m android_e2e.device
python -m android_e2e.device --json
python -m android_e2e.cli info --serial YOUR_SERIAL
```

If only one device is online, commands select it automatically. If multiple are online, pass a serial explicitly. The toolkit never silently chooses between multiple online devices.

## Run tests

Hardware-independent tests (no Appium server or phone required):

```bash
python -m pytest -m "not appium" -v
```

Appium device-session smoke tests (Appium server must be running):

```bash
python -m pytest appium_tests -m appium --android-udid YOUR_SERIAL -v --html=reports/qa-report.html --self-contained-html
```

When exactly one device is online, `--android-udid` can be omitted. Set `APP_PACKAGE` in `.env` to enable the optional app-launch smoke test.

Run the Appium smoke suite sequentially on every online device:

```bash
python -m scripts.run_device_matrix
```

Reports and failure evidence are saved under `reports/`. The matrix runner is sequential by design; it does not claim parallel execution.

## Existing utilities

### Excel-driven Android E2E checks

Create a sample workbook:

```bash
python -m android_e2e.runner --gerar-modelo casos.xlsx
```

Run via ADB (screenshots with ADB):

```bash
python -m android_e2e.runner casos.xlsx --serial YOUR_SERIAL --saida resultados.xlsx --prints prints/
```

Run via Appium (Appium screenshot API):

```bash
python -m android_e2e.runner casos.xlsx --serial YOUR_SERIAL --appium
```

Run sequentially on every online device (one workbook per device):

```bash
python -m android_e2e.runner casos.xlsx --all-devices --appium
```

### Logcat diagnostics

```bash
python -m android_e2e.cli logcat --serial YOUR_SERIAL --buffer main --out reports/logcat.txt
```

### Carrier configuration validator

```bash
python -m carrier_validator.validator carrier_validator/sample_configs.json
```

Rules in `carrier_validator/validator.py` are examples and should be adjusted to actual, authorized requirements before production use.

### Boot-animation comparison

```bash
python -m bootanim_compare.compare source.zip target.zip -o reports/bootanim-diff.md
python -m bootanim_compare.gui
```

## Repository layout

```text
android-qa-toolkit/
├── android_e2e/           # ADB, device inventory, Appium sessions, Excel E2E, Logcat
├── appium_tests/          # device-dependent pytest suite
├── bootanim_compare/      # bootanimation.zip comparison + GUI
├── carrier_validator/     # generic JSON carrier config validation
├── docs/                  # compatibility and setup notes
├── scripts/               # device matrix runner
├── tests/                 # hardware-independent unit tests
├── reports/               # generated reports/evidence (git-ignored)
└── .github/workflows/     # automated unit-test CI
```

## Compatibility and limitations

Appium UiAutomator2 support depends on driver version and Android OS level. The recommended pin is selected to retain broader older-device support; older Android releases may still have partial compatibility. OEM-specific UI changes may require app-specific locator adjustments. See `docs/DEVICE_SUPPORT.md`.

The toolkit performs ordinary ADB/Appium QA operations. It does not root devices, unlock bootloaders, bypass security controls, or alter modem firmware/carrier locks.

## License

MIT. Third-party tools and libraries retain their own licenses; see their upstream projects.
