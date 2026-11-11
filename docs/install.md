# Installation and execution

[한국어](install-ko.md) · [README](../README.md)

The platform folders contain the original application files extracted from the respective installers. Each `extracted/` directory contains application code and assets; each `resources/` directory contains the original `app.asar` and platform-specific support files. The Electron runtime itself is not included.

| Platform | Original app version | Original installer architecture |
| --- | --- | --- |
| Windows | 1.19.2 | x64 |
| macOS | 1.4.2 | Intel x64 and Apple Silicon arm64 |
| Linux | 1.0.0 | x64 |

Use **Electron 44.2.0**, the version found in all three installers. The macOS installer specifies macOS 13.0 or later. Windows ARM64 and Linux ARM64 use a different architecture from the original installers and require separate validation. Raspberry Pi ARM64 execution has not been verified.

## Prerequisites and setup

Install Git, **Node.js 22.12 or newer**, and npm. Cloning and installing Electron require an internet connection. If the repository is private, authenticate to an account with access before cloning.

Run the following commands from a new working directory. They create sibling `analysis` and `bug-clicker-electron` directories, keeping the Electron installation outside the cloned repository:

```sh
git clone -c core.autocrlf=false https://github.com/Protocol314/BUG-Clicker.git analysis
npm install --prefix ./bug-clicker-electron --no-save electron@44.2.0
node ./bug-clicker-electron/node_modules/electron/install.js
```

Use PowerShell on Windows and a terminal on macOS or Linux. If you already have an `analysis` directory, skip the clone and run the installation commands from its **parent directory**. Keep the complete installed Electron runtime directory; copying the executable alone is insufficient.

## Windows

In PowerShell, from the same parent directory used above, copy the original support files to Electron's resources directory and start the app:

```powershell
Copy-Item .\analysis\windows\resources\update.json .\bug-clicker-electron\node_modules\electron\dist\resources\
Copy-Item .\analysis\windows\resources\elevate.exe .\bug-clicker-electron\node_modules\electron\dist\resources\

& .\bug-clicker-electron\node_modules\electron\dist\electron.exe .\analysis\windows\extracted
```

## Linux

A graphical desktop session and the distribution's Electron system libraries are required, including GTK, NSS/NSPR, X11 or Wayland, and ALSA. The following commands apply only to the listed Debian-family versions; they require access to the package repositories and administrator privileges. Raspberry Pi requires a 64-bit OS and a GUI. ARM64 is different from the original x64 installer and remains unverified.

**Debian 13, Raspberry Pi OS based on Debian 13, or Ubuntu 24.04:**

```sh
sudo apt update
sudo apt install -y python3 libasound2t64 libatk-bridge2.0-0t64 \
  libatk1.0-0t64 libatspi2.0-0t64 libcups2t64 libglib2.0-0t64 \
  libgtk-3-0t64 libnspr4 libnss3 libdbus-1-3 libdrm2 libgbm1 \
  libx11-6 libx11-xcb1 libxcb1 libxcomposite1 libxdamage1 libxext6 \
  libxfixes3 libxkbcommon0 libxrandr2 libxshmfence1 libxss1 libxrender1 \
  libpango-1.0-0 libcairo2 libnotify4 libuuid1
```

**Debian 12, Raspberry Pi OS based on Debian 12, or Ubuntu 22.04:**

```sh
sudo apt update
sudo apt install -y python3 libasound2 libatk-bridge2.0-0 \
  libatk1.0-0 libatspi2.0-0 libcups2 libglib2.0-0 \
  libgtk-3-0 libnspr4 libnss3 libdbus-1-3 libdrm2 libgbm1 \
  libx11-6 libx11-xcb1 libxcb1 libxcomposite1 libxdamage1 libxext6 \
  libxfixes3 libxkbcommon0 libxrandr2 libxshmfence1 libxss1 libxrender1 \
  libpango-1.0-0 libcairo2 libnotify4 libuuid1
```

Package names may differ on other releases. After installing Electron as above, check for missing shared libraries:

```sh
ldd ./bug-clicker-electron/node_modules/electron/dist/electron | grep 'not found'
```

If this prints no `not found` lines, `ldd` found all of the libraries it inspected. From the same parent directory used for setup, copy the original update configuration and run the app:

```sh
cp ./analysis/linux/resources/update.json ./bug-clicker-electron/node_modules/electron/dist/resources/

./bug-clicker-electron/node_modules/electron/dist/electron ./analysis/linux/extracted
```

### Start automatically at graphical login

The original application's built-in autostart does not work when the extracted app runs using development Electron. On a graphical desktop that supports [XDG Autostart](https://specifications.freedesktop.org/autostart/0.5/), the following creates a separate autostart entry for the current user. First verify that manual execution works and close the app. Run the command from the **parent directory containing both `analysis` and `bug-clicker-electron`**:

```sh
python3 - <<'PY'
from pathlib import Path
import os
import shlex

base = Path.cwd().resolve()
electron = base / 'bug-clicker-electron/node_modules/electron/dist/electron'
app = base / 'analysis/linux/extracted'
if not electron.is_file() or not os.access(electron, os.X_OK) or not (app / 'package.json').is_file():
    raise SystemExit('Electron or the original app is missing. Complete setup and manual execution first.')

home = Path.home()
config = Path(os.environ.get('XDG_CONFIG_HOME') or str(home / '.config'))
if not config.is_absolute():
    raise SystemExit('XDG_CONFIG_HOME must be an absolute path.')
launcher = home / '.local/share/bug-clicker-original-autostart/start.sh'
desktop = config / 'autostart/bug-clicker-original.desktop'
if os.path.lexists(launcher) or os.path.lexists(desktop):
    raise SystemExit('An autostart file already exists. Inspect it and use the removal instructions below if appropriate.')

launcher.parent.mkdir(parents=True, exist_ok=True)
desktop.parent.mkdir(parents=True, exist_ok=True)
with launcher.open('x', encoding='utf-8') as f:
    f.write('#!/bin/sh\nexec ' + shlex.quote(str(electron)) + ' ' + shlex.quote(str(app)) + '\n')
launcher.chmod(0o700)
with desktop.open('x', encoding='utf-8') as f:
    f.write('[Desktop Entry]\nType=Application\nName=BUG Clicker\n'
            + r'Exec=sh -c "exec \\"\\$HOME/.local/share/bug-clicker-original-autostart/start.sh\\""'
            + '\nTerminal=false\n')
print('Autostart entry:', desktop)
print('Launch script:', launcher)
PY
```

Log out and sign in to the graphical desktop as the **same user**. This does not start the app on a text-console login or a system without a desktop. If the folders move, remove the entry using the commands below and recreate it from the new parent directory. Paths containing spaces are supported. The app's user-data and update behavior is the same as when started manually. The entry generator and launcher were tested in an isolated Linux environment; a full graphical logout and login remains unverified.

To remove the autostart entry:

```sh
rm -f -- "${XDG_CONFIG_HOME:-$HOME/.config}/autostart/bug-clicker-original.desktop"
rm -f -- "$HOME/.local/share/bug-clicker-original-autostart/start.sh"
```

## macOS

The commands below copy the original update configuration into the separately installed Electron app bundle and start the original app. Adding files to the bundle may change its signature status. Gatekeeper and actual execution must be tested on macOS.

```sh
cp ./analysis/mac/resources/update.json ./bug-clicker-electron/node_modules/electron/dist/Electron.app/Contents/Resources/
cp ./analysis/mac/resources/app-update.yml ./bug-clicker-electron/node_modules/electron/dist/Electron.app/Contents/Resources/

./bug-clicker-electron/node_modules/electron/dist/Electron.app/Contents/MacOS/Electron ./analysis/mac/extracted
```

To use the original typing-detection feature, close Electron, place its original helper next to the executable, and rerun the command above:

```sh
cp ./analysis/mac/resources/bug-clicker-keys ./bug-clicker-electron/node_modules/electron/dist/Electron.app/Contents/MacOS/
chmod 755 ./bug-clicker-electron/node_modules/electron/dist/Electron.app/Contents/MacOS/bug-clicker-keys
```

This helper can also affect bundle-signature validity and may require macOS Input Monitoring permission. Gatekeeper, signing, and permission behavior have not been tested on an actual Mac.

## Runtime and distribution notes

- The original app may read or modify existing BUG Clicker settings or data paths. Back up any data you need before running it.
- Original update and announcement features use the creator's GitHub URLs, and the app contains the creator's guide and contact links. These addresses have not been changed.
- To prevent the bootloader from loading an already downloaded update patch, append `--no-patch` to the execution command for your OS. This does not block all network access.
- Development Electron can differ from the original installer in app signing, icons, file paths, automatic updates, and OS integration. Basic Windows and Linux screens were checked, but not every feature or target computer has been validated.
- Electron 44.2.0 is pinned to match the original installers; it does not automatically include subsequent Electron security fixes.
- A license field in an application's `package.json` does not by itself establish redistribution rights for every character, image, sound, or other asset. Check each component's rights and licenses before publishing, and retain included notices.

## Creating a separate repository

To publish a separate copy, create an empty GitHub repository and run the commands below from this repository's root, replacing the remote URL with your own. Set `core.autocrlf` to `false` **before** initially adding files to avoid automatic text line-ending conversion. This setting cannot retroactively restore bytes already converted in an existing commit.

```sh
git init
git config core.autocrlf false
git add .
git commit -m "Add original BUG Clicker files"
git branch -M main
git remote add origin https://github.com/USER/REPOSITORY.git
git push -u origin main
```
