# Android QA Toolkit

A Python toolkit for Android device inspection, Appium UI smoke tests, and Logcat collection.

## Requirements
- Python 3.10+
- Android SDK Platform-Tools (`adb`) on PATH
- Appium Server 2.x
- Appium UiAutomator2 driver
- Android device with USB debugging enabled, or an emulator

Install the Appium driver:
```bash
appium driver install uiautomator2
```

## Setup (Windows PowerShell)
```powershell
py -3 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
Copy-Item .env.example .env
```

## Setup (Linux/macOS)
```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
pip install -r requirements.txt
cp .env.example .env
```

Set `ANDROID_UDID` in `.env` to the serial shown by `adb devices`.
Never commit `.env`.

## Run
Terminal 1:
```bash
appium
```
Terminal 2:
```bash
adb devices
python -m scripts.device_info
pytest -m "not appium" -v
pytest -m appium -v
python -m scripts.collect_logs
```

The Appium test opens Android Settings and requires a connected device and running Appium server. Log files are written to `reports/` and excluded from Git.

## Environment variables
See `.env.example`: `APPIUM_SERVER_URL`, `ANDROID_UDID`, `ANDROID_PLATFORM_VERSION`, `APP_PACKAGE`, `APP_ACTIVITY`, `LOGCAT_BUFFER`.

## Layout
- `android_qa/`: reusable ADB, Appium, and Logcat helpers
- `appium_tests/`: tests requiring an Android device
- `tests/`: local unit tests
- `scripts/`: command-line utilities
- `config/`: reserved for future project configuration

## Scope
Initial features are read-only device inspection, UI smoke testing, and log collection. The toolkit does not root devices, unlock bootloaders, change carrier settings, or modify modem firmware.

## License
MIT. See `LICENSE`.
