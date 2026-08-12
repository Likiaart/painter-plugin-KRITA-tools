# Krita Export - Substance 3D Painter plugin

This plugin adds an __Export To Krita__ button to the Send To menu of Substance Painter. The export action creates one Krita document for each channel of each stack of the Substance Painter document, preserving the stack hierarchy (layers and groups). Blend modes and filters are preserved to the best of the programs possibility.

## Installation

1. Download or clone this project into the Substance Painter plugins folder. The plugins folder can be opened from Substance Painter through the menu `Plugins/Plugins folder`.
2. Restart Substance Painter for the plugin to take effect.
3. Open the plugin's configure panel (`Plugins → Krita Export → Configure`) and paste the full path to `kritarunner.exe`, which ships with Krita and can be found at `C:\Program Files\Krita (x64)\bin\kritarunner.exe` on Windows.
4. Make sure Pillow (PIL) is available to Krita's own bundled Python interpreter — see below. Without it, exports will silently fail to produce any `.kra` files.

## Requirement: Pillow (PIL) in Krita's Python environment

The generated Krita-side script (`runner_header.py`) relies on the Pillow library (`from PIL import Image`) to read and write the exported texture PNGs. Krita ships its own embedded Python interpreter, which does **not** include Pillow by default, and it is entirely separate from any Python installed elsewhere on your system. Pillow has to be installed specifically into that embedded interpreter's search path.

**Steps (Windows):**

1. Check Krita's exact bundled Python version. As of Krita 5.2.x, this is **Python 3.10 (64-bit)**. This can change between Krita major versions, so double check if you're on a different Krita version.
2. Install a matching Python 3.10 (64-bit) on your system if you don't already have one. You do not need to set it as your default Python; it's only used once, to fetch the correct Pillow build.
   - Using the modern Python launcher: `py install 3.10`, then confirm with `py -3.10 --version`.
   - Or download the Python 3.10 64-bit installer directly from [python.org/downloads/release](https://www.python.org/downloads/release).
3. Install Pillow directly into the folder Krita's scripting engine (`kritarunner`) already searches for scripts:
   ```
   py -3.10 -m pip install --target="%APPDATA%\kritarunner" pillow
   ```
   (Replace `%APPDATA%\kritarunner` with the literal expanded path if your shell doesn't expand it automatically, e.g. `C:\Users\<you>\AppData\Roaming\kritarunner`.)
4. You can sanity-check that `kritarunner.exe` can now run without errors by running it directly from a terminal:
   ```
   "C:\Program Files\Krita (x64)\bin\kritarunner.exe" -s runner
   ```
   (This assumes a `runner.py` script already exists in `%APPDATA%\kritarunner`, which is generated automatically the first time you run an export from Substance Painter.)

If you use a different major version of Krita and it fails with `ModuleNotFoundError: No module named 'PIL'`, repeat the steps above with the Python version matching that Krita release (check via `Help → About Krita` in Krita, or the Krita release notes).

## Troubleshooting

`kritarunner.exe` is a GUI-subsystem application, so it will not print errors to a terminal even when something goes wrong internally — it just returns silently. If an export runs in Substance Painter without errors but no `.kra` files appear in the `<project>_krita_export` folder, the most reliable way to see what actually happened inside Krita's Python environment is:

- Check `%APPDATA%\kritarunner\debug.log` if present — some diagnostic logging can be re-enabled in `runner_header.py`/`runner_footer.py` (see the `debugLog` helper function) if you need to troubleshoot further.
- Or use [Sysinternals DebugView](https://learn.microsoft.com/en-us/sysinternals/downloads/debugview) to capture Krita's internal debug/error output (including Python tracebacks), which isn't visible in a normal command prompt.

A message box saying *"The generated image is not the same as the snapshot. Make sure you are not using geometry masks..."* is an expected warning, not a bug: it means one of the exported layers used a **geometry mask** (curvature, position, ambient occlusion, etc., computed from the 3D mesh), which cannot be exported to a 2D texture representation. The rest of the export completes normally; only that specific layer's mask may need to be recreated manually in Krita.
